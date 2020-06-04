---
layout:     post
title:      "Microservices with Spring Boot & Spring Cloud"
subtitle:   "Part VII: Setting Up Reservation Service (Domain Specific API)"
date:       2017-09-17 12:00:00
author:     "Tomy Jaya"
header-img: "img/post-bg-06.jpg"
tags:
- spring
- spring-boot
- microservices
- spring-cloud
- cloud
---

# Setting Up Reservation Service (Domain Specific API)

## Introduction

Huff! Finally, five services later, now we're ready to write the actual meat (aka domain specific business logic) of our application. 
Basically, our application is just a simple CRUD application to manage reservations. Nothing fancy, we'll use `spring-data` to do most of the heavy lifting configuration and boilerplate for us. We use in memory H2 for our database for this tutorial's simplicity sake. In reality, you can plug almost any popular SQL or even NoSQL databases to `spring-data`. 

In addition, for fail-safe write, we will use `spring-cloud-stream` to manage our message-driven interaction. Again, even though we use `kafka` for this tutorial, there's nothing stopping you to use other popular message brokers (e.g. RabbitMQ)

## Setting up Reservation Service's Configuration

In the existing config's git: 
```
vim reservation-service.properties

### Add the below to reservation-service.properties
server.port=${PORT:8000}
message=Hello World!

# define the destination to which the input MessageChannel should be bound
spring.cloud.stream.bindings.input.destination=reservations

# ensures 1 node in a group gets message (point-to-point, not a broadcast)
spring.cloud.stream.bindings.input.group=reservations-group

# ensure that the Q is durable
spring.cloud.stream.bindings.input.durableSubscription=true

# To avoid sending zipkin spans/ traces via kafka
spring.zipkin.sender.type=web
## End of reservation-service.properties

git add reservation-service.properties
git commit -m "Add reservation-service.properties"
```

## Setting up Reservation Service

1. Go to https://start.spring.io/

2. Type `reservation-service` as the "Artifact" name, add `Web`, `JPA`, `Actuator`, `H2`, `Config Client`, `Eureka Discovery`, `Lombok`, `Zipkin Client`, `Stream Kafka` dependencies, and "Generate Project": 
![reservation-service-spring-initializr-screenshot.png](/img/reservation-service-spring-initializr-screenshot.png){:height="343px" width="840px"}

3. Open the project in your favorite Java IDE (mine is IntelliJ)

4. Create a simple entity `Reservation` with `id` and `reservationName` and furnish it with the below JPA and Lombok annotations:

    ```java
    package com.example.reservationservice;

    import lombok.AllArgsConstructor;
    import lombok.Data;
    import lombok.NoArgsConstructor;

    import javax.persistence.Entity;
    import javax.persistence.GeneratedValue;
    import javax.persistence.Id;

    @Entity
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class Reservation {

        @Id
        @GeneratedValue
        private Long id;

        private String reservationName;
    }
    ```

    *NOTE*: If you use IntelliJ, you will need to do the below for Lombok to work seamlessly: 
    1. Install the [Lombok plugin](https://plugins.jetbrains.com/plugin/6317-lombok-plugin)
    2. Enable Annotation Processing [documentation](https://www.jetbrains.com/help/idea/configuring-annotation-processing.html)

5. Create a corresponding `JpaRepository` for the above entity, name it `ReservationRepository`: 

    ```java
    package com.example.reservationservice;

    import org.springframework.data.jpa.repository.JpaRepository;

    import java.util.Collection;

    public interface ReservationRepository extends JpaRepository<Reservation, Long> {
        Collection<Reservation> findByReservationName(String reservationName);
    }
    ```

6. Create a `@RestController` to access our reservations, name it `ReservationRestController`: 

    ```java
    package com.example.reservationservice;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;

    import java.util.Collection;

    @RestController
    public class ReservationRestController {
        private ReservationRepository reservationRepository;

        @Autowired
        ReservationRestController(ReservationRepository reservationRepository) {
            this.reservationRepository = reservationRepository;
        }

        @GetMapping("/reservations")
        Collection<Reservation> reservations() {
            return this.reservationRepository.findAll();
        }
    }
    ```

7. For the "write" part of service, we want to use an input channel named `input` declared in `ReservationChannels` interface:

    ```java
    package com.example.reservationservice;

    import org.springframework.cloud.stream.annotation.Input;
    import org.springframework.messaging.SubscribableChannel;

    public interface ReservationChannels {

        @Input
        SubscribableChannel input();
    }
    ```

8. Next, we define a processor (`MessageEndpoint`) for which is activated upon an incoming messsage from the `input` channel: 

    ```java
    package com.example.reservationservice;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.integration.annotation.MessageEndpoint;
    import org.springframework.integration.annotation.ServiceActivator;
    import org.springframework.messaging.Message;

    @MessageEndpoint
    public class ReservationProcessor {

        private ReservationRepository reservationRepository;

        @Autowired
        public ReservationProcessor(ReservationRepository reservationRepository) {
            this.reservationRepository = reservationRepository;
        }

        @ServiceActivator(inputChannel = "input")
        public void acceptNewReservation(Message<String> msg) {
            reservationRepository.save(new Reservation(null,msg.getPayload()));
        }
    }
    ```

9. Our beans are ready. Next we need to annotate `@EnableDiscoveryClient` and `@EnableBindings` on `ReservationChannels` onto our main class `ReservationServiceApplication`: 

    ```java
    package com.example.reservationservice;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
    import org.springframework.cloud.stream.annotation.EnableBinding;

    @SpringBootApplication
    @EnableDiscoveryClient
    @EnableBinding(ReservationChannels.class)
    public class ReservationServiceApplication {

    	public static void main(String[] args) {
    		SpringApplication.run(ReservationServiceApplication.class, args);
    	}
    }
    ```

10. Finally, create a `@Component` which implements `CommandLineRunner` to pre-populate our in-memory database on startup: 

    ```java
    package com.example.reservationservice;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.CommandLineRunner;
    import org.springframework.stereotype.Component;

    import java.util.stream.Stream;

    @Component
    public class SampleDataClr implements CommandLineRunner {
        private ReservationRepository reservationRepository;

        @Autowired
        public SampleDataClr(ReservationRepository reservationRepository) {
            this.reservationRepository = reservationRepository;
        }

        @Override
        public void run(String... strings) throws Exception {
            Stream.of("TOMY JAYA", "JOHN DOE").forEach(name -> {
                reservationRepository.save(new Reservation(null, name));
            });
        }
    }
    ```


11. in `src/main/resources`, create a new file called `bootstrap.properties`, add the below:

    ```properties
    # tell config service what is the name of the service and corresponding config to retrieve

    spring.application.name=reservation-service

    # tell the location the location of config server

    spring.cloud.config.uri=http://localhost:8888
    ```

12. Run `ReservationServiceApplication` and voila! Your `reservation-service` is started!

13. Go to http://localhost:8000/reservations to see the list of reservations. 

    ![reservation-service-result](/img/reservation-service-result.png){:height="220px" width="312px"}

9. Since we configure `spring-boot-actuator` on this service, we can monitor some production metrics exposed from several HTTP endpoints. 

    But to be able see Actuator's sensitive endpoints, we need to disable spring security by adding the below into `application.properties` in `src/main/resources`:

    ```properties
    management.security.enabled=false
    ```

    Restart `ReservationServiceApplication` and try to access http://localhost:8000/metrics for some useful stats about your microservice:

    ![actuator-metrics](/img/actuator-metrics.png){:height="369px" width="379px"}

    Other actuator endpoints I found useful are: 

    | Endpoint  | Use |
    | ------------- | ------------- |
    | http://localhost:8000/env  | list all properties |
    | http://localhost:8000/beans  | list all the available Spring beans |
    | http://localhost:8000/trace  | trace the last 100 HTTP requests info |
    | http://localhost:8000/mappings  | This one is a gem! It lists all the HTTP endpoint paths & corresponding handlers |
    | http://localhost:8000/health  | Tell if it's UP or DOWN |
    | http://localhost:8000/info  | You can inject custom indicator here|

    Let's try to create a custom info indicator: 

    ```properties
    info.app.name=Reservation Service

    info.app.description=Reservation Service (BACKEND API)

    info.app.version=1.0.0
    ```

    Let's try to create a custom health indicator: 

    ```java
    package com.example.reservationservice;

    import org.springframework.boot.actuate.health.Health;
    import org.springframework.boot.actuate.health.HealthIndicator;
    import org.springframework.stereotype.Component;

    @Component
    public class CustomHealthIndicator implements HealthIndicator {
        @Override
        public Health health() {
            return Health.status("YO!").build();
        }
    }
    ```

    Restart and try to access http://localhost:8000/info: 

    ![custom-info-indicator](/img/custom-info-indicator.png){:height="153px" width="461px"}

    And in http://localhost:8000/health, you should see: 

    ![custom-health-indicator](/img/custom-health-indicator.png){:height="149px" width="381px"}


## Conclusion:
We have now set up our production ready Reservation Service which can read & write reservations stored in an H2 database.
We haven't tested the write yet, though, as it requires our `reservation-client` to push the message to our subscribed input channel. Let's do that in the next post. 


## Other Useful references:
1. [Spring-Boot-Actuator Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-monitoring.html)
2. [Spring-Boot-Actuator Documentation 2](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready)

## Next Up
[Setting Up Reservation Client (Edge Service)](/2017/09/17/spring-microservices-8/)
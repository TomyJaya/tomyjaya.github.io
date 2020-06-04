---
layout:     post
title:      "Microservices with Spring Boot & Spring Cloud"
subtitle:   "Part VIII: Setting Up Reservation Client (Edge Service)"
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

# Setting Up Reservation Client (Edge Service)

## Introduction

We are now at our last service to build which is our edge service. Edge Service pattern is widely used in microservices architecture to hide multiple backend services behind one gateway/ front-door. It is similar to the Facade design pattern, however, an edge service not only routes requests to the correct backend services, it usually serves as a point to insert common cross-cutting features such as security (CORS prevention), authentication, and fault tolerance. You will usually also put your React/ Angular SPA in this edge service. Moreover, an edge service is also a logical place to insert API translation or protocol translation requirements. 

Zuul is Netflix's micro proxy solution we will use. It will sit as an intermediary between clients (e.g. smart phones, HTML5 applications) and our service. Our example will only show a client for HTML5 browser. 

## Setting up Reservation Client's Configuration

In the existing config's git: 
```
vim reservation-client.properties

### Add the below to reservation-service.properties
server.port=${PORT:9999}

# define the destination to which the output MessageChannel should be bound
spring.cloud.stream.bindings.output.destination=reservations

# define oauth URL
security.oauth2.resource.userInfoUri=http://localhost:9191/uaa/user

# To avoid sending zipkin spans/ traces via kafka
spring.zipkin.sender.type=web
## End of reservation-client.properties

git add reservation-client.properties
git commit -m "Add reservation-client.properties"
```

## Setting up Reservation Client

1. Go to https://start.spring.io/

2. Type `reservation-client` as the "Artifact" name, add `Web`, `Lombok`, `Feign`, `Zuul`, `Hystrix`, `Stream Kafka`, `Eureka Discovery`, `Config Client`, `Cloud OAuth2`, `HATEOAS` and `Zipkin Client` dependencies, and "Generate Project": 
![reservation-client-spring-initializr-screenshot.png](/img/reservation-client-spring-initializr-screenshot.png){:height="400px" width="800px"}

3. Open the project in your favorite Java IDE (mine is IntelliJ)

4. Since we'll have to have a domain model in the client side as well, again, create a simple entity `Reservation` with `id` and `reservationName` and furnish it with the below Lombok annotations:

    ```java
    package com.example.reservationclient;

    import lombok.AllArgsConstructor;
    import lombok.Data;
    import lombok.NoArgsConstructor;

    @AllArgsConstructor
    @NoArgsConstructor
    @Data
    public class Reservation {
        private Long id;
        private String reservationName;
    }
    ```

5. Define `ReservationReader` interface which is a `FeignClient`: 

    ```java
    package com.example.reservationclient;

    import org.springframework.cloud.netflix.feign.FeignClient;
    import org.springframework.web.bind.annotation.GetMapping;

    import java.util.Collection;

    @FeignClient("reservation-service")
    interface ReservationReader {

        @GetMapping("/reservations")
        Collection<Reservation> read();
    }
    ```

    [Feign](https://github.com/OpenFeign/feign) is yet another Java http client developed by Netflix. Its primary selling point is its declarative style which saves a lot of overheads.

6. If we just want a dumb proxy, `@EnableZuulProxy` will suffice. However, in this scenario, our API gateway usually needs to do some "work". Create a class called `ReservationApiGatewayRestController`: 

    ```java
    package com.example.reservationclient;

    import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.cloud.stream.messaging.Source;
    import org.springframework.core.ParameterizedTypeReference;
    import org.springframework.hateoas.Resources;
    import org.springframework.http.HttpMethod;
    import org.springframework.http.ResponseEntity;
    import org.springframework.messaging.Message;
    import org.springframework.messaging.MessageChannel;
    import org.springframework.messaging.support.MessageBuilder;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.RestController;
    import org.springframework.web.client.RestTemplate;

    import java.util.Collection;
    import java.util.Collections;
    import java.util.stream.Collectors;

    @RestController
    @RequestMapping("/reservations")
    public class ReservationApiGatewayRestController {

        private ReservationReader reservationReader;
        private MessageChannel output;

        @Autowired
        public ReservationApiGatewayRestController(ReservationReader reservationReader, Source source) {
            this.reservationReader = reservationReader;
            this.output = source.output();
        }

        public Collection<String> fallback() {
            System.out.println("fallback hit");
            return Collections.EMPTY_LIST;
        }

        @RequestMapping(method = RequestMethod.GET, value="names")
        @HystrixCommand(fallbackMethod = "fallback")
        public Collection<String> getReservationNames() {
            return this.reservationReader.read()
                    .stream()
                    .map(Reservation::getReservationName)
                    .collect(Collectors.toList());
        }

        @RequestMapping(method = RequestMethod.POST)
        public void write(@RequestBody Reservation r) {
            Message<String> msg = MessageBuilder.withPayload(r.getReservationName()).build();
            this.output.send(msg);
        }

    }
    ```

    1. There is a lot happening above. First we declare a RESTful endpoint "reservations". 
    2. It has the usual Spring constructor injection setup. We need 2 beans: 1) `RestTemplate` to relay calls to our backend reservation service AND 2) `MessageChannel` obtained via `Source` sink to push write messages to our backend reservation-service as well. 
    3. We define a fallback method for Hystrix circuit breaker, we name the method "`fallback`". It just returns an empty List. 
    4. Next, we define a GET endpoint called "`names`" and inside, we use our feign client `reservationReader` to get the reservations from our backend service and massage the data to only return the names. Note that it's annotated with `HystrixCommand` with `fallbackMethod` called `fallback` (as defined in 3). In case there's any exception (e.g. our backend service is down), this endpoint will run the fallback method instead of reporting back the ugly exception message. 
    5. Finally, we have the POST endpoint for write which just send a message to the `output` channel. 
    
7. Create a class called `RateLimiterFilter` below: 

    ```java
    package com.example.reservationclient;

    import com.google.common.util.concurrent.RateLimiter;
    import com.netflix.zuul.ZuulFilter;
    import com.netflix.zuul.context.RequestContext;
    import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
    import org.springframework.core.Ordered;
    import org.springframework.http.HttpStatus;
    import org.springframework.stereotype.Component;

    import javax.servlet.http.HttpServletResponse;

    @Component
    class RateLimitingZuulFilter extends ZuulFilter {

        private RateLimiter rateLimiter = RateLimiter.create(.1);

        @Override
        public String filterType() {
            return FilterConstants.PRE_TYPE;
        }

        @Override
        public int filterOrder() {
            return Ordered.HIGHEST_PRECEDENCE;
        }

        @Override
        public boolean shouldFilter() {
            return true;
        }

        @Override
        public Object run() {
            RequestContext currentContext = RequestContext.getCurrentContext();
            HttpServletResponse response = currentContext.getResponse();

            if (!rateLimiter.tryAcquire()) {
                currentContext.setSendZuulResponse(false);
                response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
                
            }
            return null;
        }
    }
    ```

    NOTE: 
    1. To avoid our API gateways from being bombarded with requests, we usually set up what's called rate limiters. Basically, requests happening more frequently than a specified range will be bounced back. 
    2. Spring will register all filters as long as it's annotated `@Component` and extends `ZuulFilter`
    3. We use Guava's rate limiter with value `0.1`. It means 1 request every 10 seconds. 
    4. `ZuulFilter`s *only* filters Zuul's proxied services (e.g. not our defined RestController `ReservationApiGatewayController`)

8. And finally, in `ReservationClientApplication`: 

    ```java
    package com.example.reservationclient;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
    import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
    import org.springframework.cloud.client.loadbalancer.LoadBalanced;
    import org.springframework.cloud.netflix.feign.EnableFeignClients;
    import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
    import org.springframework.cloud.stream.annotation.EnableBinding;
    import org.springframework.context.annotation.Bean;
    import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
    import org.springframework.web.client.RestTemplate;

    import org.springframework.cloud.stream.messaging.Source;

    @SpringBootApplication
    @EnableResourceServer
    @EnableBinding(Source.class)
    @EnableCircuitBreaker
    @EnableFeignClients
    @EnableDiscoveryClient
    @EnableZuulProxy
    public class ReservationClientApplication {

        @Bean
        @LoadBalanced
        RestTemplate restTemplate() {
            return new RestTemplate();
        }

        public static void main(String[] args) {
            SpringApplication.run(ReservationClientApplication.class, args);
        }
    }
    ```

    Let me have a shot as explaining all the annotations above: 
    1. `@EnableResourceServer` is to activate OAuth2 security
    2. `@EnableBinding` is to set up our channel with default name `output`
    3. `@EnableCircuitBreaker` is to tell hystrix to activate its circuit breaker
    4. `@EnableFeignClients` & `@LoadBalanced` are to use Feign as our web service client (it's integrated with Eureka and can provide load-balancing)
    5. `@EnableDiscoveryClient` is again, to register our service to Eureka
    6. `@EnableZuulProxy` is our main annotation to tell Zuul to proxy all services in Eureka in this `reservation-client`

9. in `src/main/resources`, create a new file called `bootstrap.properties`, add the below:

    ```properties
    # tell config service what is the name of the service and corresponding config to retrieve

    spring.application.name=reservation-client
    
    # tell the location the location of config server
    
    spring.cloud.config.uri=http://localhost:8888
    ```

10. Make sure you run all the other services in the previous posts (including Kafka). When they are all booted up, run `ReservationClientApplication` and finally our edge service is started!

11. Let's first test our vanilla Zuul proxied services at http://localhost:9999/reservation-service/reservations. You should get an authorized error message, which means our spring security is working correctly: 

    ```xml
    <oauth>
        <error_description>
        Full authentication is required to access this resource
        </error_description>
        <error>unauthorized</error>
    </oauth>
    ```

12. Open your favorite Terminal/ Git Bash as we'll need `curl`'s help for now onwards to test our services. 

13. Fire the below curl command to get an oauth token: 

    `curl -X POST -vu android:secret http://localhost:9191/uaa/oauth/token -H "Accept: application/json" -d "password=spring&username=tomy&grant_type=password&scope=openid&client_secret=secret&client_id=android"`

14. Now use your retrieved token: 

    `curl http://localhost:9999/reservation-service/reservations -H "Authorization: Bearer 02d0ce25-c667-4571-a31c-91b14bc79fd9"`

    NOTE: Change `02d0ce25-c667-4571-a31c-91b14bc79fd9` with our own generated token from 13. 

    you should get the below JSON results: 

    `[{"id":1,"reservationName":"TOMY JAYA"},{"id":2,"reservationName":"JOHN DOE"}]`

15. Now let's see if our RateLimiter is working, use the same curl command within 10 seconds of the last request and you should *not* see anything returned. 


16. Next, let's test our API Gateway: 

    `curl http://localhost:9999/reservations/names -H "Authorization: Bearer 02d0ce25-c667-4571-a31c-91b14bc79fd9"`

    you should get the below JSON results:

    `["TOMY JAYA","JOHN DOE"]`

17. So far so good, let's digress and see our Eureka dashboard at http://localhost:8761/. You should see our services properly registered:
    
    ![eureka-dashboard-final.png](/img/eureka-dashboard-final.png){:height="434px" width="908px"}

18. Let's test if our reservation-client breaks gracefully as we've set up a circuit breaker. You can kill our reservation-service instance then: 

    `curl http://localhost:9999/reservations/names -H "Authorization: Bearer 02d0ce25-c667-4571-a31c-91b14bc79fd9"`

    you should get an empty array `[]`. However, if you hit the below: 

    `curl http://localhost:9999/reservation-service/reservations -H "Authorization: Bearer 02d0ce25-c667-4571-a31c-91b14bc79fd9"` 

    you should see something like the below: 

    `{"timestamp":1515170276976,"status":404,"error":"Not Found","message":"No message available","path":"/reservation-service/reservations"}`

    Now we understand the importance of guarding our clients using circuit breakers. 

19. Okay, how about writes? Let's try to add a new reservation: 

    `curl -X POST -d '{"reservationName": "VALERIE"}' -H "Content-Type: application/json" -H "Authorization: Bearer 02d0ce25-c667-4571-a31c-91b14bc79fd9" http://localhost:9999/reservations`

20. I'm aware that `reservation-service` is down. But no problem, we've designed for fail-safe writes. Let's boot up our `reservation-service` now by running `ReservationServiceApplication`.

21. Wait for a while since Hystrix needs time to flip back the circuit breaker. When we query again: 

    `curl http://localhost:9999/reservations/names -H "Authorization: Bearer 02d0ce25-c667-4571-a31c-91b14bc79fd9"`
    
    we should get the newly added reservation: 

    `["TOMY JAYA","JOHN DOE","VALERIE"]`

22. Open our hystrix dashboard at http://localhost:8010/hystrix.html. Put in http://localhost:9999/hystrix.stream in the textbox and click "Monitor". Now, you should be able to see requests going to our circuit breakers. Play around by killing reservation-service and bringing it up again to see how the graph looks like. 

    ![hystrix-dashboard-final.png](/img/hystrix-dashboard-final.png){:height="443px" width="769px"}

23. Finally, let's open our zipkin dashboard to see if we can trace our requests

    ![zipkin-final-1.png](/img/zipkin-final-1.png){:height="270px" width="858px"}

    select "reservation-client" and click "Find Traces", and click on the first result: 

    ![zipkin-final-2.png](/img/zipkin-final-2.png){:height="243px" width="860px"}

    we can observe how our request time spent between the `reservation-client` and `reservation-service`. 


## Conclusion & Congratulations!
That's it! Hope you learnt a thing or two from this series. If you have any questions, feel free to ask 'em in the Disqus forum below. 



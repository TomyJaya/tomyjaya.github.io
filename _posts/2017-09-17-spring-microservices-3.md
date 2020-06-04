---
layout:     post
title:      "Microservices with Spring Boot & Spring Cloud"
subtitle:   "Part III: Setting Up Eureka Service (Service Discovery & Registration)"
date:       2017-09-17 12:00:00
author:     "Tomy Jaya"
header-img: "img/post-bg-06.jpg"
tags:
tags:
- spring
- spring-boot
- microservices
- spring-cloud
- cloud
- eureka
---


# Setting Up Eureka Service (Service Discovery & Registration)

## Introduction 

One of the primary goals of maintaining a distributed systems architecture is to achieve fault-tolerance and load balancing. Hence, it's no surprise that we usually have multiple redundant services essentially doing the same thing. We can use DNS to keep track of the address and port of these services; however, DNS is fairly static and in the microservices world, and especially in the cloud, services come and go fairly frequently. That is why we need another separate service, commonly referred as *Service Discovery and Registration*, to keep track of a registry of available services and their corresponding locations. Since [Spring Cloud Netflix](https://cloud.spring.io/spring-cloud-netflix/) has integrated Netflix's [Eureka](https://github.com/Netflix/eureka) easily to Spring Boot, we'll use Eureka for as our *Service Discovery and Registration*. Two primary components make up Spring Cloud Eureka:

1. `spring-cloud-eureka-server`: a super easy way to set up your Service Discovery and Registration *server* (Eureka Server)
2. `spring-cloud-starter-eureka`: easy plug-in dependency to set up your *client* to register as a service and fetch list of services from the registry

>In summary, Service Registry is a logical mapping between a service id/ name to a hostname and port combination. 

## Setting up Eureka Service's Configuration
In the existing config's git: 

```
vim eureka-service.properties

### Add the below to eureka-service.properties
server.port=${PORT:8761}

eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
## End of eureka-service.properties

git add eureka-service.properties
git commit -m "Add eureka-service.properties"
```

*NOTE*: the reason we set `eureka.client.register-with-eureka` and `eureka.client.fetch-registry` to `false` is that because eureka-server has  eureka-client built-in in as well. Hence, we need to prevent the client for registering and querying itself (causing unnecessary infinite loop). 


## Setting up Eureka Service

1. Go to https://start.spring.io/

2. Type `eureka-service` as the "Artifact" name, add `Eureka Server` & `Config Client` dependencies, and "Generate Project": 

    ![eureka-service-spring-initializr-screenshot.png](/img/eureka-service-spring-initializr-screenshot.png){:height="343px" width="840px"}

*NOTE*: the reason we need `Config Client` is such that we can connect to our previously set up `config-service` to retrieve centrally hosted configuration

3. Open the project in your favorite Java IDE (mine is IntelliJ)

4. Add `@EnableEurekaServer` in `src/main/java/EurekaServiceApplication`:

    ```java
    package com.example.eurekaservice;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
    
    @EnableEurekaServer
    @SpringBootApplication
    public class EurekaServiceApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(EurekaServiceApplication.class, args);
        }
    }
    ```

5. in `src/main/resources`, create a new file called `bootstrap.properties`, add the below:

    ```properties
    # tell config service what is the name of the service and corresponding config to retrieve
    
    spring.application.name=eureka-service
    
    # tell the location the location of config server
    
    spring.cloud.config.uri=http://localhost:8888
    ```

    *Note*: Why `bootstrap.properties`? Refer to: https://stackoverflow.com/questions/32997352/what-is-the-diference-between-putting-a-property-on-application-yml-or-bootstrap

6. Run `EurekaServiceApplication` and voila! Your Eureka server is started!

7. You can check the following URL to see the Eureka's Dashboard: http://localhost:8761/

    ![eureka-service-result](/img/eureka-service-result.png){:height="400px" width="850px"}

## THINGS TO NOTE:

1. Again, it's just so convenient to bootstrap a fully functioning Service Discovery & Registration server by just a single annotation! 
2. You will see that our `eureka-service` is running on port 8761. How does it now that? From the `server.port` we configured in `eureka-service.properties`. And how did our service know how to talk to our `config-service`? We configured it to have `Config Client` (aka `spring-cloud-starter-config` in the `pom.xml`) and tell the URL of the config service in `bootstrap.properties`. 
3. Right now, we haven't registered any service to Eureka yet. In a later tutorial, you'll see how you can register your service to Eureka and see how Eureka keeps track of it when it's terminated.  

## Reference
http://www.baeldung.com/spring-cloud-netflix-eureka

## Next Up
[Setting Up Auth Service](/2017/09/17/spring-microservices-4/)
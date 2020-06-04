---
layout:     post
title:      "Microservices with Spring Boot & Spring Cloud"
subtitle:   "Part V: Setting Up Zipkin Service"
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

# Setting Up Zipkin Service 

## Introduction

When it comes to microservices, tracing what happens becomes enormously more complex. You can't just put a breakpoint and debug your way through just like how you would do with a monolith. Fortunately, folks at Twitter came up with [Zipkin](https://zipkin.io/). Used and popularized by engineers Netflix and Uber, Zipkin became sort of a de facto choice for distributed system's tracing solution. And just like with other popular libraries, integrating Zipkin to Spring Boot is a matter of plug and play.  

There are 2 components to use Zipkin: 
1. Zipkin Server (`zipkin-server` & `zipkin-autoconfigure-ui`): This will be our service which stores all our tracing data and visualize them.
2. Zipkin Client (`zipkin-cloud-starter-zipkin`): This will send stream our tracing data from a client's service.  

## Setting up Zipkin Service's Configuration

In the existing config's git: 
```
vim zipkin-service.properties

### Add the below to zipkin-service.properties
server.port=${PORT:9411}
spring.datasource.initialize = false
spring.sleuth.enabled = false
zipkin.store.type = mem
## End of zipkin-service.properties

git add zipkin-service.properties
git commit -m "Add zipkin-service.properties"
```

## Setting up Zipkin Service

1. Go to https://start.spring.io/

2. Type `zipkin-service` as the "Artifact" name, add `Zipkin UI`, `Eureka Discovery` and `Config Client` dependencies, and "Generate Project": 

    ![zipkin-service-spring-initializr-screenshot.png](/img/zipkin-service-spring-initializr-screenshot.png){:height="343px" width="840px"}

3. Open the project in your favorite Java IDE (mine is IntelliJ)

4. Add the below zipkin server's dependency to `pom.xml`: 

    ```xml
    <dependency>
        <groupId>io.zipkin.java</groupId>
        <artifactId>zipkin-server</artifactId>
    </dependency>
    ```

5. Add `@EnableDiscoveryClient` and `@EnableZipkinServer` in `src/main/java/ZipkinServiceApplication`:

    ```java

    package com.example.zipkinservice;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
    import zipkin.server.EnableZipkinServer;

    @SpringBootApplication
    @EnableDiscoveryClient
    @EnableZipkinServer
    public class ZipkinServiceApplication {
    
	    public static void main(String[] args) {
		    SpringApplication.run(ZipkinServiceApplication.class, args);
	    }
    }
    ```

6. in `src/main/resources`, create a new file called `bootstrap.properties`, add the below:

    ```properties
    # tell config service what is the name of the service and corresponding config to retrieve
    
    spring.application.name=zipkin-service
    
    # tell the location the location of config server

    spring.cloud.config.uri=http://localhost:8888
    ```

7. Run `ZipkinServiceApplication` and voila! Your Zipkin server is started!

8. Go to http://localhost:9411/zipkin/ to visit Zipkin's UI. There is currently nothing, but soon, we'll register our microservices in the next posts and be able to trace requests going through the pipeline. 

    ![zipkin-service-result](/img/zipkin-service-result.png){:height="212px" width="956px"}

## Conclusion:
We have now set up our zipkin service which can diagrammatically show a timeline of requests going through our microservices. In the next posts, you will see how we can use Zipkin Client to feed data to our server.  

## Next Up
[Setting Up Hystrix Dashboard](/2017/09/17/spring-microservices-6/)
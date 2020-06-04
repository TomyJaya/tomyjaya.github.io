---
layout:     post
title:      "Microservices with Spring Boot & Spring Cloud"
subtitle:   "Part VI: Setting Up Hystrix Dashboard"
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

# Setting Up Hystrix Dashboard

## Introduction

Failure is inevitable. It's even more so in distributed systems. As services communicate remotely to each other, failure points/ surface area exponentially increase. Ergo, designing for fault tolerance is a top priority when building microservices. 

[Hystrix](https://github.com/Netflix/Hystrix) is a library developed by Netflix which can help us build a more resilient system. One architecture pattern we'll borrow from Hystrix is called "Circuit Breaker" pattern. Just like the normal household circuit breaker, microservices' circuit breaker identifies fault in our system and fallback to a default implementation. In this post, we'll only set up the dashboard to monitor our circuit breakers. The implementation of the actual circuit breaker will be discussed in subsequent post.  

## Setting up Hystrix Dashboard's Configuration

In the existing config's git: 
```
vim hystrix-dashboard.properties

### Add the below to hystrix-dashboard.properties
server.port=${PORT:8010}
## End of hystrix-dashboard.properties

git add hystrix-dashboard.properties
git commit -m "Add hystrix-dashboard.properties"
```

## Setting up Hystrix Dashboard

1. Go to https://start.spring.io/

2. Type `hystrix-dashboard` as the "Artifact" name, add `Eureka Discovery`, `Config Client`, and `Hystrix Dashboard` dependencies, and "Generate Project": 
    ![hystrix-dashboard-spring-initializr-screenshot.png](/img/hystrix-dashboard-spring-initializr-screenshot.png){:height="343px" width="840px"}

3. Open the project in your favorite Java IDE (mine is IntelliJ)

4. Add `@EnableHystrixDashboard` in `src/main/java/HystrixDashboardApplication`:

    ```java
    package com.example.hystrixdashboard;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;
    
    @SpringBootApplication
    @EnableHystrixDashboard
    public class HystrixDashboardApplication {
    
    	public static void main(String[] args) {
		    SpringApplication.run(HystrixDashboardApplication.class, args);
	    }
    }
    ```

6. in `src/main/resources`, create a new file called `bootstrap.properties`, add the below:

    ```properties
    # tell config service what is the name of the service and corresponding config to retrieve
    
    spring.application.name=hystrix-dashboard
    
    # tell the location the location of config server

    spring.cloud.config.uri=http://localhost:8888
    ```

7. Run `HystrixDashboardApplication` and voila! Your Hystrix Dashboard is started!

8. Go to http://localhost:8010/hystrix.html to visit our Hystrix Dashboard. There is currently nothing, but soon, we'll register our circuit breaker in the next posts and be able to see our circuit breakers in action.  

    ![hystrix-dashboard-result](/img/hystrix-dashboard-result.png){:height="509px" width="829px"}

## Conclusion:
We have now set up our Hystrix Dashboard service which can visually show our system's fault tolerance/ resiliency. In the next posts, you will see how we can use Hystrix's circuit breaker to feed the stream of data to be visualized by this dashboard.  

## Next Up
[Setting Up Reservation Service](/2017/09/17/spring-microservices-7/)
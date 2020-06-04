---
layout:     post
title:      "Microservices with Spring Boot & Spring Cloud"
subtitle:   "Part I: The Big Picture"
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


# Microservices with Spring Boot & Spring Cloud (I: The Big Picture)

## Foreword
My colleague and I went to [Cloud Native Java](https://www.youtube.com/watch?v=I053xBvPhSY) talk by [Josh Long](https://twitter.com/starbuxman) and were immediately blown away. We've been hosting our own custom service discovery/ registration, config server, authentication services for our distributed systems, whilst Spring & Netflix had actually built, perfected, and open-sourced all these cool things and we could have just used them for free. What's more, using them is just a matter of including several annotations! 

So, I decided to learn more about Spring Boot & Spring Cloud. 
Here are the main references I took: 
* [Cloud Native Java - Safari Video](https://www.safaribooksonline.com/library/view/cloud-native-java/9780134690377/)
* [Cloud Native Java Workshop - Github](https://github.com/joshlong/cloud-native-workshop)
* [Cloud Native Java Book](https://www.amazon.com/Cloud-Native-Java-Designing-Resilient/dp/1449374646)

The following blog series is therefore, the by-product of my learning process (aka my notes). 

## Disclaimer
Again, all credits go to the original workshop author Josh Long. This blog series is just a step-by-step guide, or if you will, written re-representation of his work. 

## Pre-requisites
To follow along, you'll need to know: 
* Java 8 
* Spring Boot
* Basic idea of what microservices architecture is

## The Big Picture
The following diagram might be overwhelming at first. But this is what we're gonna get to at the end of this blog series. 

![spring-microservices-architecture](/img/spring-microservices-architecture.png){:height="600px" width="800px"}

Things to note: 

1. The components in yellow are the business domain specific components. In this sample, reservation is our domain (bounded context).
2. Josh Long used RabbitMQ in the original tutorial. I substituted RabbitMQ with a more popular distributed streams, Kafka which in turns need zookeeper for coordination. 
3. We are just simulating one edge service (i.e. HTML5 browsers). In real life, you'll probably have multiple edge services; one for each target device. 

## Setting Up Kafka

Follow Step 1 and Step 2 here: https://kafka.apache.org/quickstart

I saved the commands to start Zookeeper & Kafka here: 

```bash
# change the directory to where you put your kafka installation
cd /Users/macpro/apps/kafka_2.11-0.11.0.1
# start zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties
# start kafka
bin/kafka-server-start.sh config/server.properties
```

## Next Up
[Setting Up Config Service](/2017/09/17/spring-microservices-2/)

---
title: Spring Cloud Gateway
date: 2021-08-20 11:16:55.626
updated: 2021-12-05 21:03:18.517
url: /archives/springcloudgateway
categories: 
- 框架
tags: 
- spring gateway
---



# Gateway 简介
Gateway是在Spring生态系统之上构建的API网关服务，基于Spring 5，Spring Boot 2和 Project Reactor等技术。Gateway旨在提供一种简单而有效的方式来对API进行路由，以及提供一些强大的过滤器功能， 例如：熔断、限流、重试等。

Spring Cloud Gateway 具有如下特性：
- 基于Spring Framework 5, Project Reactor 和 Spring Boot 2.0 进行构建；
- 动态路由：能够匹配任何请求属性；
- 可以对路由指定 Predicate（断言）和 Filter（过滤器）；
- 集成Hystrix的断路器功能；
- 集成 Spring Cloud 服务发现功能；
- 易于编写的 Predicate（断言）和 Filter（过滤器）；
- 请求限流功能；
- 支持路径重写。

# 相关概念
- Route（路由）：路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由；
- Predicate（断言）：指的是Java 8 的 Function Predicate。 输入类型是Spring框架中的ServerWebExchange。 这使开发人员可以匹配HTTP请求中的所有内容，例如请求头或请求参数。如果请求与断言相匹配，则进行路由；
- Filter（过滤器）：指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前后对请求进行修改。

# 使用
1. 引入依赖
```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

2. 在application.yml中进行配置
```java
spring:
  cloud:
    gateway:
      routes:
        - id: path_route #路由的ID
          uri: ${service-url.user-service}/user/{id} #匹配后路由地址
          predicates: # 断言，路径相匹配的进行路由
            - Path=/user/{id}
          filters:
            - StripPrefix=1 #去掉部分URL路径
```
# Route Predicate 的使用
多个Route Predicate工厂可以进行组合，下面我们来介绍下一些常用的Route Predicate。
## Before Route Predicate
在指定时间之后的请求会匹配该路由。
```java
spring:
  cloud:
    gateway:
      routes:
        - id: before_route
          uri: ${service-url.user-service}
          predicates:
            - Before=2019-09-24T16:30:00+08:00[Asia/Shanghai]
```
## Between Route Predicate
在指定时间区间内的请求会匹配该路由。
```java
spring:
  cloud:
    gateway:
      routes:
        - id: before_route
          uri: ${service-url.user-service}
          predicates:
            - Between=2019-09-24T16:30:00+08:00[Asia/Shanghai], 2019-09-25T16:30:00+08:00[Asia/Shanghai]
```
## Cookie Route Predicate
带有指定Cookie的请求会匹配该路由。
```java
spring:
  cloud:
    gateway:
      routes:
        - id: cookie_route
          uri: ${service-url.user-service}
          predicates:
            - Cookie=username,macro 
```
## Header Route Predicate
带有指定请求头的请求会匹配该路由。
```java
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: ${service-url.user-service}
        predicates:
        - Header=X-Request-Id, \d+
```
## Host Route Predicate
带有指定Host的请求会匹配该路由。
```java
spring:
  cloud:
    gateway:
      routes:
        - id: host_route
          uri: ${service-url.user-service}
          predicates:
            - Host=**.macrozheng.com
```
## Method Route Predicate
发送指定方法的请求会匹配该路由。

## Path Route Predicate 常用
发送指定路径的请求会匹配该路由。
```java
spring:
  cloud:
    gateway:
      routes:
        - id: path_route
          uri: ${service-url.user-service}/user/{id}
          predicates:
            - Path=/user/{id}
```
## Query Route Predicate
带指定查询参数的请求可以匹配该路由。
## RemoteAddr Route Predicate
从指定远程地址发起的请求可以匹配该路由。
## Weight Route Predicate
使用权重来路由相应请求，以下表示有80%的请求会被路由到localhost:8201，20%会被路由到localhost:8202。
```java
spring:
  cloud:
    gateway:
      routes:
      - id: weight_high
        uri: http://localhost:8201
        predicates:
        - Weight=group1, 8
      - id: weight_low
        uri: http://localhost:8202
        predicates:
        - Weight=group1, 2
```
# Route Filter 的使用
路由过滤器可用于修改进入的HTTP请求和返回的HTTP响应

## AddRequestParameter GatewayFilter
给请求添加参数的过滤器。
```java
spring:
  cloud:
    gateway:
      routes:
        - id: add_request_parameter_route
          uri: http://localhost:8201
          filters:
            - AddRequestParameter=username, macro
          predicates:
            - Method=GET
```
## StripPrefix GatewayFilter
对指定数量的路径前缀进行去除的过滤器。
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: strip_prefix_route
        uri: http://localhost:8201
        predicates:
        - Path=/user-service/**
        filters:
        - StripPrefix=2

```

以上配置会把以/user-service/开头的请求的路径去除两位，通过curl工具使用以下命令进行测试。
```bash
http://localhost:9201/user-service/a/user/1
```
相当于发起该请求：
```bash
http://localhost:8201/user/1
```
## PrefixPath GatewayFilter
与StripPrefix过滤器恰好相反，会对原有路径进行增加操作的过滤器。
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefix_path_route
        uri: http://localhost:8201
        predicates:
        - Method=GET
        filters:
        - PrefixPath=/user
```
以上配置会对所有GET请求添加/user路径前缀
## Hystrix GatewayFilter
Hystrix 过滤器允许你将断路器功能添加到网关路由中，使你的服务免受级联故障的影响，并提供服务降级处理。
***TODO***
## RequestRateLimiter GatewayFilter
RequestRateLimiter 过滤器可以用于限流，使用RateLimiter实现来确定是否允许当前请求继续进行，如果请求太大默认会返回HTTP 429-太多请求状态。
***TODO***

# 结合注册中心使用
在结合注册中心使用过滤器的时候，我们需要注意的是uri的协议为lb，这样才能启用Gateway的负载均衡功能。
```yaml
spring:
  cloud:
    gateway:
      routes: #配置路由路径
        - id: mall-auth
          uri: lb://mall-auth
          predicates:
            - Path=/mall-auth/**
          filters:
            - StripPrefix=1 #去掉部分URL路径
```
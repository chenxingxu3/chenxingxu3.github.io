---
layout:     post
title:      15.SpringCloud学习笔记
subtitle:   Gateway
date:       2020-08-16
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloud
---
# 15.SpringCloud学习笔记--Gateway

Zuul参考：

https://github.com/Netflix/zuul

Gateway参考：

https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/

https://spring.io/projects/spring-cloud-gateway

## 什么是 Gateway 网关

### 概述

![](/img-post/2020-08-16-springcloud/15-01.jpg)

Gateway 是在 Spring 生态系统之上构建的 API 网关服务，基于 Spring 5, Spring Boot 2 和 Project Reactor 等技术。

Gateway 旨在提供一 种简单而有效的方式来对 API 进行路由，以吸提供一些强大的过滤器功能，例如: **熔断、限流、重试**等。

Spring Cloud Gateway 是 Spring Cloud 的一个全新项目，基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式。

Spring Cloud Gateway 作为 Spring Cloud 生态系统中的网关，目标是替代 Zuul。在 Spring Cloud2.0 以上版本中，没有对新版本的 Zuul2.0 以上最新高性能版本进行集成，仍然还是使用的 Zuul1.x 非 Reactor 模式的老版本。而为了提升网关的性能，Spring Cloud Gateway 是基于 **WebFlux 框架实现的**，而 WebFlux 框架底层则使用了**高性能的 Reactor 模式通信框架 Netty**

SpringCloud Gateway 的目标提供统一的路由方式且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控 / 指标，和限流。

Spring Cloud Gateway 使用的是 Webflux 中的 reactor-netty 响应式编程组件，底层使用了 Netty 通讯框架。

### Gateway 网关能干什么

- 反向代理

- 鉴权

- 流量控制

- 熔断

- 日志监控

![](/img-post/2020-08-16-springcloud/15-03.jpg)

## GateWay非阻塞异步模型

### Gateway 特性：

- 基于 SpringFramework5, Project Reactor 和 SpringBoot2.0 进行构建
- 动态路由：能够匹配任何请求属性
- 可以对路由指定 Predicate (断言) 和 Filter (过滤器)
- 集成 Hystrix 的断路器功能
- 集成 SpringCloud 服务发现功能
- 易于编写的 Predicate (断言) 和 Filter (过滤器)
- 请求限流功能
- 支持路径重写

### 为什么使用 Gateway

因为 Zuul 1.0 已经进入了维护阶段，而且 Gateway 是 SpringCloud 团队研发的。

Gateway 是基于异步非阻塞模型上进行开发的，性能方面不需要担心。虽然 Netflix 早就发布了最新的 Zuul2.x, 但 Spring Cloud 似乎没有整合计划。而且 Netflix 相关组件都宣布进入维护期，截至目前，Gateway 是最理想的网关选择。

### Gateway 与 Zuul 的区别

在 SpringCloud Finchley 正式版之前，SpringCloud 推荐的网关是 Netflix 提供的 Zuul:

1、Zuul 1.x, 是一个基于阻塞 I/ O 的 API Gateway

2、Zuul 1.x 基于 Servlet2.5 使用阻塞架构它不支持任何长连接 (如 WebSocket)，Zuul 的设计模式和 Nginx 较像，每次 I/O 操作都是从工作线程中选择一个执行，请求线程被阻塞到工作线程完成，但是差别是 Nginx 用 C++ 实现，Zuul 用 Java 实现，而 JVM 本身会有第一次加载较慢的情况，使得 Zuul 的性能相对较差。

3、Zuul 2.x 理念更先进，想基于 Netty 非阻塞和支持长连接，但 SpringCloud 目前还没有整合。Zuul2.x 的性能较 Zuul1.x 有较大提升。在性能方面，根据官方提供的基准测试，Gateway 的 RPS (每秒请求数) 是 Zuul 的 1.6 倍。

4、Gateway 建立在 SpringFramework5、Project Reactor 和 SpringBoot2 之上，使用非阻塞 APl。

5、Gateway 还支持 WebSocket，并且与 Spring 紧密集成拥有更好的开发体验。

### Zuul 1.x 模型

Spring Cloud中所集成的Zuul版本，采用的是Tomcat容器,使用的是传统的Servlet IO处理模型。

Servlet的生命周期：servlet由servlet container进行生命周期管理。

container启动时构造servlet对象并调用servlet init()进行初始化;

container运行时接受请求，并为每个请求分配一 个线程(一般从线程池中获取空闲线程)然后调用service(）。

container关闭时调用servlet destory()销毁servlet;

![](/img-post/2020-08-16-springcloud/15-04.jpg)

上述模型的缺点：

servlet堤一个简单的网络IO模型, 当请求进入servlet container时, servlet container就会为其绑定一个线程， 在**并发不高的场景下**这种模型是适用的。但是一旦高并发(比如用jemeter压测)，线程数量就会上涨，而线程资源代价是昂贵的（上下文切换，内存消耗大），严重影响请求的处理时间。在一些简单业务场景下，不希望为每个request分配一个线程, 只需要1个或几个线程就能应对极大并发的请求，这种业务场景下servlet模型没有优势。

所以Zuul 1.X是基于servlet之上的一个阻塞式处理模型，即spring实现了处理所有request请求的一个servlet (DispatcherServlet) 并由该servlet阻塞式处理。所以Springcloud Zuul无法摆脱servlet模型的弊端

### Gateway 模型

#### WebFlux

传统的Web框架，比如说: struts2, springmvc等都是基于Servlet API与Servlet容器基础之上运行的。

但是在Servlet3.1之后有了异步非阻塞的支持。而**WebFlux是一个 典型非阻塞异步的框架**,它的核心是基于Reactor的相关API实现的。相对于传统的web框架来说，它可以运行在诸如Netty, Undertow及支持Servlet3.1的容器上。非阻塞式+函数式编程(Spring5必须使用java8)。

Spring WebFlux是Spring 5.0引入的**新的响应式框架**,区别于Spring MVC,它**不需要依赖Servlet API,它是完全异步非阻塞的，并且基于Reactor来实现响应式流规范。**

参考：

https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#spring-webflux

## Gateway工作流程

### Spring Cloud Gateway 三大核心概念

**Route(路由)**

路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由。

**Predicate（断言）**

参考的是java8的java.util.function.Predicate开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由。

**Filter(过滤)**

指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求路由前或者之后对请求进行修改。

![](/img-post/2020-08-16-springcloud/15-05.jpg)

web请求通过一些匹配条件，定位到真正的服务节点。并在这个转发过程的前后，进行一些精细化控制。

**predicate是匹配条件**；

而 **filter 可以理解为一个无所不能的拦截器**。有了这两个元素，再加上目标url，就可以实现一个具体的路由了。

### Gateway工作流程

![](/img-post/2020-08-16-springcloud/15-06.png)

> Clients make requests to Spring Cloud Gateway. If the Gateway Handler Mapping determines that a request matches a route, it is sent to the Gateway Web Handler. This handler runs the request through a filter chain that is specific to the request. The reason the filters are divided by the dotted line is that filters can run logic both before and after the proxy request is sent. All “pre” filter logic is executed. Then the proxy request is made. After the proxy request is made, the “post” filter logic is run.

- 客户端向Spring Cloud Gateway发出请求。然后在Gateway Handler Mapping中找到与请求相匹配的路由,将其发送到Gateway Web Handler。
- Handler再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。
- 过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前( “pre” )或之后( “post” )执行业务逻辑。**Filter在"pre" 类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等,在"post"类型的过滤器中可以做响应内容、响应头的修改，日志的输出,流量监控等**有着非常重要的作用。

核心逻辑：路由转发 + 执行过滤器链

参考：

https://cloud.spring.io/spring-cloud-gateway/reference/html/#gateway-how-it-works

## Gateway9527搭建

### 1、建Module

cloud-gateway-gateway9527

### 2、POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>demo.yangxu.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-gateway-gateway9527</artifactId>

    <dependencies>
        <!--gateway-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <!--eureka-client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>demo.yangxu.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!--一般基础配置类-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```



### 3、YML

不想暴露8001端口，在8001外面套一层9527

![](/img-post/2020-08-16-springcloud/15-07.jpg)

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由

eureka:
  instance:
    hostname: cloud-gateway-service
    instance-id: gateway9527
    prefer-ip-address: true
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```



### 4、业务类

无

### 5、主启动类

**GateWayMain9527**

```java
package demo.yangxu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class GateWayMain9527 {
    public static void main(String[] args) {
        SpringApplication.run(GateWayMain9527.class,args);
    }
}

```



### 6、测试

启动

- cloud-eureka-server7001（EurekaMain7001）

- cloud-provider-payment8001（PaymentMain8001）

- cloud-gateway-gateway9527（GateWayMain9527）

http://eureka7001.com:7001

添加网关前：

http://localhost:8001/payment/get/31

http://localhost:8001/payment/lb

添加网关后：

http://localhost:9527/payment/get/31

http://localhost:9527/payment/lb

## Gateway配置路由的两种方式

Gateway网关路由有两种配置方式：

- 在配置文件yml中配置（推荐）

- 代码中注入RouteLocator的Bean

### 在配置文件yml中配置（推荐）

参考`Gateway9527搭建`

### 代码中注入RouteLocator的Bean

官网提供的案例：

Example 12. GatewayConfig.java

```java
RemoteAddressResolver resolver = XForwardedRemoteAddressResolver
    .maxTrustedIndex(1);

...

.route("direct-route",
    r -> r.remoteAddr("10.1.1.1", "10.10.1.1/24")
        .uri("https://downstream1")
.route("proxied-route",
    r -> r.remoteAddr(resolver, "10.10.1.1", "10.10.1.1/24")
        .uri("https://downstream2")
)
```

Example 26. Application.java

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("circuitbreaker_route", r -> r.path("/consumingServiceEndpoint")
            .filters(f -> f.circuitBreaker(c -> c.name("myCircuitBreaker").fallbackUri("forward:/inCaseOfFailureUseThis"))
                .rewritePath("/consumingServiceEndpoint", "/backingServiceEndpoint")).uri("lb://backing-service:8088")
        .build();
}
```

参考：https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/#spring-cloud-circuitbreaker-filter-factory

### 案例

通过9527网关访问到外网的百度新闻网址

在 cloud-gateway-gateway9527 新建一个 GateWayConfig 类

```java
package demo.yangxu.springcloud.config;

import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class GateWayConfig {

    /**
     * 配置了一个id为path_route_yangxu的路由规则
     * 当访问地址http://localhost:9527/guonei时会自动转发到地址http://news.baidu.com/guonei
     * @param routeLocatorBuilder
     * @return
     */
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder){
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();

        routes.route("path_route_yangxu",
                r -> r.path("/guonei")
                        .uri("http://news.baidu.com/guonei")).build();

        return routes.build();
    }

    /**
     * 配置了一个id为path_route_yangxu2的路由规则
     * 当访问地址http://localhost:9527/guoji时会自动转发到地址http://news.baidu.com/guoji
     * @param routeLocatorBuilder
     * @return
     */
    @Bean
    public RouteLocator customRouteLocator2(RouteLocatorBuilder routeLocatorBuilder){
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();

        routes.route("path_route_yangxu2",
                r -> r.path("/guoji")
                        .uri("http://news.baidu.com/guoji")).build();

        return routes.build();
    }
}

```

测试

http://localhost:9527/guonei

http://localhost:9527/guoji

## GateWay配置动态路由

如果同时有 8001，8002，怎么去调用多个微服务呢？

把 application.yml 中的 uri 改成 `lb://微服务名称`，就可以了，lb 代表的是 load balance（负载均衡）

例如：`uri: lb://cloud-payment-service`

默认情况下Gateway会根据注册中心注册的服务列表，以注册中心上微服务名为路径**创建动态路由进行转发，从而实现动态路由的功能**。

### 启动

- cloud-eureka-server7001（EurekaMain7001）

- cloud-provider-payment8001（PaymentMain8001）

- cloud-provider-payment8002（PaymentMain8002）

### YML

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由


eureka:
  instance:
    hostname: cloud-gateway-service
    instance-id: gateway9527
    prefer-ip-address: true
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

### 测试

http://localhost:9527/payment/lb

8001 / 8002 两个端口切换

## GateWay常用的Predicate

![](/img-post/2020-08-16-springcloud/15-08.jpg)

Spring Cloud Gateway将路由匹配作为Spring WebFlux HandlerMapping基础架构的一部分。

**Spring Cloud Gateway包括许多内置的Route Predicate工厂**。所有这些Predicate都与HTTP请求的不同属性匹配。多个RoutePredicate工厂可以进行组合。

Spring Cloud Gateway创建Route对象时，使用RoutePredicateFactory创建Predicate对象, Predicate 对象可以赋值给Route。Spring Cloud Gateway包含许多内置的Route Predicate Factories。

所有这些谓词都匹配HTTP请求的不同属性。**多种谓词工厂可以组合，并通过逻辑and。**

通过这些断言的配置，就可以控制 http 请求哪些有效，及在什么条件下有效。

**Predicate就是为了实现一组匹配规则，让请求过来找到对应的Route进行处理。**

### After Route Predicate

路由在[设置时间]之后有效

```yaml
- After=2020-07-30T11:29:15.380+08:00[Asia/Shanghai]  #路由什么时候起效
```

获取时间串的代码：

```java
package demo.yangxu.time;

import java.time.ZonedDateTime;

public class T2 {
    public static void main(String[] args) {
        ZonedDateTime zonedDateTime = ZonedDateTime.now();
        System.out.println(zonedDateTime);
    }
}

```

配置YML实例：

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
            - After=2020-07-30T11:29:15.380+08:00[Asia/Shanghai]


eureka:
  instance:
    hostname: cloud-gateway-service
    instance-id: gateway9527
    prefer-ip-address: true
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

### Before Route Predicate

路由在[设置时间]之前有效

```yaml
- Before=2020-07-30T11:29:15.380+08:00[Asia/Shanghai] #路由有效截止日期
```

配置YML实例：

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
            - Before=2020-07-30T11:29:15.380+08:00[Asia/Shanghai] #路由有效截止日期


eureka:
  instance:
    hostname: cloud-gateway-service
    instance-id: gateway9527
    prefer-ip-address: true
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

### Between Route Predicate

路由在[设置时间]之间有效

```yaml
- Between=2020-07-29T10:59:34.102+08:00[Asia/Shanghai],2020-07-30T10:59:34.102+08:00[Asia/Shanghai] #有效时间段
```

配置YML实例：

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
            - Between=2020-07-29T10:59:34.102+08:00[Asia/Shanghai],2020-07-30T10:59:34.102+08:00[Asia/Shanghai] #有效时间段


eureka:
  instance:
    hostname: cloud-gateway-service
    instance-id: gateway9527
    prefer-ip-address: true
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```



### Cookie Route Predicate

Cookie Route Predicate需要两个参数, 一个是Cookie name ,一个是正则表达式。路由规则会通过获取对应的Cookie name值和正则表达式去匹配，如果**匹配上就会执行路由，如果没有匹配上则不执行**

```yaml
- Cookie=username,yangxu  #请求信息携带Cookie信息与此匹配
```

配置YML实例：

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
            - Cookie=username,yangxu  #请求信息携带Cookie信息与此匹配


eureka:
  instance:
    hostname: cloud-gateway-service
    instance-id: gateway9527
    prefer-ip-address: true
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

使用 cURL 进行测试

```bash
#不带Cookie的请求
curl http://192.168.25.146:9527/payment/lb
```

错误信息：

```json
{
  "timestamp": "2020-07-30T03:56:31.301+0000",
  "path": "/payment/lb",
  "status": 404,
  "error": "Not Found",
  "message": null,
  "requestId": "09ef3bde"
}
```

```bash
#带Cookie的请求
curl http://192.168.25.146:9527/payment/lb --cookie "username=yangxu"
```

返回正确结果：

```
8002
```



### Header Route Predicate

两个参数: 一个是属性名称和一个正则表达式，这个属性值和正则表达式匹配则执行。

```yaml
- Header=X-Request-Id,\d+ #请求头要有X-Request- Id属性并且值为整数的正则表达式
```

配置YML实例：

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
            - Header=X-Request-Id,\d+ #请求头要有X-Request- Id属性并且值为整数的正则表达式


eureka:
  instance:
    hostname: cloud-gateway-service
    instance-id: gateway9527
    prefer-ip-address: true
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

使用 cURL 进行测试

```bash
#Header为负数的请求
curl http://192.168.25.146:9527/payment/lb -H "X-Request-Id:-123"
```

错误信息：

```json
{
  "timestamp": "2020-07-30T04:07:17.548+0000",
  "path": "/payment/lb",
  "status": 404,
  "error": "Not Found",
  "message": null,
  "requestId": "1aabdf20"
}
```

```bash
#Header为正数的请求
curl http://192.168.25.146:9527/payment/lb -H "X-Request-Id:123"
```

返回正确结果：

```
8002
```



### Host Route Predicate

主机号相同才匹配

```yaml
- Host=**.eureka.com	#主机号相同才匹配
```

配置YML实例：

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
            - Host=**.eureka.com	#主机号相同才匹配


eureka:
  instance:
    hostname: cloud-gateway-service
    instance-id: gateway9527
    prefer-ip-address: true
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

使用 cURL 进行测试

```bash
#错误的主机号
curl http://192.168.25.146:9527/payment/lb
```

错误信息：

```json
{
  "timestamp": "2020-07-30T04:13:27.309+0000",
  "path": "/payment/lb",
  "status": 404,
  "error": "Not Found",
  "message": null,
  "requestId": ""b7becbc3"
}
```

```bash
#正确的主机号
curl http://192.168.25.146:9527/payment/lb -H "Host:www.eureka.com"
```

返回正确结果：

```
8002
```



### Method Route Predicate

请求方式相同则匹配

```yaml
- Method=GET #请求方式相同则匹配
```

配置YML实例：

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
            - Method=GET #请求方式相同则匹配


eureka:
  instance:
    hostname: cloud-gateway-service
    instance-id: gateway9527
    prefer-ip-address: true
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```



### Path Route Predicate

断言，路径相匹配的进行路由

```yaml
- Path=/payment/lb/**   #断言,路径相匹配的进行路由
```

配置YML实例：

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由


eureka:
  instance:
    hostname: cloud-gateway-service
    instance-id: gateway9527
    prefer-ip-address: true
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```



### Query Route Predicate

要有参数名称并且是正整数才能路由

```yaml
- Query=username, \d+ #要有参数名称并且是正整数才能路由
```

配置YML实例：

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
            - Query=username, \d+ #要有参数名称并且是正整数才能路由


eureka:
  instance:
    hostname: cloud-gateway-service
    instance-id: gateway9527
    prefer-ip-address: true
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

使用 cURL 进行测试

```bash
#错误的参数
curl http://192.168.25.146:9527/payment/lb?username=-31
```

错误信息：

```json
{
  "timestamp": "2020-07-30T04:18:30.214+0000",
  "path": "/payment/lb",
  "status": 404,
  "error": "Not Found",
  "message": null,
  "requestId": ""35f74dd1"
}
```

```bash
#正确的参数
curl http://192.168.25.146:9527/payment/lb?username=31
```

返回正确结果：

```
8002
```



参考：

https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/#gateway-request-predicates-factories

## GateWay的Filter

>GatewayFilter Factories
>
>Route filters allow the modification of the incoming HTTP request or outgoing HTTP response in some manner. Route filters are scoped to a particular route. Spring Cloud Gateway includes many built-in GatewayFilter Factories.

路由过滤器可用于修改进入的HTTP请求和返回的HTTP响应,路由过滤器只能指定路由进行使用。 

Spring Cloud Gateway内置了多种路由过滤器,他们都由GatewayFilter的工厂类来产生。

参考：

https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/#gatewayfilter-factories

### 生命周期

只有两个，**pre：在业务逻辑之前执行；post：在业务逻辑之后执行**

### 单一过滤器----GatewayFilter

>5.1. The AddRequestHeader GatewayFilter Factory
>5.2. The AddRequestParameter GatewayFilter Factory
>5.3. The AddResponseHeader GatewayFilter Factory
>5.4. The DedupeResponseHeader GatewayFilter Factory
>5.5. The Hystrix GatewayFilter Factory
>5.6. Spring Cloud CircuitBreaker GatewayFilter Factory
>5.7. The FallbackHeaders GatewayFilter Factory
>5.8. The MapRequestHeader GatewayFilter Factory
>5.9. The PrefixPath GatewayFilter Factory
>5.10. The PreserveHostHeader GatewayFilter Factory
>5.11. The RequestRateLimiter GatewayFilter Factory
>5.12. The RedirectTo GatewayFilter Factory
>5.13. The RemoveHopByHopHeadersFilter GatewayFilter Factory
>5.14. The RemoveRequestHeader GatewayFilter Factory
>5.15. RemoveResponseHeader GatewayFilter Factory
>5.16. The RemoveRequestParameter GatewayFilter Factory
>5.17. The RewritePath GatewayFilter Factory
>5.18. RewriteLocationResponseHeader GatewayFilter Factory
>5.19. The RewriteResponseHeader GatewayFilter Factory
>5.20. The SaveSession GatewayFilter Factory
>5.21. The SecureHeaders GatewayFilter Factory
>5.22. The SetPath GatewayFilter Factory
>5.23. The SetRequestHeader GatewayFilter Factory
>5.24. The SetResponseHeader GatewayFilter Factory
>5.25. The SetStatus GatewayFilter Factory
>5.26. The StripPrefix GatewayFilter Factory
>5.27. The Retry GatewayFilter Factory
>5.28. The RequestSize GatewayFilter Factory
>5.29. Modify a Request Body GatewayFilter Factory
>5.30. Modify a Response Body GatewayFilter Factory
>5.31. Default Filters

YML配置示例：

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          filters:
            AddRequestParameter=X-Request-Id, 1024 #过滤器工厂会在匹配的请求头加上-对请求头，名称为X-Request- Id值为1024
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
            - Query=username, \d+ #要有参数名称并且是正整数才能路由


eureka:
  instance:
    hostname: cloud-gateway-service
    instance-id: gateway9527
    prefer-ip-address: true
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

参考：

https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/#gatewayfilter-factories



### 全局过滤器----GlobalFilter

>6.1. Combined Global Filter and GatewayFilter Ordering
>6.2. Forward Routing Filter
>6.3. The LoadBalancerClient Filter
>6.4. The ReactiveLoadBalancerClientFilter
>6.5. The Netty Routing Filter
>6.6. The Netty Write Response Filter
>6.7. The RouteToRequestUrl Filter
>6.8. The Websocket Routing Filter
>6.9. The Gateway Metrics Filter
>6.10. Marking An Exchange As Routed

参考：

https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/#global-filters

### 自定义过滤器

**自定义全局GlobalFilter：**

编写Java 类实现 GlobalFilter和Orderd 接口

```java
package demo.yangxu.springcloud.filter;

import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import java.util.Date;

@Component
@Slf4j
public class MyLogGateWayFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("***********come in MyLogGateWayFilter:  "+new Date());

        String uname = exchange.getRequest().getQueryParams().getFirst("uname");

        if(uname == null){
            log.info("*******用户名为null，非法用户，o(╥﹏╥)o");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        //加载过滤器的优先级
        //数字越小，优先级越高
        return 0;
    }
}

```

启动

- cloud-eureka-server7001（EurekaMain7001）
- cloud-provider-payment8001（PaymentMain8001）
- cloud-provider-payment8002（PaymentMain8002）
- cloud-gateway-gateway9527（GateWayMain9527）

测试：

正确地址示例：

http://localhost:9527/payment/lb?uname=z3

错误地址示例：

http://localhost:9527/payment/lb

http://localhost:9527/payment/lb?uname123=z3
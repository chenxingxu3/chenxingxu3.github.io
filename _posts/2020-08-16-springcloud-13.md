---
layout:     post
title:      13.SpringCloud学习笔记
subtitle:   OpenFeign
date:       2020-08-16
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloud
---
# 13.SpringCloud学习笔记--OpenFeign

## OpenFeign是什么

### 概述

>[Feign](https://github.com/OpenFeign/feign) is a declarative web service client. It makes writing web service clients easier. To use Feign create an interface and annotate it. It has pluggable annotation support including Feign annotations and JAX-RS annotations. Feign also supports pluggable encoders and decoders. Spring Cloud adds support for Spring MVC annotations and for using the same `HttpMessageConverters` used by default in Spring Web. Spring Cloud integrates Ribbon and Eureka, as well as Spring Cloud LoadBalancer to provide a load-balanced http client when using Feign.

>Declarative REST Client: Feign creates a dynamic implementation of an interface decorated with JAX-RS or Spring MVC annotations

Feign 是一个声明式 WebService 客户端。使用 Feign 能让编写 Web Service 客户端更加简单。

它的使用方法是**定义一个服务接口然后在上面添加注解**。Feign 也支持可插拔式的编码器和解码器。Spring Cloud 对 Feign 进行了封装，使其支持了 Spring MVC 标准注解和 HttpMessageConverters。Feign 可以与 Eureka 和 Ribbon 组合使用以支持负载均衡。


参考：

https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/#spring-cloud-openfeign

https://github.com/spring-cloud/spring-cloud-openfeign

### Feign能干什么

Feign旨在使编写Java HTTP客户端变得更容易。

前面在**使用 Ribbon+RestTemplate 时，利用 RestTemplate 对 http 请求的封装处理，形成了一套模板化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。**所以，Feign 在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。**在 Feign 的实现下，我们只需创建一个接口并使用注解的方式来配置它（以前是 Dao 接口上面标注 Mapper 注解，现在是一个微服务接口上面标注衣一个 Feign 注解即可），即可完成对服务提供方的接口绑定**，简化了使用 Spring Cloud Ribbon 时，自动封装服务调用客户端的开发量。

**Feign 集成了 Ribbon**

利用 Ribbon 维护了 Payment 的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与 Ribbon 不同的是，通过 feign 只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用。

### Feign 和 OpenFeign 两者区别

| Feign                                                        | OpenFeign                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Feign 是 Spring Cloud 组件中的一个轻量级 Restful 的 HTTP 服务客户端，Feign 内置了 Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。Feign 的使用方式是：使用 Feign 的注解定义接口，调用这个接口，就可以调用服务注册中心的服务。 | OpenFeign 是 Spring Cloud 在 Feign 的基础上支持了 Spring MVC 的注解，如 @RequestMapping 等等。OpenFeign 的 @FeignClient 可以解析 Spring MVC 的 @RequestMapping 注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。 |

```xml
<!--openfeign-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

<!--feign-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

## OpenFeign服务调用

### 接口+注解

微服务调用接口+@FeignClient

![](/img-post/2020-08-16-springcloud/13-01.jpg)

### 1、建module

cloud-consumer-feign-order80

### 2、改pom

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

    <artifactId>cloud-consumer-feign-order80</artifactId>

    <dependencies>
        <!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--eureka client-->
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
        <!--web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--一般基础通用配置-->
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



### 3、写yml

```yaml
server:
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
```



### 4、主启动类

demo.yangxu.springcloud.OrderFeignMain80

```java
package demo.yangxu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableFeignClients
public class OrderFeignMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderFeignMain80.class, args);
    }
}

```



### 5、业务层

#### 业务逻辑接口+@FeignClient配置调用provider服务

新建PaymentFeignService接口并新增注解@FeignClient

demo.yangxu.springcloud.service.PaymentFeignService

```java
package demo.yangxu.springcloud.service;

import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentFeignService {

    @GetMapping(value = "/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
}

```



#### 控制层Controller

demo.yangxu.springcloud.controller.OrderFeignController

```java
package demo.yangxu.springcloud.controller;

import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import demo.yangxu.springcloud.service.PaymentFeignService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderFeignController {

    @Resource
    private PaymentFeignService paymentFeignService;

    @GetMapping(value = "/consumer/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
        return paymentFeignService.getPaymentById(id);
    }
}

```



### 6、验证测试

启动

- cloud-eureka-server7001
- cloud-eureka-server7002
- cloud-provider-payment8001
- cloud-provider-payment8002
- cloud-consumer-feign-order80

http://localhost/consumer/payment/get/31

**Feign自带负载均衡配置项**

## OpenFeign超时控制

### 超时设置

故意设置超时来演示出错的情况

服务提供方8001故意写暂停程序paymentFeignTimeout

demo.yangxu.springcloud.controller.PaymentController

```java
package demo.yangxu.springcloud.controller;

import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import demo.yangxu.springcloud.service.PaymentService;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;
import java.util.List;
import java.util.concurrent.TimeUnit;

@RestController
@Slf4j
public class PaymentController {

    @Resource
    private PaymentService paymentService;

    @Resource
    private DiscoveryClient discoveryClient;

    @Value("${server.port}")
    private String serverPort;

    @PostMapping(value = "/payment/create")
    public CommonResult create(@RequestBody Payment payment){
        int result = paymentService.create(payment);
        log.info("*****插入结果：" + result);

        if(result > 0){
            return new CommonResult(200,"插入数据库成功,serverPort:"+serverPort,result);
        }else{
            return new CommonResult(444,"插入数据库失败",null);
        }
    }

    @GetMapping(value = "/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
        Payment payment = paymentService.getPaymentById(id);
        log.info("*****查询结果：" + payment + "热部署测试123456");

        if(payment != null){
            return new CommonResult(200,"查询成功,serverPort:"+serverPort,payment);
        }else{
            return new CommonResult(444,"没有对应记录，查询ID："+id,null);
        }
    }

    @GetMapping(value="/payment/discovery")
    public Object discovery(){
        //服务清单列表
        List<String> services = discoveryClient.getServices();
        for (String element : services) {
            log.info("*******element: "+element);
        }

        //一个微服务下的全部实例
        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        for (ServiceInstance instance : instances) {
            log.info(instance.getServiceId()+"\t"+instance.getHost()+"\t"+instance.getPort()+"\t"+instance.getUri());
        }

        return this.discoveryClient;
    }

    @GetMapping(value = "/payment/lb")
    public String getPaymentLB(){
        return serverPort;
    }

    @GetMapping(value = "/payment/feign/timeout")
    public String paymentFeignTimeout(){
        try {
            TimeUnit.SECONDS.sleep(3);
        }catch (InterruptedException e){
            e.printStackTrace();
        }

        return serverPort;
    }

}

```

服务消费方80的PaymentFeignService添加超时方法paymentFeignTimeout

demo.yangxu.springcloud.service.PaymentFeignService

```java
package demo.yangxu.springcloud.service;

import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentFeignService {

    @GetMapping(value = "/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);

    @GetMapping(value = "/payment/feign/timeout")
    public String paymentFeignTimeout();
}

```

服务消费方80的OrderFeignController添加超时方法paymentFeignTimeout

demo.yangxu.springcloud.controller.OrderFeignController

```java
package demo.yangxu.springcloud.controller;

import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import demo.yangxu.springcloud.service.PaymentFeignService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderFeignController {

    @Resource
    private PaymentFeignService paymentFeignService;

    @GetMapping(value = "/consumer/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
        return paymentFeignService.getPaymentById(id);
    }

    @GetMapping(value = "/consumer/payment/feign/timeout")
    public String paymentFeignTimeout(){
        // OpenFeign客户端一般默认等待1秒钟
        return paymentFeignService.paymentFeignTimeout();
    }
}

```

测试

http://localhost:8001/payment/feign/timeout

http://localhost/consumer/payment/feign/timeout

```
This application has no explicit mapping for /error, so you are seeing this as a fallback.

Sat Jul 25 12:03:39 CST 2020
There was an unexpected error (type=Internal Server Error, status=500).
Read timed out executing GET http://CLOUD-PAYMENT-SERVICE/payment/feign/timeout
```



OpenFeign默认等待1秒，超时后报错

默认 Feign 客户端只等待一秒钟， 但是服务端处理需要超过 1 秒钟，导致 Feign 客户端不想等待了，直接返回报错。为了避免这样的情况，有时候我们需要设置 Feign 客户端的超时控制。

### YML文件开启OpenFeign客户端超时控制

```yaml
server:
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
#设置feign客户端超时时间(OpenFeign默认支持ribbon)
ribbon:
  #指建立连接后从服务端读取到可用资源所用的时间
  ReadTimeout: 5000
  #建立连接所用的时间，适用于网络状况正常的情况下，两端连接所需要的时间
  ConnectTimeout: 5000
```

测试

http://localhost/consumer/payment/feign/timeout

## OpenFeign日志增强

Feign 提供了日志打印功能，可以通过配置来调整日志级别，从而了解 Feign 中 Http 请求的细节。
简单来说就是对接口的调用情况进行监控和输出。

### 日志级别

- **NONE：默认的，不显示任何日志**
- **BASIC：仅记录请求方法、URL、响应状态码及执行时间**
- **HEADERS：除了 BASIC 中定义的信息之外，还有请求和响应的头信息**
- **FULL：除了 HEADERS 中定义的信息之外，还有请求和响应的正文及元数据**

### 配置日志Bean

demo.yangxu.springcloud.config.FeignConfig

```java
package demo.yangxu.springcloud.config;

import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignConfig {

    @Bean
    Logger.Level feignLoggerLevel(){
        //默认的，不显示任何日志
        //return Logger.Level.NONE;

        //仅记录请求方法、URL、响应状态码及执行时间
        //return Logger.Level.BASIC;

        //除了 BASIC 中定义的信息之外，还有请求和响应的头信息
        //return Logger.Level.HEADERS;

        //除了 HEADERS 中定义的信息之外，还有请求和响应的正文及元数据
        return Logger.Level.FULL;
    }
}

```

### 配置application.yml

```yaml
server:
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
#设置feign客户端超时时间(OpenFeign默认支持ribbon)
ribbon:
  #指建立连接后从服务端读取到可用资源所用的时间
  ReadTimeout: 5000
  #建立连接所用的时间，适用于网络状况正常的情况下，两端连接所需要的时间
  ConnectTimeout: 5000

logging:
  level:
    # feign日志以什么级别监控哪个接口
    demo.yangxu.springcloud.service.PaymentFeignService: debug

```

### 测试

http://localhost/consumer/payment/get/31

```
2020-07-25 12:26:30.713 DEBUG 1624 --- [p-nio-80-exec-1] d.y.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] ---> GET http://CLOUD-PAYMENT-SERVICE/payment/get/31 HTTP/1.1
2020-07-25 12:26:30.714 DEBUG 1624 --- [p-nio-80-exec-1] d.y.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] ---> END HTTP (0-byte body)
2020-07-25 12:26:31.806 DEBUG 1624 --- [p-nio-80-exec-1] d.y.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] <--- HTTP/1.1 200 (1091ms)
2020-07-25 12:26:31.807 DEBUG 1624 --- [p-nio-80-exec-1] d.y.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] connection: keep-alive
2020-07-25 12:26:31.808 DEBUG 1624 --- [p-nio-80-exec-1] d.y.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] content-type: application/json
2020-07-25 12:26:31.808 DEBUG 1624 --- [p-nio-80-exec-1] d.y.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] date: Sat, 25 Jul 2020 04:26:31 GMT
2020-07-25 12:26:31.809 DEBUG 1624 --- [p-nio-80-exec-1] d.y.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] keep-alive: timeout=60
2020-07-25 12:26:31.809 DEBUG 1624 --- [p-nio-80-exec-1] d.y.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] transfer-encoding: chunked
2020-07-25 12:26:31.809 DEBUG 1624 --- [p-nio-80-exec-1] d.y.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] 
2020-07-25 12:26:31.813 DEBUG 1624 --- [p-nio-80-exec-1] d.y.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] {"code":200,"message":"查询成功,serverPort:8001","data":{"id":31,"serial":"aaabbb01"}}
2020-07-25 12:26:31.814 DEBUG 1624 --- [p-nio-80-exec-1] d.y.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] <--- END HTTP (90-byte body)
```


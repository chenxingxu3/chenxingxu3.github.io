---
layout:     post
title:      14.SpringCloud学习笔记
subtitle:   Hystrix
date:       2020-08-16
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloud
---
# 14.SpringCloud学习笔记--Hystrix

## Hystrix是什么

**分布式系统面临的问题**

复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候（异常故障）将不可避免出现损失的情况。

![](/img-post/2020-08-16-springcloud/14-01.jpg)

**服务雪崩**

分布式系统环境下，通常会有很多层的服务调用。由于网络原因或自身的原因，服务一般无法保证 100% 可用。如果一个服务出现了问题，调用这个服务就会出现线程阻塞的情况，此时若有大量的请求涌入，就会出现多条线程阻塞等待，进而导致服务瘫痪。

多个微服务之间调用的时候，假设微服务 A 调用微服务 B 和微服务 C, 微服务 B 和微服务 C 又调用其它的微服务，这就是所谓的 "扇出"。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务 A 的调用就会占用越来越多的系统资源，进而引起系统崩溃，就是服务故障的 “雪崩效应”.

**对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和。**比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。

所以，**通常当你发现一个模块下的某个实例失败后，这时候这个模块依然还会接收流量，然后这个有问题的模块还调用了其他的模块，这样就会发生级联故障，或者叫雪崩。**

要防止雪崩的扩散，我们就要做好服务的容错：保护自己不被猪队友拖垮的一些措施。

常见的容错方案：隔离、超时、限流、熔断、降级

**Hystrix**

Hystrix 是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等。

Hystrix 能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。

“断路器” 本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控 (类似熔断保险丝)，向调用方返回一个符合预期的、可处理的备选响应 (FallBack) ，而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

**目前：Hystrix 已经停更，后面会使用阿里的 sentinel，但是 Hystrix 仍然有值得学习的思想和设计。**

## Hystrix停更进维

https://github.com/NetFlix/Hystrix/wiki/How-To-Use

>Hystrix is no longer in active development, and is currently in maintenance mode.

https://github.com/NetFlix/Hystrix

## Hystrix的服务降级熔断限流概念初步认识

### 服务降级

服务器忙，请稍后再试，不让客户端等待并立刻返回一个友好提示，fallback

发生服务降级的情况：

- 程序运行异常
- 超时
- 服务熔断出发服务降级
- 线程池 / 信号量打满也会导致服务降级

### 服务熔断

类比保险丝，达到最大服务访问后，直接拒绝访问，拉闸限电，然后调用服务降级的方法并返回友好提示。

服务降级 -> 进而熔断 -> 恢复调用链路

### 服务限流

秒杀高并发等操作，严禁一窝蜂的过来拥挤，大家排队，一秒钟N个，有序进行

## Hystrix支付微服务构建

### 1、建module

cloud-provider-hystrix-payment8001

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

    <artifactId>cloud-provider-hystrix-payment8001</artifactId>

    <dependencies>
        <!--hystrix-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <!--eureka client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
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
        <dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
            <groupId>demo.yangxu.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
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
  port: 8001

spring:
  application:
    name: cloud-provider-hystrix-payment

eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
      defaultZone: http://eureka7001.com:7001/eureka
```



### 4、主启动

demo.yangxu.springcloud.PaymentHystrixMain8001

```java
package demo.yangxu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class PaymentHystrixMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentHystrixMain8001.class, args);
    }
}

```



### 5、业务类

**Service**

demo.yangxu.springcloud.service.PaymentService

```java
package demo.yangxu.springcloud.service;

import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

@Service
public class PaymentService {
    /**
     * 正常访问，肯定OK
     * @param id
     * @return
     */
    public String paymentInfo_OK(Integer id)
    {
        return "线程池:  "+Thread.currentThread().getName()+"  paymentInfo_OK,id:  "+id+"\t"+"O(∩_∩)O哈哈~";
    }

    public String paymentInfo_TimeOut(Integer id)
    {
        int timeNumber = 3;
        try { TimeUnit.SECONDS.sleep(timeNumber); } catch (InterruptedException e) { e.printStackTrace(); }
        return "线程池:  "+Thread.currentThread().getName()+" paymentInfo_TimeOut,id:  "+id+"\t"+"O(∩_∩)O哈哈~"+"  耗时(秒): "+timeNumber;
    }
}

```

**Controller**

demo.yangxu.springcloud.controller.PaymentController

```java
package demo.yangxu.springcloud.controller;

import demo.yangxu.springcloud.service.PaymentService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
@Slf4j
public class PaymentController {

    @Resource
    private PaymentService paymentService;

    @Value("${server.port}")
    private String severPort;

    @GetMapping(value = "/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id){
        String result = paymentService.paymentInfo_OK(id);
        log.info("*****result: "+result);
        return result;
    }

    @GetMapping(value = "/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
        String result = paymentService.paymentInfo_TimeOut(id);
        log.info("*****result: "+result);
        return result;
    }
}

```



### 6、正常测试

#### 启动eureka7001

#### 启动cloud-provider-hystrix-payment8001

#### 访问

success的方法

http://localhost:8001/payment/hystrix/ok/31

```
线程池: http-nio-8001-exec-10 paymentInfo_OK,id: 31 O(∩_∩)O哈哈~
```

每次调用耗费3秒

http://localhost:8001/payment/hystrix/timeout/31

```
线程池: http-nio-8001-exec-7 paymentInfo_TimeOut,id: 31 O(∩_∩)O哈哈~ 耗时(秒): 3
```

上述module均ok

以上述为根基平台，从正确 -> 错误 -> 降级熔断 -> 恢复

## JMeter高并发压测后卡顿

 JMeter 工具下载地址：

http://jmeter.apache.org/download_jmeter.cgi

 JMeter 工具历史版本下载地址：

https://archive.apache.org/dist/jmeter/binaries/

下载解压后，进入 bin 目录，双击 jmeter.bat 即可启动。

将JMeter 修改为中文版：

在apache-jmeter-2.13\bin\jmeter.properties文件中添加以下内容：

```properties
language=zh_CN
```

进行压力测试：

1、测试计划->添加->Threads(Users)->线程组

![](/img-post/2020-08-16-springcloud/14-02.jpg)

2、线程数200，1秒钟1个，循环100次，共200*100=20000个并发

![](/img-post/2020-08-16-springcloud/14-03.jpg)

3、保存

![](/img-post/2020-08-16-springcloud/14-04.jpg)

4、线程组20200726->添加->Sampler->HTTP请求

![](/img-post/2020-08-16-springcloud/14-05.jpg)

5、填写以下信息

![](/img-post/2020-08-16-springcloud/14-06.jpg)

6、保存，启动

![](/img-post/2020-08-16-springcloud/14-07.jpg)

7、查看效果

此时 2 万个线程访问的是 `http://localhost:8001/payment/hystrix/timeout/31`

但是此时访问 `http://localhost:8001/payment/hystrix/ok/31` 会发现不能立即加载出来，有一定的延迟。

因为大家都是同一个微服务，此时 timeout 压力大，服务器集中去处理这 2 万个线程了，所以导致 ok 这边被拖累。tomcat的默认工作线程被打满了，没有多余的线程来分解压力和处理。

## 订单微服务调用支付服务出现卡顿

### 1、建module

cloud-consumer-feign-hystrix-order80

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

    <artifactId>cloud-consumer-feign-hystrix-order80</artifactId>

    <dependencies>
        <!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--hystrix-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
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
      defaultZone: http://eureka7001.com:7001/eureka/

spring:
  devtools:
    restart:
      poll-interval: 3000ms
      quiet-period: 2999ms

```



### 4、主启动

OrderHystrixMain80

```java
@SpringBootApplication
@EnableFeignClients
public class OrderHystrixMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderHystrixMain80.class,args);
    }
}
```



### 5、业务类

**Service**

PaymentHystrixService

```java
package demo.yangxu.springcloud.service;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@Component
@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT")
public interface PaymentHystrixService {

    @GetMapping(value = "/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id);

    @GetMapping(value = "/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id);
}

```

**Controller**

OrderHystirxController

```java
package demo.yangxu.springcloud.controller;

import com.netflix.hystrix.contrib.javanica.annotation.DefaultProperties;
import demo.yangxu.springcloud.service.PaymentHystrixService;
import javafx.beans.DefaultProperty;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderHystirxController {
    
    @Resource
    private PaymentHystrixService paymentHystrixService;

    @GetMapping(value = "/consumer/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id){
        String result = paymentHystrixService.paymentInfo_OK(id);
        return result;
    }

    @GetMapping(value = "/consumer/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
        String result = paymentHystrixService.paymentInfo_TimeOut(id);
        return result;
    }
}

```



### 6、正常测试

启动：

- cloud-eureka-server7001（EurekaMain7001）
- cloud-provider-hystrix-payment8001（PaymentHystrixMain8001）
- cloud-consumer-feign-hystrix-order80（OrderHystrixMain80）

http://localhost/consumer/payment/hystrix/ok/31

```
线程池: http-nio-8001-exec-8 paymentInfo_OK,id: 31 O(∩_∩)O哈哈~
```



### 7、高并发测试

2W个线程压 `http://localhost:8001/payment/hystrix/timeout/31`

![](/img-post/2020-08-16-springcloud/14-08.jpg)

![](/img-post/2020-08-16-springcloud/14-09.jpg)

消费端80微服务再去访问正常的ok微服务8001地址 `http://localhost/consumer/payment/hystrix/ok/32`

消费者80的访问出现卡顿和延迟，甚至因为访问超时报错。

```
Read timed out executing GET http://CLOUD-PROVIDER-HYSTRIX-PAYMENT/payment/hystrix/ok/31
```

故障现象和导致原因：

8001同一层次的其他接口服务被困死，因为tomcat线程池里面的工作线程已经被挤占完毕。80此时调用8001，客户端访问响应缓慢，甚至因为访问超时报错。

正因为有上述故障或不佳表现，才有降级 / 容错 / 限流等技术诞生。

## 降级容错解决的维度要求

### 现象及对策

超时导致服务器变慢：超时不再等待

出错（宕机或程序运行出错）：出错要有兜底

### 解决

对方服务（8001）超时了，调用者（80）不能一直卡死等待，必须有服务降级

对方服务（8001）down机了，调用者（80）不能一直卡死等待，必须有服务降级

对方服务（8001）ok，调用者（80）自己出故障或有自我要求（自己的等待时间小于服务提供者），自己降级处理

## Hystrix之服务降级支付侧fallback

### 降级配置

@HystrixCommand

### 8001先从自身找问题

设置自身调用超时时间的峰值，峰值内可以正常运行，超过了需要有兜底的方法处理，作服务降级fallback

### 8001fallback

#### 超时异常

**业务类启用**

PaymentService

```java
package demo.yangxu.springcloud.service;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

@Service
public class PaymentService {
    /**
     * 正常访问，肯定OK
     * @param id
     * @return
     */
    public String paymentInfo_OK(Integer id)
    {
        return "线程池:  "+Thread.currentThread().getName()+"  paymentInfo_OK,id:  "+id+"\t"+"O(∩_∩)O哈哈~";
    }

    @HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler", commandProperties = {
            //3秒内访问正常，超过3秒交由fallbackMethod处理
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="3000")
    })
    public String paymentInfo_TimeOut(Integer id)
    {
        int timeNumber = 5;
        try { TimeUnit.SECONDS.sleep(timeNumber); } catch (InterruptedException e) { e.printStackTrace(); }
        return "线程池:  "+Thread.currentThread().getName()+" paymentInfo_TimeOut,id:  "+id+"\t"+"O(∩_∩)O哈哈~"+"  耗时(秒): "+timeNumber;
    }

    public String paymentInfo_TimeOutHandler(Integer id){
        return "线程池:  "+ Thread.currentThread().getName()+
                " 8001系统繁忙或者运行报错，请稍后再试,id:  "+id+
                "\t"+"o(╥﹏╥)o";
    }
}

```

**@HystrixCommand报异常后如何处理**

一旦调用服务方法失败并抛出了错误信息后，会自动调用@HystrixCommand标注好的fallbackMethod调用类中的指定方法。

**主启动类激活**

添加新注解@EnableCircuitBreaker

PaymentHystrixMain8001

```java
package demo.yangxu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class PaymentHystrixMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentHystrixMain8001.class, args);
    }
}

```

**测试**

启动：

- cloud-eureka-server7001（EurekaMain7001）
- cloud-provider-hystrix-payment8001（PaymentHystrixMain8001）

访问 `http://localhost:8001/payment/hystrix/timeout/31`

```
线程池: HystrixTimer-1 8001系统繁忙或者运行报错，请稍后再试,id: 31 o(╥﹏╥)o
```

#### 计算异常

**业务类**

PaymentService

```java
package demo.yangxu.springcloud.service;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

@Service
public class PaymentService {
    /**
     * 正常访问，肯定OK
     * @param id
     * @return
     */
    public String paymentInfo_OK(Integer id)
    {
        return "线程池:  "+Thread.currentThread().getName()+"  paymentInfo_OK,id:  "+id+"\t"+"O(∩_∩)O哈哈~";
    }

    @HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler", commandProperties = {
            //3秒内访问正常，超过3秒交由fallbackMethod处理
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="3000")
    })
    public String paymentInfo_TimeOut(Integer id)
    {
        //int timeNumber = 5;
        //try { TimeUnit.SECONDS.sleep(timeNumber); } catch (InterruptedException e) { e.printStackTrace(); }
        //return "线程池:  "+Thread.currentThread().getName()+" paymentInfo_TimeOut,id:  "+id+"\t"+"O(∩_∩)O哈哈~"+"  耗时(秒): "+timeNumber;
        int age = 10/0;
        return "线程池:  "+Thread.currentThread().getName()+" paymentInfo_TimeOut,id:  "+id+"\t"+"O(∩_∩)O哈哈~"+"  演示计算异常";
    }

    public String paymentInfo_TimeOutHandler(Integer id){
        return "线程池:  "+ Thread.currentThread().getName()+
                " 8001系统繁忙或者运行报错，请稍后再试,id:  "+id+
                "\t"+"o(╥﹏╥)o";
    }
}

```

**测试**

启动：

- cloud-eureka-server7001（EurekaMain7001）
- cloud-provider-hystrix-payment8001（PaymentHystrixMain8001）

访问 `http://localhost:8001/payment/hystrix/timeout/31`

```
线程池: hystrix-PaymentService-1 8001系统繁忙或者运行报错，请稍后再试,id: 31 o(╥﹏╥)o
```



#### 总结

只要是当前方案不可用了，就做服务降级，兜底的方案都是paymentInfo_TimeOutHandler

## Hystrix之服务降级订单侧fallback

80订单微服务，也可以更好地保护自己，进行客户端降级保护。一般服务降级放在客户端。

热部署方式对java代码的改动明显，但是对@HystrixCommand内属性的修改建议重启微服务。

### 1、写yml

```yaml
server:
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/

feign:
  hystrix:
    enabled: true

spring:
  devtools:
    restart:
      poll-interval: 3000ms
      quiet-period: 2999ms

```

### 2、主启动

```java
package demo.yangxu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.hystrix.EnableHystrix;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableFeignClients
@EnableHystrix
public class OrderHystrixMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderHystrixMain80.class,args);
    }
}

```

### 3、业务类（演示超时）

**Controller**

OrderHystirxController

```java
package demo.yangxu.springcloud.controller;

import com.netflix.hystrix.contrib.javanica.annotation.DefaultProperties;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import demo.yangxu.springcloud.service.PaymentHystrixService;
import javafx.beans.DefaultProperty;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderHystirxController {

    @Resource
    private PaymentHystrixService paymentHystrixService;

    @GetMapping(value = "/consumer/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id){
        String result = paymentHystrixService.paymentInfo_OK(id);
        return result;
    }

    @GetMapping(value = "/consumer/payment/hystrix/timeout/{id}")
    @HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod",commandProperties = {
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="1500")
    })
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
        String result = paymentHystrixService.paymentInfo_TimeOut(id);
        return result;
    }

    public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id)
    {
        return "我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
    }
}

```

修改8001中的demo.yangxu.springcloud.service.PaymentService#paymentInfo_TimeOut

```java
@HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler", commandProperties = {
        //设置访问正常的时间，超时交由fallbackMethod处理
        @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="5000")
})
public String paymentInfo_TimeOut(Integer id)
{
    int timeNumber = 3000;
    try { TimeUnit.MILLISECONDS.sleep(timeNumber); } catch (InterruptedException e) { e.printStackTrace(); }
    return "线程池:  "+Thread.currentThread().getName()+" paymentInfo_TimeOut,id:  "+id+"\t"+"O(∩_∩)O哈哈~"+"  耗时(毫秒): "+timeNumber;
}
```

测试：

启动

- cloud-eureka-server7001（EurekaMain7001）
- cloud-provider-hystrix-payment8001（PaymentHystrixMain8001）
- cloud-consumer-feign-hystrix-order80（OrderHystrixMain80）

访问 `http://localhost/consumer/payment/hystrix/timeout/31`

```
我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o
```

### 4、业务类（演示计算异常）

**Controller**

修改demo.yangxu.springcloud.controller.OrderHystirxController#paymentInfo_TimeOut

```java
@GetMapping(value = "/consumer/payment/hystrix/timeout/{id}")
@HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod",commandProperties = {
        @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="1500")
})
public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
    int age = 10/0;
    String result = paymentHystrixService.paymentInfo_TimeOut(id);
    return result;
}
```

测试：

启动

- cloud-eureka-server7001（EurekaMain7001）
- cloud-provider-hystrix-payment8001（PaymentHystrixMain8001）
- cloud-consumer-feign-hystrix-order80（OrderHystrixMain80）

访问 `http://localhost/consumer/payment/hystrix/timeout/31`

```
我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o
```

## Hystrix之全局服务降级DefaultProperties

### 目前问题

每个业务方法对应一个兜底的方法，代码膨胀

全局通用的和自定义的需要分开

### 解决问题

#### 配置全局服务降级

![](/img-post/2020-08-16-springcloud/14-10.jpg)

1: N 除了个别重要核心业务有专属的兜底方法，其他普通的业务可以通过@DefaultProperties(defaultFallback = "")跳转到统一的处理结果页面。

通用的和共享的各自分开，避免了代码膨胀，合理减少了代码量。

**在80上配置Controller**

OrderHystirxController

```java
package demo.yangxu.springcloud.controller;

import com.netflix.hystrix.contrib.javanica.annotation.DefaultProperties;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import demo.yangxu.springcloud.service.PaymentHystrixService;
import javafx.beans.DefaultProperty;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
@Slf4j
@DefaultProperties(defaultFallback = "payment_Global_FallbackMethod")
public class OrderHystirxController {

    @Resource
    private PaymentHystrixService paymentHystrixService;

    @GetMapping(value = "/consumer/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id){
        String result = paymentHystrixService.paymentInfo_OK(id);
        return result;
    }

    @GetMapping(value = "/consumer/payment/hystrix/timeout/{id}")
    @HystrixCommand
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
        int age = 10/0;
        String result = paymentHystrixService.paymentInfo_TimeOut(id);
        return result;
    }

    public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id) {
        return "我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
    }

    // 全局fallback方法
    public String payment_Global_FallbackMethod() {
        return "Global异常处理信息，请稍后再试，/(ㄒoㄒ)/~~";
    }

}

```

测试：

启动

- cloud-eureka-server7001（EurekaMain7001）
- cloud-provider-hystrix-payment8001（PaymentHystrixMain8001）
- cloud-consumer-feign-hystrix-order80（OrderHystrixMain80）

访问 `http://localhost/consumer/payment/hystrix/timeout/31`

```
Global异常处理信息，请稍后再试，/(ㄒoㄒ)/~~
```

## Hystrix之通配服务降级FeignFallback

为Feign客户端定义的接口添加一个服务降级处理的实现类即可实现解耦。

要处理的常见异常：

- 运行时异常
- 超时
- 宕机

### 修改cloud-consumer-feign-hystrix-order80

根据cloud-consumer-feign-hystrix-order80已有的PaymentHystrixService接口，新建一个类PaymentFallbackService实现该接口，统一为接口里面的方法进行异常处理。

PaymentFallbackService类实现PaymentFeignClientService接口

```java
package demo.yangxu.springcloud.service;

import org.springframework.stereotype.Component;

@Component
public class PaymentFallbackService implements PaymentHystrixService {
    @Override
    public String paymentInfo_OK(Integer id) {
        return "-----PaymentFallbackService fall back-paymentInfo_OK ,o(╥﹏╥)o";
    }

    @Override
    public String paymentInfo_TimeOut(Integer id) {
        return "-----PaymentFallbackService fall back-paymentInfo_TimeOut ,o(╥﹏╥)o";
    }
}

```



### 修改YML

```yaml
server:
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/

feign:
  hystrix:
    enabled: true

spring:
  devtools:
    restart:
      poll-interval: 3000ms
      quiet-period: 2999ms

```



### PaymentFeignClientService接口

```java
package demo.yangxu.springcloud.service;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@Component
@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT", fallback = PaymentFallbackService.class)
public interface PaymentHystrixService {

    @GetMapping(value = "/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id);

    @GetMapping(value = "/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id);
}

```



### 测试

启动

- cloud-eureka-server7001（EurekaMain7001）
- cloud-consumer-feign-hystrix-order80（OrderHystrixMain80）

关闭

- cloud-provider-hystrix-payment8001（PaymentHystrixMain8001）

访问 `http://localhost/consumer/payment/hystrix/ok/31`

```
-----PaymentFallbackService fall back-paymentInfo_OK ,o(╥﹏╥)o
```

此时服务端provider已经宕机了，但是因为做了服务降级处理，使得客户端在服务端不可用的情况下，也会返回提示信息。

## Hystrix之服务熔断理论

### 断路器

可以理解为家里的保险丝

### 熔断机制概述

熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回”错误”的响应信息。

当检测到该节点微服务调用响应正常后恢复调用链路。

在SpringCloud框架里熔断机制通过Hystrix实现。Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，缺省是5秒内20次调用失败就会启动熔断机制。熔断机制的注解是@HystrixCommand。

![](/img-post/2020-08-16-springcloud/14-11.jpg)

参考：https://martinfowler.com/bliki/CircuitBreaker.html

## Hystrix之服务熔断案例

### Circuit Breaker

The following diagram shows how a `HystrixCommand` or `HystrixObservableCommand` interacts with a [`HystrixCircuitBreaker`](http://netflix.github.io/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCircuitBreaker.html) and its flow of logic and decision-making, including how the counters behave in the circuit breaker.

![](/img-post/2020-08-16-springcloud/14-12.jpg)

The precise way that the circuit opening and closing occurs is as follows:

1. Assuming the volume across a circuit meets a certain threshold (`HystrixCommandProperties.circuitBreakerRequestVolumeThreshold()`)...
2. And assuming that the error percentage exceeds the threshold error percentage (`HystrixCommandProperties.circuitBreakerErrorThresholdPercentage()`)...
3. Then the circuit-breaker transitions from `CLOSED` to `OPEN`.
4. While it is open, it short-circuits all requests made against that circuit-breaker.
5. After some amount of time (`HystrixCommandProperties.circuitBreakerSleepWindowInMilliseconds()`), the next single request is let through (this is the `HALF-OPEN` state). If the request fails, the circuit-breaker returns to the `OPEN` state for the duration of the sleep window. If the request succeeds, the circuit-breaker transitions to `CLOSED` and the logic in **1.** takes over again.

参考：https://github.com/Netflix/Hystrix/wiki/How-it-Works#CircuitBreaker

### 1、修改cloud-provider-hystrix-payment8001

**PaymentService**

10秒(sleepWindowInMilliseconds)中10次访问(requestVolumeThreshold)有60%的请求失败(errorThresholdPercentage)后跳闸。

```java
package demo.yangxu.springcloud.service;

import cn.hutool.core.util.IdUtil;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.PathVariable;

import java.util.concurrent.TimeUnit;

@Service
public class PaymentService {
    /**
     * 正常访问，肯定OK
     * @param id
     * @return
     */
    public String paymentInfo_OK(Integer id)
    {
        return "线程池:  "+Thread.currentThread().getName()+"  paymentInfo_OK,id:  "+id+"\t"+"O(∩_∩)O哈哈~";
    }

    @HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler", commandProperties = {
            //设置访问正常的时间，超时交由fallbackMethod处理
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="5000")
    })
    public String paymentInfo_TimeOut(Integer id)
    {
        int timeNumber = 3000;
        try { TimeUnit.MILLISECONDS.sleep(timeNumber); } catch (InterruptedException e) { e.printStackTrace(); }
        return "线程池:  "+Thread.currentThread().getName()+" paymentInfo_TimeOut,id:  "+id+"\t"+"O(∩_∩)O哈哈~"+"  耗时(毫秒): "+timeNumber;
        //int age = 10/0;
        //return "线程池:  "+Thread.currentThread().getName()+" paymentInfo_TimeOut,id:  "+id+"\t"+"O(∩_∩)O哈哈~"+"  演示计算异常";
    }

    public String paymentInfo_TimeOutHandler(Integer id){
        return "线程池:  "+ Thread.currentThread().getName()+
                " 8001系统繁忙或者运行报错，请稍后再试,id:  "+id+
                "\t"+"o(╥﹏╥)o";
    }

    //=====服务熔断
    @HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),// 是否开启断路器
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),// 请求次数
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"), // 时间窗口期
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"),// 失败率达到多少后跳闸
    })
    public String paymentCircuitBreaker(@PathVariable("id") Integer id)
    {
        if(id < 0)
        {
            throw new RuntimeException("******id 不能负数");
        }
        //等价于UUID.randomUUID().toString()
        String serialNumber = IdUtil.simpleUUID();

        return Thread.currentThread().getName()+"\t"+"调用成功，流水号: " + serialNumber;
    }
    public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id)
    {
        return "id 不能负数，请稍后再试，/(ㄒoㄒ)/~~   id: " +id;
    }
}

```

参考：https://github.com/Netflix/Hystrix/blob/master/hystrix-core/src/main/java/com/netflix/hystrix/HystrixCommandProperties.java

**PaymentController**

```java
package demo.yangxu.springcloud.controller;

import demo.yangxu.springcloud.service.PaymentService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
@Slf4j
public class PaymentController {

    @Resource
    private PaymentService paymentService;

    @Value("${server.port}")
    private String severPort;

    @GetMapping(value = "/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id){
        String result = paymentService.paymentInfo_OK(id);
        log.info("*****result: "+result);
        return result;
    }

    @GetMapping(value = "/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
        String result = paymentService.paymentInfo_TimeOut(id);
        log.info("*****result: "+result);
        return result;
    }

    //====服务熔断
    @GetMapping("/payment/circuit/{id}")
    public String paymentCircuitBreaker(@PathVariable("id") Integer id)
    {
        String result = paymentService.paymentCircuitBreaker(id);
        log.info("****result: "+result);
        return result;
    }
}

```



### 2、测试

启动

- cloud-eureka-server7001（EurekaMain7001）
- cloud-provider-hystrix-payment8001（PaymentHystrixMain8001）

访问 `http://localhost:8001/payment/circuit/31`

```
hystrix-PaymentService-1 调用成功，流水号: e33e8c8a3e714505b247942b4c42a713
```

访问 `http://localhost:8001/payment/circuit/-31`

```
id 不能负数，请稍后再试，/(ㄒoㄒ)/~~ id: -31
```

使用 JMeter 进行测试

按照下面两张图进行配置

![](/img-post/2020-08-16-springcloud/14-13.jpg)

![](/img-post/2020-08-16-springcloud/14-14.jpg)

开启压力测试后，在浏览器中访问`http://localhost:8001/payment/circuit/31`，结果为错误。在一个窗口期内（10秒），一个线路的压力测试使用了12个线程，而且12个线程访问的都是错误接口，触发了熔断条件（熔断器的熔断要求为10秒中10次访问有60%的请求失败）。此时终止压力测试，进入下一个10秒的窗口期，再次访问`http://localhost:8001/payment/circuit/31`，结果为正确。

原理：

1、请求次数达到阈值（`HystrixCommandProperties.circuitBreakerRequestVolumeThreshold()`)...

2、失败率达到阈值（`HystrixCommandProperties.circuitBreakerErrorThresholdPercentage()`)...

3、然后断路器从`CLOSED`切换到`OPEN`。

4、在`OPEN`状态时，直接拒绝访问，拉闸限电，然后调用服务降级的方法并返回友好提示。

5、经过一段时间(`HystrixCommandProperties.circuitBreakerSleepWindowInMilliseconds()`)，下一个请求被允许通过（这是`HALF-OPEN`状态）。如果请求失败，断路器将在休眠窗口期间返回`OPEN` 状态。如果请求成功，断路器将转换到“CLOSED”，并且**1.**中的逻辑将再次接管。

## Hystrix之服务熔断总结

熔断类型：

- 熔断打开：请求不再进行调用当前服务，内部设置时钟一般为MTTR（平均故障处理时间），当打开时长达到所设时钟则进入半熔断状态
- 熔断关闭：熔断关闭不会对服务进行熔断
- 熔断半开：部分请求根据规则调用当前服务，如果请求成功且符合规则则认为当前服务恢复正常，关闭熔断

涉及到断路器的三个重要参数：快照时间窗、请求总数阈值、错误百分比阈值

- 快照时间窗：断路器确定是否打开需要统计一些请求和错误数据，而统计的时间范围就是快照时间窗，默认为最近的10秒。
- 请求总数阈值：在快照时间窗内，必须满足请求总数阈值才有资格熔断。默认为20，意味着在10秒内，如果该hystrix命令的调用次数不足20次，即使所有的请求都超时或其他原因失败，断路器都不会打开。
- 错误百分比阈值：当请求总数在快照时间窗内超过了阈值，比如发生了30次调用，如果在这30次调用中，有18次发生了超时异常，也就是超过了60%的错误百分比，这时候就会将断路器打开。

短路器开启的条件：

- 当满足一定的阈值的时候（默认10秒内超过20个请求次数）
- 当失败率达到一定的阈值的时候（默认10秒内超过50%的请求失败）

到达以上阈值，断路器将会开启。

当开启的时候，所有请求都不会进行转发。一段时间之后（默认是5秒），这个时候断路器是半开状态，会让其中一个请求进行转发。如果成功，断路器将关闭；如果失败，断路器将维持开启状态。

断路器打开之后：

1、再有请求调用的时候，将不会调用主逻辑，而是直接调用降级fallback，降级逻辑临时成为主逻辑。

2、当断路器打开，对主逻辑进行熔断之后，hystrix会启动一个休眠时间窗，在这个时间窗内，降级逻辑临时成为主逻辑。当休眠时间窗到期，断路器进入半开状态，释放一次请求到原来的主逻辑上，如果此次请求正常返回，那么断路器切换到闭合状态，主逻辑恢复；如果这次请求依然有问题，断路器维持打开状态，休眠时间窗重新计时。

Hystrix所有配置属性：

```java
 @HystrixCommand(fallbackMethod = "fallbackMethod", groupKey = "strGroupCommand", commandKey = "strCommand", threadPoolKey = "strThreadPool",
   commandProperties = {
     // 设置隔离策略，THREAD 表示线程池 SEMAPHORE：信号池隔离
     @HystrixProperty(name = "execution.isolation.strategy", value = "THREAD"),
     // 当隔离策略选择信号池隔离的时候，用来设置信号池的大小（最大并发数）
     @HystrixProperty(name = "execution.isolation.semaphore.maxConcurrentRequests", value = "10"),
     // 配置命令执行的超时时间
     @HystrixProperty(name = "execution.isolation.thread.timeoutinMilliseconds", value = "10"),
     // 是否启用超时时间
     @HystrixProperty(name = "execution.timeout.enabled", value = "true"),
     // 执行超时的时候是否中断
     @HystrixProperty(name = "execution.isolation.thread.interruptOnTimeout", value = "true"),
     // 执行被取消的时候是否中断
     @HystrixProperty(name = "execution.isolation.thread.interruptOnCancel", value = "true"),

     // 允许回调方法执行的最大并发数
     @HystrixProperty(name = "fallback.isolation.semaphore.maxConcurrentRequests", value = "10"),
     // 服务降级是否启用，是否执行回调函数
     @HystrixProperty(name = "fallback.enabled", value = "true"),

     // 是否启用断路器
     @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
     // 该属性用来设置在滚动时间窗中，断路器熔断的最小请求数。例如，默认该值为 20 的时候，如果滚动时间窗（默认10秒）内仅收到了19个请求， 即使这19个请求都失败了，断路器也不会打开。
     @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "20"),
     // 该属性用来设置在滚动时间窗中，表示在滚动时间窗中，在请求数量超过 circuitBreaker.requestVolumeThreshold 的情况下，如果错误请求数的百分比超过50, 就把断路器设置为 "打开" 状态，否则就设置为 "关闭" 状态。
     @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"),
     // 该属性用来设置当断路器打开之后的休眠时间窗。 休眠时间窗结束之后，会将断路器置为 "半开" 状态，尝试熔断的请求命令，如果依然失败就将断路器继续设置为 "打开" 状态，如果成功就设置为 "关闭" 状态。
     @HystrixProperty(name = "circuitBreaker.sleepWindowinMilliseconds", value = "5000"),
     // 断路器强制打开
     @HystrixProperty(name = "circuitBreaker.forceOpen", value = "false"),
     // 断路器强制关闭
     @HystrixProperty(name = "circuitBreaker.forceClosed", value = "false"),

     // 滚动时间窗设置，该时间用于断路器判断健康度时需要收集信息的持续时间
     @HystrixProperty(name = "metrics.rollingStats.timeinMilliseconds", value = "10000"),
     // 该属性用来设置滚动时间窗统计指标信息时划分"桶"的数量，断路器在收集指标信息的时候会根据设置的时间窗长度拆分成多个 "桶" 来累计各度量值，每个"桶"记录了一段时间内的采集指标。
     // 比如 10 秒内拆分成 10 个"桶"收集这样，所以 timeinMilliseconds 必须能被 numBuckets 整除。否则会抛异常
     @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "10"),
     // 该属性用来设置对命令执行的延迟是否使用百分位数来跟踪和计算。如果设置为 false, 那么所有的概要统计都将返回 -1。
     @HystrixProperty(name = "metrics.rollingPercentile.enabled", value = "false"),
     // 该属性用来设置百分位统计的滚动窗口的持续时间，单位为毫秒。
     @HystrixProperty(name = "metrics.rollingPercentile.timeInMilliseconds", value = "60000"),
     // 该属性用来设置百分位统计滚动窗口中使用 “ 桶 ”的数量。
     @HystrixProperty(name = "metrics.rollingPercentile.numBuckets", value = "60000"),
     // 该属性用来设置在执行过程中每个 “桶” 中保留的最大执行次数。如果在滚动时间窗内发生超过该设定值的执行次数，
     // 就从最初的位置开始重写。例如，将该值设置为100, 滚动窗口为10秒，若在10秒内一个 “桶 ”中发生了500次执行，
     // 那么该 “桶” 中只保留 最后的100次执行的统计。另外，增加该值的大小将会增加内存量的消耗，并增加排序百分位数所需的计算时间。
     @HystrixProperty(name = "metrics.rollingPercentile.bucketSize", value = "100"),
     // 该属性用来设置采集影响断路器状态的健康快照（请求的成功、 错误百分比）的间隔等待时间。
     @HystrixProperty(name = "metrics.healthSnapshot.intervalinMilliseconds", value = "500"),

     // 是否开启请求缓存
     @HystrixProperty(name = "requestCache.enabled", value = "true"),
     // HystrixCommand的执行和事件是否打印日志到 HystrixRequestLog 中
     @HystrixProperty(name = "requestLog.enabled", value = "true"),

   },
   threadPoolProperties = {
     // 该参数用来设置执行命令线程池的核心线程数，该值也就是命令执行的最大并发量
     @HystrixProperty(name = "coreSize", value = "10"),
     // 该参数用来设置线程池的最大队列大小。当设置为 -1 时，线程池将使用 SynchronousQueue 实现的队列，否则将使用 LinkedBlockingQueue 实现的队列。
     @HystrixProperty(name = "maxQueueSize", value = "-1"),
     // 该参数用来为队列设置拒绝阈值。 通过该参数， 即使队列没有达到最大值也能拒绝请求。
     // 该参数主要是对 LinkedBlockingQueue 队列的补充,因为 LinkedBlockingQueue 队列不能动态修改它的对象大小，而通过该属性就可以调整拒绝请求的队列大小了。
     @HystrixProperty(name = "queueSizeRejectionThreshold", value = "5"),
   }
 )
 public String strConsumer() {
  ResponseEntity<String> result = restTemplate.getForEntity("http://cloud-eureka-client/hello", String.class);
  return result.getBody();
 }
```

参考：https://github.com/Netflix/Hystrix/wiki/Configuration

## Hystrix工作流程最后总结

![](/img-post/2020-08-16-springcloud/14-15.png)

参考：https://github.com/Netflix/Hystrix/wiki/How-it-Works

## Hystrix图形化Dashboard搭建

除了隔离依赖服务的调用以外，Hystrix还提供了准实时的调用监控。Hystrix会持续的记录所有通过Hystrix发起的请求的执行信息，并统计报表和图形的形式展示给用户，包括每秒执行多少次请求、成功、失败等。Netflix通过hystrix-metrics-event-stream项目实现了对以上指标的监控。spring cloud 也提供了对Hystrix Dashboard的整合，对监控内容转化成可视化的界面。

### 1、新建Module

cloud-consumer-hystrix-dashboard9001

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

    <artifactId>cloud-consumer-hystrix-dashboard9001</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

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

```yaml
server:
  port: 9001
```

随着 Module 的增加，如果新建的 Module 没有被 IDEA 识别为 Spring 工程，可以尝试下面图片所示的步骤进行解决。

![](/img-post/2020-08-16-springcloud/14-16.jpg)

![](/img-post/2020-08-16-springcloud/14-17.jpg)

![](/img-post/2020-08-16-springcloud/14-18.jpg)

![](/img-post/2020-08-16-springcloud/14-19.jpg)

![](/img-post/2020-08-16-springcloud/14-20.jpg)

### 4、HystrixDashboardMain9001

添加新注解@EnableHystrixDashboard

```java
package demo.yangxu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;

@SpringBootApplication
@EnableHystrixDashboard
public class HystrixDashboardMain9001 {
    public static void main(String[] args) {
        SpringApplication.run(HystrixDashboardMain9001.class,args);
    }
}

```



### 5、配置provider微服务提供类

所有provider微服务提供类（8001/8002/8003）都需要监控依赖配置

```xml
<!--web-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



### 6、启动

启动cloud-consumer-hystrix-dashboard9001

访问 `http://localhost:9001/hystrix`

![](/img-post/2020-08-16-springcloud/14-21.jpg)

## Hystrix图形化Dashboard监控实战

### 1、修改 cloud-provider-hystrix-payment8001

PaymentHystrixMain8001

```java
package demo.yangxu.springcloud;

import com.netflix.hystrix.contrib.metrics.eventstream.HystrixMetricsStreamServlet;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class PaymentHystrixMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentHystrixMain8001.class, args);
    }

    /**
     *此配置是为了服务监控而配置，与服务容错本身无关，springcloud升级后的坑
     *ServletRegistrationBean因为springboot的默认路径不是"/hystrix.stream"，
     *只要在自己的项目里配置上下面的servlet就可以了
     */
    @Bean
    public ServletRegistrationBean getServlet() {
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
}

```

参考：

https://blog.csdn.net/ddxd0406/article/details/79643059

http://www.voidcc.com/content/spring-boot-hystrix-stream-404

https://developpaper.com/hystrix-and-dashboard-in-spring-cloud/

### 2、启动

- cloud-eureka-server7001（EurekaMain7001）
- cloud-consumer-hystrix-dashboard9001（HystrixDashboardMain9001）
- cloud-provider-hystrix-payment8001（PaymentHystrixMain8001）

### 3、配置监控地址

![](/img-post/2020-08-16-springcloud/14-22.jpg)

http://localhost:9001/hystrix

http://localhost:8001/hystrix.stream

![](/img-post/2020-08-16-springcloud/14-23.jpg)

### 4、测试

http://localhost:8001/payment/circuit/31

http://localhost:8001/payment/circuit/-31

正常情况：

![](/img-post/2020-08-16-springcloud/14-27.gif)

熔断情况：

![](/img-post/2020-08-16-springcloud/14-28.gif)

### 5、怎么看 Dashboard

#### 7色

![](/img-post/2020-08-16-springcloud/14-26.jpg)

#### 1圈

实心圆：共有两种含义。它通过颜色的变化代表了实例的健康程度。它的健康度从 **绿色 < 黄色 < 橙色 < 红色** 递减。

该实心圆除了颜色的变化之外，它的大小也会根据实例的请求流量发生变化，流量越大该实心圆越大。所以通过该实心圆就可以在大量的实例中快速地发现**故障实例和高压力实例**。

#### 1线

曲线：用来记录2分钟内流量的相对变化，可以通过它来观察流量的上升和下降趋势。

#### 整图说明

![](/img-post/2020-08-16-springcloud/14-24.jpg)

![](/img-post/2020-08-16-springcloud/14-25.jpg)
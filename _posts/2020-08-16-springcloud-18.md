---
layout:     post
title:      18.SpringCloud学习笔记
subtitle:   Stream消息驱动
date:       2020-08-16
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloud
---
# 18.SpringCloud学习笔记--Stream消息驱动

## Stream为什么被引入

屏蔽底层消息中间件的差异，降低切换成本，统一消息的编程模型。

比如说我们用到了 RabbitMQ 和 Kafka，由于这两个消息中间件的架构上的不同，像 RabbitMQ 有 exchange，Kafka 有 Topic 和 Partitions 分区，这些中间件的差异性导致在实际项目开发中，给开发人员造成了一定的困扰。如果用了某一种消息队列，由于业务需求的变更，需要往另一种消息队列进行迁移，这时候一大堆东西都要重新推倒重新做，因为它跟我们的系统耦合了。

有没有一种技术，让我们不再关注具体 MQ 的细节，只需要用一种适配绑定的方式，降低各种 MQ 切换的成本呢？这时候 Spring Cloud Stream 给我们提供了一种解耦合的方式。

## Stream是什么及Binder介绍

官网：

https://spring.io/projects/spring-cloud-stream#overview

官方文档：

https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/3.0.1.RELEASE/reference/html/

中文指导手册：

https://www.springcloud.cc/spring-cloud-greenwich.html

在目录中找到 “五. Spring Cloud Stream”

**什么是 Spring Cloud Stream**

官方定义 Spring Cloud Stream 是一个**构建消息驱动微服务的框架。**

**应用程序**通过 inputs 或者 outputs 与 Spring Cloud Stream 中 **binder 对象交互** ，通过我们配置来 binding (绑定) 。而 Spring Cloud Stream 的 **binder对象** 负责与 **消息中间件交互** 。所以，我们只需要搞清楚如何与 Spring Cloud Stream 交互就可以方便使用消息驱动的方式。

通过使用 Spring Integration 来连接消息代理中间件以实现消息事件驱动。Spring Cloud Stream 为一些供应商的消息中间件产品提供了个性化的自动化配置实现，引入了**发布-订阅、消费组、消息分区**这三个核心概念。

目前**仅支持 RabbitMQ、Kafka**

## Stream的设计思想

### 标准MQ架构

![](/img-post/2020-08-16-springcloud/18-01.jpg)

- 生产者 / 消费者之间靠消息媒介传递信息内容 —— Message
- 消息必须走特定的通道 —— 消息通道 MessageChannel
- 消息通道 MessageChannel 的子接口 SubscribableChannel 负责收发处理消息，由 MessageHandler 消息处理器订阅消费

### 为什么用 Spring Cloud Stream

屏蔽底层消息中间件的差异，降低切换成本，统一消息的编程模型。

![](/img-post/2020-08-16-springcloud/18-02.png)

### 为什么 Stream 可以统一底层差异

在没有绑定器这个概念的情况下，Spring Boot 应用要直接与消息中间件进行信息交互的时候，由于各消息中间件构建的初衷不同，它们的实现细节上会有较大的差异性，通过**定义绑定器作为中间层**，完美地实现了应用程序与消息中间件细节之间的隔离。通过向应用程序暴露统一的 Channel 通道，使得应用程序不需要再考虑各种不同的消息中间件的实现细节。

通过定义绑定器 Binden 作为中间层，**实现了应用程序与消息中间件细节之间的隔离**

### Binder

- INPUT 对应于消费者（消费者发送消息时使用的是`Sink`接口里的`input`方法）
- OUTPUT 对应于生产者（生产者发送消息时使用的是`Source`接口里的`output`方法）
- Stream中的消息通信方式遵循了发布-订阅模式，Topic 主题进行广播，在 RabbitMQ 就是 Exchange，在kafka 中就是 Topic

![](/img-post/2020-08-16-springcloud/18-03.png)

Stream 对消息中间件的进一步封装可以做到**代码层面对中间件的无感知**，甚至于动态的切换中间件(rabbitmq 切换为 kafka)，使得微服务开发的高度解耦，服务可以关注更多自己的业务流程。

通过定义绑定器 Binder 作为中间层，**实现了应用程序与消息中间件细节之间的隔离。**

## Stream编码常用注解简介

### Spring Cloud Stream 标准流程

![](/img-post/2020-08-16-springcloud/18-04.jpg)

- **Binder：** 用来物理连接外部的中间件，屏蔽差异
- **Channel：** 通道，是队列 Queue 的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过 Channel 对队列进行配置
- **Source 和 Sink：** 简单的可理解为参照对象是 Spring Cloud Stream 自身，从 Stream 发布消息就是输出，接受消息就是输入

### 编码API和常用注解

![](/img-post/2020-08-16-springcloud/18-03.png)

| 组成            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| Middleware      | 中间件，目前只支持RibbitMQ和Kafaka                           |
| Binder          | Binder是应用与消息中间件之间的封装，目前实行了Kafka和RabbitMQ的Binder，通过Binder可以很方便的连接中间件，可以动态的改变消息类型（对应于Kafka的topic，RabbitMQ的exchange），这些都可以通过配置文件来实现。 |
| @Input          | 注解标识输入通道，通过该输入通道接收到的消息进入应用程序     |
| @Output         | 注解标识输出通道，发布的消息将通过该通道离开应用程序         |
| @StreamListener | 监听队列，用于消费者的队列的消息接收                         |
| @EnableBinding  | 指信道channel和exchange绑定在一起                            |

### 案例说明

RabbitMQ环境准备完毕

工程中新建三个子模块

- cloud-stream-rabbitmq-provider8801 作为生产者发送消息的模块

- cloud-stream-rabbitmq-consumer8802 作为消费者接收消息的模块

- cloud-stream-rabbitmq-consumer8803 作为消费者接收消息的模块

## Stream消息驱动之生产者

### **1、建Module**

cloud-stream-rabbitmq-provider8801

### **2、改POM**

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

    <artifactId>cloud-stream-rabbitmq-provider8801</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
        <!--基础配置-->
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

### **3、写YML**

application.yml

```yaml
server:
  port: 8801

spring:
  application:
    name: cloud-stream-provider
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: rabbitmq
                port: 5672
                username: guest
                password: guest
                virtual-host: /

      bindings: # 服务的整合处理
        output: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: send-8801.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址


```

如果连接到远程 RabbitMQ 报 Rabbit health check failed 异常，解决方法可以参考下面这篇博文：

[解决SpringCloud使用RabbitMQ报错Rabbit health check failed的问题](https://blog.csdn.net/telyfox/article/details/107783828)

### **4、主启动类**

StreamMQMain8801

```java
package demo.yangxu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StreamMQMain8801 {
    public static void main(String[] args)
    {
        SpringApplication.run(StreamMQMain8801.class,args);
    }
}

```

### **5、业务类**

发送消息接口

IMessageProvider

```java
package demo.yangxu.springcloud.service;

public interface IMessageProvider {
    public String send();
}

```

发送消息接口实现类

![](/img-post/2020-08-16-springcloud/18-04.jpg)



MessageProviderImpl

```java
package demo.yangxu.springcloud.service.impl;

import demo.yangxu.springcloud.service.IMessageProvider;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Source;
import org.springframework.integration.support.MessageBuilder;
import org.springframework.messaging.MessageChannel;

import javax.annotation.Resource;
import java.util.UUID;

@EnableBinding(Source.class) //定义消息的推送管道
public class MessageProviderImpl implements IMessageProvider {

    @Resource
    private MessageChannel output; // 消息发送管道

    @Override
    public String send() {
        String serial = UUID.randomUUID().toString();
        output.send(MessageBuilder.withPayload(serial).build());
        System.out.println("*****serial: "+serial);
        return null;
    }
}

```

参考：

https://cloud.spring.io/spring-cloud-static/spring-cloud-stream-binder-rabbit/3.0.1.RELEASE/reference/html/spring-cloud-stream-binder-rabbit.html#_partitioning_with_the_rabbitmq_binder

Controller

SendMessageController

```java
package demo.yangxu.springcloud.controller;

import demo.yangxu.springcloud.service.IMessageProvider;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
public class SendMessageController {

    @Resource
    private IMessageProvider messageProvider;

    @GetMapping(value = "/sendMessage")
    public String sendMessage(){
        return messageProvider.send();
    }
}

```

### **6、测试**

启动

- cloud-eureka-server7001（EurekaMain7001）

- RabbitMQ服务

- cloud-stream-rabbitmq-provider8801（StreamMQMain8801）


在 Exchanges 中可以看到 studyExchange

![](/img-post/2020-08-16-springcloud/18-06.jpg)

频繁访问：

http://localhost:8801/sendMessage

可以在 Overview 中的图表中看到波动

![](/img-post/2020-08-16-springcloud/18-07.jpg)

在 IDEA 中的控制台也可以看到输出的信息：

![](/img-post/2020-08-16-springcloud/18-08.jpg)

## Stream消息驱动之消费者

### **1、建Module**

cloud-stream-rabbitmq-consumer8802

### **2、改POM**

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

    <artifactId>cloud-stream-rabbitmq-consumer8802</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--基础配置-->
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

### **3、写YML**

```yaml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: rabbitmq
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: receive-8802.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址

```

如果连接到远程 RabbitMQ 报 Rabbit health check failed 异常，解决方法可以参考下面这篇博文：

[解决SpringCloud使用RabbitMQ报错Rabbit health check failed的问题](https://blog.csdn.net/telyfox/article/details/107783828)

### **4、主启动类**

StreamMQMain8802

```java
package demo.yangxu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StreamMQMain8802 {
    public static void main(String[] args)
    {
        SpringApplication.run(StreamMQMain8802.class,args);
    }
}

```

### **5、业务类**

![](/img-post/2020-08-16-springcloud/18-04.jpg)

Controller

ReceiveMessageListenerController

```java
package demo.yangxu.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.cloud.stream.messaging.Sink;
import org.springframework.messaging.Message;
import org.springframework.stereotype.Component;

@Component
@EnableBinding(Sink.class)
public class ReceiveMessageListenerController {

    @Value("${server.port}")
    private String serverPort;

    @StreamListener(Sink.INPUT)
    public void input(Message<String> message){
        System.out.println("消费者1号,----->接受到的消息: "+message.getPayload()+"\t  port: "+serverPort);
    }
}

```

### **6、测试**

测试 8801 发送 8802 接收消息

启动

- cloud-eureka-server7001（EurekaMain7001）

- RabbitMQ服务

- cloud-stream-rabbitmq-provider8801（StreamMQMain8801）
- cloud-stream-rabbitmq-consumer8802（StreamMQMain8802）

访问：

http://localhost:8801/sendMessage

在 8801 的控制台上可以看到发送的消息：

![](/img-post/2020-08-16-springcloud/18-09.jpg)

在 8802 的控制台上可以看到接收的消息：

![](/img-post/2020-08-16-springcloud/18-10.jpg)

在 Exchange: studyExchange 的 Bindings 中可以看到一个匿名的订阅者：

![](/img-post/2020-08-16-springcloud/18-11.jpg)

## Stream之消息重复消费

### 准备工作

根据 cloud-stream-rabbitmq-consumer8802 克隆出一个 cloud-stream-rabbitmq-consumer8803

启动

- cloud-eureka-server7001（EurekaMain7001）
- RabbitMQ服务
- cloud-stream-rabbitmq-provider8801（StreamMQMain8801）
- cloud-stream-rabbitmq-consumer8802（StreamMQMain8802）
- cloud-stream-rabbitmq-consumer8803（StreamMQMain8803）

### 运行后的问题

- 重复消费

- 消息持久化

运行后，8801 发送消息，8802 和 8803 均接收到消息。即发生了**重复消费问题。**

以一个实际生产案例来说明。

比如在如下场景中，订单系统做集群部署，会从 RabbitMQ 中获取订单信息，如果一个订单同时被两个服务获取，那么就会造成数据错误，需要避免这种情况。这时我们就可以使用 Stream 中的消息分组来解决。

![](/img-post/2020-08-16-springcloud/18-12.jpg)

注意，**Stream 中处于同一个 group 中的多个消费者是竞争关系，就能够保证消息只会被其中一个应用消费一次。不同组是可以全面消费的(重复消费)，同一组内会发生竞争关系，只有其中一个可以消费。**

查看 RabbitMQ 当前的分组情况：

![](/img-post/2020-08-16-springcloud/18-13.jpg)

## Stream之group解决消息重复消费

重复消费导致原因：默认分组 group 是不同的，组流水号不一样，被 RabbitMQ 认定为不同的组，造成了重复消费。

分布式微服务应用为了实现高可用和负载均衡，实际上会部署多个实例。

多数情况，生产者发送消息给某个具体微服务时只希望被消费一次，虽然 8802 和 8803 同属一个应用，但是这个消息出现了被重复消费两次的情况。为了解决这个问题，在 Spring Cloud Stream 中提供了消费组的概念。

解决思路：

自定义配置分组，自定义配置为同一个组，解决重复消费问题。

8802 / 8803 实现轮询分组，每次只有一个消费者。

8801 模块发的消息只能被 8802 和 8803 其中一个接收到，避免了重复消费。

8802 修改 YML

```yaml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: rabbitmq
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
          group: yangxuA

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: receive-8802.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址

```

如果连接到远程 RabbitMQ 报 Rabbit health check failed 异常，解决方法可以参考下面这篇博文：

[解决SpringCloud使用RabbitMQ报错Rabbit health check failed的问题](https://blog.csdn.net/telyfox/article/details/107783828)

8803 修改 YML

```yaml
server:
  port: 8803

spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: rabbitmq
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
          group: yangxuA

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: receive-8803.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址

```

如果连接到远程 RabbitMQ 报 Rabbit health check failed 异常，解决方法可以参考下面这篇博文：

[解决SpringCloud使用RabbitMQ报错Rabbit health check failed的问题](https://blog.csdn.net/telyfox/article/details/107783828)

结果：同一个组的多个微服务实例，每次只会有一个拿到。

![](/img-post/2020-08-16-springcloud/18-14.jpg)

## Stream之消息持久化

通过上述测试，解决了重复消费问题，再看看持久化。

停止 8802 / 8803 并去除掉 8802 的分组 group:groupA，8803 的分组 group:groupA 保留。

8801 先发送 4 条信息到 rabbitmq。

![](/img-post/2020-08-16-springcloud/18-15.jpg)

先启动8802，无分组属性配置，后台没有打出来消息。

![](/img-post/2020-08-16-springcloud/18-16.jpg)

再启动 8803，有分组属性配置，后台打出来了四条消息。

![](/img-post/2020-08-16-springcloud/18-17.jpg)

即**配置了group: 属性后**当消费者停机再次启动后，依然能从 stream 上获取生产者已发送的消息，即不会错过或丢失任何消息，但是不配置 group 属性就收不到了，这就是**消息的持久化。**
---
layout:     post
title:      10.SpringCloud学习笔记
subtitle:   ZooKeeper
date:       2020-08-16
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloud
---
# 10.SpringCloud学习笔记--ZooKeeper

## 支付服务注册进zookeeper

### 1、建module

cloud-provider-payment8004

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

    <artifactId>cloud-provider-payment8004</artifactId>

    <dependencies>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
            <groupId>demo.yangxu.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!-- SpringBoot整合zookeeper客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <!--先排除自带的zookeeper3.5.3-->
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!--添加zookeeper3.4.14版本-->
        <!--zookeeper版本要与服务器上的版本一致-->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.14</version>
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
#8004 表示注册到zookeeper服务器的支付服务提供者端口号
server:
  port: 8004

spring:
  #服务别名——注册zookeeper到注册中心名称
  application:
    name: cloud-provider-payment

  cloud:
    zookeeper:
      connect-string: 192.168.25.157:2181
```



### 4、主启动类

```java
package demo.yangxu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient //该注解用于向使用consul或者zookeeper作为注册中心时注册服务
public class PaymentMain8004 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8004.class,args);
    }
}

```



### 5、业务层

**Controller**

demo.yangxu.springcloud.controller.PaymentController

```java
package demo.yangxu.springcloud.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

@RestController
@Slf4j
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    @RequestMapping(value = "/payment/zk")
    public String paymentzk()
    {
        return "springcloud with zookeeper: "+serverPort+"\t"+ UUID.randomUUID().toString();
    }
}

```

### 6、启动8004注册进zookeeper

解决zookeeper版本冲突问题

排除自带的zookeeper3.5.3，添加zookeeper3.4.14版本

```xml
<!-- SpringBoot整合zookeeper客户端 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
    <!--先排除自带的zookeeper3.5.3-->
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!--添加zookeeper3.4.14版本-->
<!--zookeeper版本要与服务器上的版本一致-->
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.14</version>
</dependency>
```

### 7、测试

在 linux 下启动 zkCli.sh

```bash
[root@localhost bin]# ./zkCli.sh

[zk: localhost:2181(CONNECTED) 0] ls /
[services, zookeeper]

[zk: localhost:2181(CONNECTED) 1] ls /services
[cloud-provider-payment]

[zk: localhost:2181(CONNECTED) 4] ls /services/cloud-provider-payment
[5b833c7a-6a7b-43a9-aecf-a6f8cafc76b7]

[zk: localhost:2181(CONNECTED) 6] ls /services/cloud-provider-payment/5b833c7a-6a7b-43a9-aecf-a6f8cafc76b7
[]

[zk: localhost:2181(CONNECTED) 8] get /services/cloud-provider-payment/5b833c7a-6a7b-43a9-aecf-a6f8cafc76b7
{"name":"cloud-provider-payment","id":"5b833c7a-6a7b-43a9-aecf-a6f8cafc76b7","address":"DESKTOP-D47B342.localdomain","port":8004,"sslPort":null,"payload":{"@class":"org.springframework.cloud.zookeeper.discovery.ZookeeperInstance","id":"application-1","name":"cloud-provider-payment","metadata":{}},"registrationTimeUTC":1595304583390,"serviceType":"DYNAMIC","uriSpec":{"parts":[{"value":"scheme","variable":true},{"value":"://","variable":false},{"value":"address","variable":true},{"value":":","variable":false},{"value":"port","variable":true}]}}
cZxid = 0x6
ctime = Tue Jul 21 12:09:43 CST 2020
mZxid = 0x6
mtime = Tue Jul 21 12:09:43 CST 2020
pZxid = 0x6
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x1000000949a0001
dataLength = 548
numChildren = 0


[zk: localhost:2181(CONNECTED) 2] quit
```

对json串进行格式化

```json
{
  "name": "cloud-provider-payment",
  "id": "5b833c7a-6a7b-43a9-aecf-a6f8cafc76b7",
  "address": "DESKTOP-D47B342.localdomain",
  "port": 8004,
  "sslPort": null,
  "payload": {
    "@class": "org.springframework.cloud.zookeeper.discovery.ZookeeperInstance",
    "id": "application-1",
    "name": "cloud-provider-payment",
    "metadata": {}
  },
  "registrationTimeUTC": 1595304583390,
  "serviceType": "DYNAMIC",
  "uriSpec": {
    "parts": [
      {
        "value": "scheme",
        "variable": true
      },
      {
        "value": "://",
        "variable": false
      },
      {
        "value": "address",
        "variable": true
      },
      {
        "value": ":",
        "variable": false
      },
      {
        "value": "port",
        "variable": true
      }
    ]
  }
}
```

浏览器访问

http://localhost:8004/payment/zk

```
springcloud with zookeeper: 8004 a53e5915-65cc-4457-8a99-72241791357b
```

## 临时还是持久节点

临时

## 订单服务注册进zookeeper

### 1、建module

cloud-consumerzk-order80

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

    <artifactId>cloud-consumerzk-order80</artifactId>

    <dependencies>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- SpringBoot整合zookeeper客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <!--先排除自带的zookeeper-->
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!--添加zookeeper3.4.14版本-->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.14</version>
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
  port: 80

spring:
  #服务别名——注册zookeeper到注册中心名称
  application:
    name: cloud-consumer-order

  cloud:
    zookeeper:
      connect-string: 192.168.25.157:2181
```



### 4、主启动类

```java
package demo.yangxu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class OrderZKMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderZKMain80.class,args);
    }
}

```



### 5、业务层

**配置Bean**

demo.yangxu.springcloud.config.ApplicationContextConfig

```java
package demo.yangxu.springcloud.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}

```

**Controller**

demo.yangxu.springcloud.controller.OrderZKController

```java
package demo.yangxu.springcloud.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderZKController {

    public static final String INVOKE_URL = "http://cloud-provider-payment";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping(value = "/consumer/payment/zk")
    public String paymentInfo(){
        String result = restTemplate.getForObject(INVOKE_URL + "/payment/zk", String.class);
        return result;
    }
}

```

### 6、测试

在 linux 下启动 zkCli.sh

```bash
[root@localhost ~]# /usr/local/zookeeper-3.4.14/bin/zkCli.sh

[zk: localhost:2181(CONNECTED) 1] ls /services
[cloud-provider-payment, cloud-consumer-order]


```

浏览器访问

http://localhost/consumer/payment/zk

```
springcloud with zookeeper: 8004 312798e7-cf07-4ac9-9112-980464ea487a
```


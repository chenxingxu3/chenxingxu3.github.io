---
layout:     post
title:      11.SpringCloud学习笔记
subtitle:   Consul
date:       2020-08-16
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloud
---
# 11.SpringCloud学习笔记--Consul

## Consul简介

### 是什么

>Consul is a service mesh solution providing a full featured control plane with service discovery, configuration, and segmentation functionality. Each of these features can be used individually as needed, or they can be used together to build a full service mesh. Consul requires a data plane and supports both a proxy and native integration model. Consul ships with a simple built-in proxy so that everything works out of the box, but also supports 3rd party proxy integrations such as Envoy.

参考：https://www.consul.io/intro

### 能干嘛

>The key features of Consul are:
>
>- **Service Discovery**: Clients of Consul can register a service, such as `api` or `mysql`, and other clients can use Consul to discover providers of a given service. Using either DNS or HTTP, applications can easily find the services they depend upon.
>- **Health Checking**: Consul clients can provide any number of health checks, either associated with a given service ("is the webserver returning 200 OK"), or with the local node ("is memory utilization below 90%"). This information can be used by an operator to monitor cluster health, and it is used by the service discovery components to route traffic away from unhealthy hosts.
>- **KV Store**: Applications can make use of Consul's hierarchical key/value store for any number of purposes, including dynamic configuration, feature flagging, coordination, leader election, and more. The simple HTTP API makes it easy to use.
>- **Secure Service Communication**: Consul can generate and distribute TLS certificates for services to establish mutual TLS connections. [Intentions](https://www.consul.io/docs/connect/intentions) can be used to define which services are allowed to communicate. Service segmentation can be easily managed with intentions that can be changed in real time instead of using complex network topologies and static firewall rules.
>- **Multi Datacenter**: Consul supports multiple datacenters out of the box. This means users of Consul do not have to worry about building additional layers of abstraction to grow to multiple regions.

参考：https://www.consul.io/intro

### 去哪下

https://www.consul.io/downloads

### 怎么玩

https://www.springcloud.cc/spring-cloud-consul.html

https://www.consul.io/docs/agent/options.html

https://learn.hashicorp.com/consul/day-0/containers-guide

https://www.consul.io/docs/agent/options.html#_bind

## 安装并运行Consul

https://learn.hashicorp.com/consul/getting-started/install.html

**在windows下运行**

```
consul agent -dev
```

**在 Docker 中安装及运行**

docker 拉取 consul 镜像

```bash
docker pull consul:1.6
```

启动 server

启动前, 先建立 `/data/consul` 文件夹, 保存 consul 的数据

```kotlin
mkdir -p /data/consul
```

使用 docker run 启动 server

```kotlin
docker run -d -p 8500:8500 -v /data/consul:/consul/data -e CONSUL_BIND_INTERFACE='eth0' --name=consul1 consul:1.6 agent -server -bootstrap -ui -client='0.0.0.0'
```

- `agent`: 表示启动 agent 进程
- `server`: 表示 consul 为 server 模式
- `client`: 表示 consul 为 client 模式
- `bootstrap`: 表示这个节点是 Server-Leader
- `ui`: 启动 Web UI, 默认端口 8500
- `node`: 指定节点名称, 集群中节点名称唯一
- `client`: 绑定客户端接口地址, 0.0.0.0 表示所有地址都可以访问

一般第一个容器的ip地址是 172.17.0.2，可以通过下面的命令查询容器ip：

```bash
docker inspect --format '{{ .NetworkSettings.IPAddress }}' consul1
```

往集群插入其他节点

```csharp
docker run -d --name=consul2 -e CONSUL_BIND_INTERFACE=eth0 consul agent --server=true --client=0.0.0.0 --join 172.17.0.2;
docker run -d --name=consul3 -e CONSUL_BIND_INTERFACE=eth0 consul agent --server=true --client=0.0.0.0 --join 172.17.0.2;
docker run -d --name=consul4 -e CONSUL_BIND_INTERFACE=eth0 consul agent --server=false --client=0.0.0.0 --join 172.17.0.2;
```

- `join`: 表示加入到指定集群中

查看集群下面的节点

```bash
docker exec -it consul1 consul members
```

至此 consul 的部署已经完成。

上述只搭建了dc1，下面开始搭建dc2，并将dc1和dc2关联起来

```kotlin
docker run -d --name=consul5 -e CONSUL_BIND_INTERFACE='eth0' consul agent -server -bootstrap-expect 3 -datacenter=dc2
```

往dc2添加节点

```csharp
docker run -d --name=consul6 -e CONSUL_BIND_INTERFACE=eth0 consul agent --datacenter=dc2 --server=true --client=0.0.0.0 --join 172.17.0.6;
docker run -d --name=consul7 -e CONSUL_BIND_INTERFACE=eth0 consul agent --datacenter=dc2 --server=true --client=0.0.0.0 --join 172.17.0.6;
docker run -d --name=consul8 -e CONSUL_BIND_INTERFACE=eth0 consul agent --datacenter=dc2 --server=false --client=0.0.0.0 --join 172.17.0.6;
```

关联dc1和dc2

```css
docker exec -it consul6 consul join -wan 172.17.0.2  
```

至此可以在web ui看到dc1和dc2！

参考：https://www.jianshu.com/p/df3ef9a4f456

**访问consul的web ui**

http://192.168.25.157:8500/

## 服务提供者注册进Consul

### 1、建module

cloud-providerconsul-payment8006

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

    <artifactId>cloud-providerconsul-payment8006</artifactId>

    <dependencies>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>demo.yangxu.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!--SpringCloud consul-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--日常通用jar包配置-->
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
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>RELEASE</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>RELEASE</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```



### 3、写yml

```yaml
###consul服务端口号
server:
  port: 8006

spring:
  application:
    name: consul-provider-payment
  ####consul注册中心地址
  cloud:
    consul:
      host: 192.168.25.157
      port: 8500
      discovery:
        health-check-url: http://${spring.cloud.client.ip-address}:${server.port}/actuator/health
        prefer-ip-address: true
        #hostname: 127.0.0.1
        service-name: ${spring.application.name}
```



### 4、主启动类

```java
package demo.yangxu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain8006 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8006.class,args);
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

    @RequestMapping(value = "/payment/consul")
    public String paymentConsul(){
        return "springcloud with consul: "+serverPort+"\t   "+ UUID.randomUUID().toString();
    }
}

```



### 6、验证测试

http://192.168.25.157:8500/

![](/img-post/2020-08-16-springcloud/11-01.jpg)

![](/img-post/2020-08-16-springcloud/11-02.jpg)

http://localhost:8006/payment/consul

```
springcloud with consul: 8006 81f4242b-c9cf-47da-aa0c-7a68fe9e604a
```

## 服务消费者注册进Consul

### 1、建module

cloud-consumerconsul-order80

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

    <artifactId>cloud-consumerconsul-order80</artifactId>

    <dependencies>
        <!--SpringCloud consul-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--日常通用jar包配置-->
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
###consul服务端口号
server:
  port: 80

spring:
  application:
    name: cloud-consumer-order
  ####consul注册中心地址
  cloud:
    consul:
      host: 192.168.25.157
      port: 8500
      discovery:
        health-check-url: http://${spring.cloud.client.ip-address}:${server.port}/actuator/health
        prefer-ip-address: true
        #hostname: 127.0.0.1
        service-name: ${spring.application.name}


```



### 4、主启动类

```java
package demo.yangxu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class OrderConsulMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderConsulMain80.class,args);
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

demo.yangxu.springcloud.controller.OrderConsulController

```java
package demo.yangxu.springcloud.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderConsulController {
    public static final String INVOKE_URL = "http://consul-provider-payment";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping(value = "/consumer/payment/consul")
    public String paymentInfo(){
        String result = restTemplate.getForObject(INVOKE_URL+"/payment/consul",String.class);
        return result;
    }
}

```



### 6、验证测试

http://192.168.25.157:8500/

![](/img-post/2020-08-16-springcloud/11-03.jpg)

http://localhost/consumer/payment/consul

```
springcloud with consul: 8006 aa2845a5-ed98-405d-a32d-88337459dda4
```

## 三个注册中心异同点

| 组件名    | 语言 | CAP  | 服务健康检查 | 对外暴露接口 | SpringCloud 集成 |
| --------- | ---- | ---- | ------------ | ------------ | ---------------- |
| Eureka    | Java | AP   | 可配支持     | HTTP         | 支持             |
| Consul    | Go   | CP   | 支持         | HTTP/DNS     | 支持             |
| Zookeeper | Java | CP   | 支持         | 客户端       | 支持             |

## CAP Theorem

![](/img-post/2020-08-16-springcloud/11-04.png)

### 什么是 CAP？

- C：Consistency（强一致性）
- A：Availability（可用性）
- P：Partition tolerance（分区容错性）

最多只能同时较好的满足两个

CAP 理论的核心是：一个分布式系统不可能同时很好的满足一致性、可用性、分区容错性这三个需求，因此，根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三大类：

- CA：单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大
- CP：满足一致性，分区容忍性的系统，通常性能不是特别高
- AP：满足可用性，分区容忍性的系统，通常可能对一致性要求低一些

CAP 理论关注粒度是数据，而不是整个系统设计的策略

### AP 架构

当网络分区出现后，为了保证可用性，系统 B 可以返回旧值，保证系统的可用性

结论：违背了一致性 C 的要求，只满足可用性和分区容错，即 AP

![](/img-post/2020-08-16-springcloud/11-05.jpg)

### CP 架构

当网络分区出现后，为了保证一致性，就必须拒接请求，否则无法保证一致性

结论：违背了可用性 A 的要求，只满足一致性和分区容错，即 AP

![](/img-post/2020-08-16-springcloud/11-06.jpg)
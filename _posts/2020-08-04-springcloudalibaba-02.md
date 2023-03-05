---
layout:     post
title:      02.Spring Cloud Alibaba学习笔记
subtitle:   Nacos
date:       2020-08-04
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloudAlibaba
---
# 02.Spring Cloud Alibaba学习笔记--Nacos

## Nacos简介和下载

### 为什么叫 Nacos

前四个字母分别为 Naming 和 Configuration 的前两个字母，最后的 s 为 Service。

### Nacos 是什么

一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

Nacos: Dynamic Naming and Configuration Service.

Nacos 就是注册中心与配置中心的组合。

Nacos = Eureka + Config + Bus

### Nacos 用途

替代 Eureka 做服务注册中心。

替代 Config 做服务配置中心。

### Nacos 获取与文档

https://github.com/alibaba/nacos/releases/tag/1.1.4

https://github.com/alibaba/Nacos

https://nacos.io/zh-cn/

https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_nacos_discovery

### 各种注册中心比较

![](/img-post/2020-08-04-springcloudalibaba/02-01.jpg)

据说 Nacos 在阿里巴巴内部有超过 10 万的实例运行，经过了双十一等各种大型流量的考验。

## Nacos安装

**检查本地 Java 8 与 Maven 环境**

**从官网下载 Nacos**

https://github.com/alibaba/nacos/releases/tag/1.1.4

**解压安装包，直接运行 bin 目录下的 startup.cmd**

![](/img-post/2020-08-04-springcloudalibaba/02-02.jpg)

**命令运行成功后直接访问 http://localhost:8848/nacos**

默认的 用户名 / 密码 为 nacos / nacos

**结果页面**

![](/img-post/2020-08-04-springcloudalibaba/02-03.jpg)

## Nacos之服务提供者注册

### 建 Module

cloudalibaba-provider-payment9001

### 改 POM

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

    <artifactId>cloudalibaba-provider-payment9001</artifactId>

    <dependencies>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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



### 写 YML

application.yml

```yaml
server:
  port: 9001

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #配置Nacos地址

management:
  endpoints:
    web:
      exposure:
        include: '*'

```



### 主启动

PaymentMain9001

```java
package demo.yangxu.springcloud.alibaba;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain9001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain9001.class, args);
    }
}

```



### 业务类

Controller

PaymentController

```java
package demo.yangxu.springcloud.alibaba.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "/payment/nacos/{id}")
    public String getPayment(@PathVariable("id") Integer id)
    {
        return "nacos registry, serverPort: "+ serverPort+"\t id"+id;
    }
}

```



### 测试

启动：

- Nacos 服务器
- cloudalibaba-provider-payment9001

http://localhost:9001/payment/nacos/1

![](/img-post/2020-08-04-springcloudalibaba/02-04.jpg)

http://localhost:8848/nacos

![](/img-post/2020-08-04-springcloudalibaba/02-05.jpg)

![](/img-post/2020-08-04-springcloudalibaba/02-06.jpg)

![](/img-post/2020-08-04-springcloudalibaba/02-07.jpg)

### 负载均衡

演示负载均衡需要多个微服务。

一种方法是仿照 cloudalibaba-provider-payment9001 新建 cloudalibaba-provider-payment9002，另一种方法是使用虚拟端口映射。

找到 PaymentMain9001，右键选择 Copy Configuration：

![](/img-post/2020-08-04-springcloudalibaba/02-08.jpg)

填写好 Name，在 VM options 中填写

```
-DServer.port=9011
```

![](/img-post/2020-08-04-springcloudalibaba/02-09.jpg)

找到 9011 运行：

![](/img-post/2020-08-04-springcloudalibaba/02-10.jpg)

在 Nacos 中可以看到 9011 已经注册进来：

![](/img-post/2020-08-04-springcloudalibaba/02-11.jpg)

![](/img-post/2020-08-04-springcloudalibaba/02-12.jpg)

访问

http://localhost:9011/payment/nacos/1

返回结果：

```
nacos registry, serverPort: 9011 id1
```



## Nacos之服务消费者注册和负载

Nacos 整合了 ribbon，支持负载均衡。

### 建 Module

cloudalibaba-consumer-nacos-order83

### 改 POM

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

    <artifactId>cloudalibaba-consumer-nacos-order83</artifactId>

    <dependencies>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>demo.yangxu.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
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



### 写 YML

```yaml
server:
  port: 83


spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848


#消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
service-url:
  nacos-user-service: http://nacos-payment-provider

```



### 主启动

OrderNacosMain83

```java
package demo.yangxu.springcloud.alibaba;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableDiscoveryClient
@SpringBootApplication
public class OrderNacosMain83
{
    public static void main(String[] args)
    {
        SpringApplication.run(OrderNacosMain83.class,args);
    }
}

```



### 业务类

ApplicationContextBean

```java
package demo.yangxu.springcloud.alibaba.config;

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

OrderNacosController

```java
package demo.yangxu.springcloud.alibaba.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderNacosController {

    @Resource
    private RestTemplate restTemplate;

    @Value("${service-url.nacos-user-service}")
    private String serverURL;

    @GetMapping(value = "/consumer/payment/nacos/{id}")
    public String paymentInfo(@PathVariable("id") Long id){
        return restTemplate.getForObject(serverURL+"/payment/nacos/"+id,String.class);
    }
}

```



### 测试

启动：

- Nacos 服务器

- cloudalibaba-provider-payment9001

- cloudalibaba-provider-payment9002

- cloudalibaba-consumer-nacos-order83


Nacos 控制台：

http://localhost:8848/nacos/

![](/img-post/2020-08-04-springcloudalibaba/02-13.jpg)

http://localhost:83/consumer/payment/nacos/13

83 访问 9001 / 9002，轮询负载OK

## Nacos服务注册中心对比提升

**Nacos 全景图**

![](/img-post/2020-08-04-springcloudalibaba/02-14.png)

**Nacos 服务发现实例模型**

![](/img-post/2020-08-04-springcloudalibaba/02-15.jpg)

**Nacos 与其他注册中心特性对比**

![](/img-post/2020-08-04-springcloudalibaba/02-16.jpg)

**Nacos 支持 AP 和 CP 模式的切换**

- C - Consistency 所有节点在同一时间看到的数据是一致的
- A - Availability 所有的请求都会收到响应
- P - Partition tolerance 分区容错性

**何时选择使用何种模式**

一般来说，如果**不需要存储服务级别的信息，且服务实例是通过 nacos-client 注册的，并能够保持心跳上报，那么就可以选择 AP 模式**。

当前主流的服务如 Spring Cloud 和 Dubbo 服务，都适用于 AP 模式，AP 模式为了服务的可能性而减弱了一致性，因此 **AP 模式下只支持注册临时实例**。

**如果需要在服务级别编辑或者存储配置信息，那么必需选择 CP 模式，** K8S 服务和 DNS 服务则适用于 CP 模式。**CP 模式下支持注册持久化实例**，此时则是以 Raft 协议为集群运行模式，该模式下注册实例之前必须先注册服务，如果服务不存在，则会返回错误。

Raft 协议介绍：

http://thesecretlivesofdata.com/raft/

切换（默认工作于 AP 模式）：

```bash
curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP'
```



## Nacos之服务配置中心

官方文档：

https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_nacos_config

### Nacos 作为配置中心的基础配置

#### 建 Module

cloudalibaba-config-nacos-client3377

#### 改 POM

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

    <artifactId>cloudalibaba-config-nacos-client3377</artifactId>

    <dependencies>
        <!--nacos-config-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <!--nacos-discovery-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--web + actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--一般基础配置-->
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



#### 写 YAML

需要配置两个 yaml 文件。

Nacos 同 Spring Cloud Config 一样，在项目初始化时，要保证先从配置中心进行配置拉取。拉取配置之后，才能保证项目的正常启动。

Spring Boot 中配置文件的加载是存在优先级顺序的，bootstrap.yaml 的优先级高于 application.yaml。

bootstrap.yaml

```yaml
# nacos配置
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置


# ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
# nacos-config-client-dev.yaml
```

application.yaml

```yaml
spring:
  profiles:
    active: dev # 表示开发环境
```



#### 主启动

NacosConfigClientMain3377

```java
package demo.yangxu.springcloud.alibaba;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableDiscoveryClient
@SpringBootApplication
public class NacosConfigClientMain3377 {
    public static void main(String[] args) {
        SpringApplication.run(NacosConfigClientMain3377.class, args);
    }
}

```



#### 业务类

ConfigClientController

```java
package demo.yangxu.springcloud.alibaba.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RefreshScope //支持Nacos的动态刷新功能。
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        return configInfo;
    }
}

```



#### 在 Nacos 中添加配置信息

添加步骤：

1、配置管理 -> 配置列表 -> +

![](/img-post/2020-08-04-springcloudalibaba/02-18.jpg)

2、添加以下信息

Data ID: nacos-config-client-dev.yaml

配置格式: YAML

配置内容: 

```yaml
config:
    info: from nacos config center, nacos-config-client-dev.yaml, version = 1
```

![](/img-post/2020-08-04-springcloudalibaba/02-19.jpg)

3、发布

![](/img-post/2020-08-04-springcloudalibaba/02-20.jpg)

官方文档：

https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html

在 Nacos Spring Cloud 中，`dataId` 的完整格式如下：

```
${prefix}-${spring.profiles.active}.${file-extension}
```

- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
- `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)。 **注意：当 `spring.profiles.active` 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`**
- `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。

通过 Spring Cloud 原生注解 `@RefreshScope` 实现配置自动更新：

```java
@RestController
@RequestMapping("/config")
@RefreshScope
public class ConfigController {

    @Value("${useLocalCache:false}")
    private boolean useLocalCache;

    @RequestMapping("/get")
    public boolean get() {
        return useLocalCache;
    }
}
```

最终的公式：

```
${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
```

![](/img-post/2020-08-04-springcloudalibaba/02-17.jpg)

#### 测试

启动前需要在 nacos 客户端-配置管理-配置管理栏目下有对应的 yaml 配置文件。

运行 cloud-config-nacos-client3377 的主启动类。

调用接口查看配置信息：

http://localhost:3377/config/info

#### 自带动态刷新

修改 Nacos 中的 yaml 配置文件，再次调用查看配置的接口，发现配置已经刷新。

## Nacos之命名空间分组和DataID三者关系

### Nacos 作为配置中心的分类配置

#### 多环境多项目管理的问题

**问题1**

实际开发中，通常一个系统会准备

- dev 开发环境
- test 测试环境
- prod 生产环境

如何保证指定环境启动时，服务能正确读取到 Nacos 上相应环境的配置文件呢？

**问题2**

一个大型分布式微服务系统会有很多微服务子项目，

每个微服务项目又都会有相应的开发环境、测试环境、预发环境、正式环境……

那怎么对这些微服务的配置进行管理呢？

#### Namespace+Group+Data ID 三者关系

![](/img-post/2020-08-04-springcloudalibaba/02-21.jpg)

![](/img-post/2020-08-04-springcloudalibaba/02-22.jpg)

分类设计思想类似 Java 里面的 package 名和类名。

最外层的 NameSpace 是可以用于区分部署环境的，Group 和 DatalD 逻辑上区分两个目标对象。

![](/img-post/2020-08-04-springcloudalibaba/02-23.jpg)

**默认值:**

NameSpace = public, Group = DEFAULT_GROUP, Cluster = DEFAULT

Nacos 默认的命名空间是 public，NameSpace 主要用来实现隔离。比方说我们现在有三个环境：开发、测试、生产，我们就可以创建三个 NameSpace，不同的 NameSpace 之间是隔离的。

Group 默认是 DEFAULT_ GROUP, Group 可以把不同的微服务划分到同一个分组里面。

Service 就是微服务。一个 Service 可以包含多个 Cluster (集群)，Nacos 默认 Cluster是 DEFAULT, Cluster 是对指定微服务的一个虚拟划分。比方说为了容灾，将 Service 微服务分别部署在了杭州机房和广州机房，这时就可以给杭州机房的 Service 微服务起一个集群名称 (HZ)，给广州机房的 Service 微服务起一个集群名称 (GZ)， 还可以尽量让同一个机房的微服务互相调用，以提升性能。

最后是 Instance, 就是微服务的实例。

## Nacos之DataID配置

指定 spring.profile.active 和配置文件的 DataID，在不同的环境下读取不同的配置。

默认空间 + 默认分组 + 新建 dev 和 test 两个 DataID：

![](/img-post/2020-08-04-springcloudalibaba/02-24.jpg)

![](/img-post/2020-08-04-springcloudalibaba/02-26.jpg)

nacos-config-client-dev.yaml

```yaml
config:
    info: from nacos config center, nacos-config-client-dev.yaml, version = 2
```

nacos-config-client-test.yaml

```yaml
config:
    info: from nacos config center, nacos-config-client-test.yaml, version = 1
```

通过 spring.profile.active 属性就能进行多环境下配置文件的读取：

![](/img-post/2020-08-04-springcloudalibaba/02-25.jpg)

测试：

http://localhost:3377/config/info

## Nacos之Group分组方案

通过 Group 实现服务分组区分：

nacos-config-client-info.yaml, TEST_GROUP

![](/img-post/2020-08-04-springcloudalibaba/02-27.jpg)

```yaml
config:
    info: from nacos config center, nacos-config-client-info.yaml, TEST_GROUP,version = 1
```

nacos-config-client-info.yaml, DEV_GROUP

![](/img-post/2020-08-04-springcloudalibaba/02-28.jpg)

```yaml
config:
    info: from nacos config center, nacos-config-client-info.yaml, DEV_GROUP,version = 1
```

![](/img-post/2020-08-04-springcloudalibaba/02-29.jpg)

在 config 下增加一条 group 的配置即可。

可配置为 DEV_GROUP 或 TEST_GROUP

![](/img-post/2020-08-04-springcloudalibaba/02-30.jpg)

application.yaml

```yaml
spring:
  profiles:
    #active: dev # 表示开发环境
    #active: test # 表示测试环境
    active: info
```

bootstrap.yaml

```yaml
# nacos配置
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
        group: TEST_GROUP

```

测试：

http://localhost:3377/config/info

## Nacos之Namespace空间方案

**新建命名空间**

![](/img-post/2020-08-04-springcloudalibaba/02-31.jpg)

![](/img-post/2020-08-04-springcloudalibaba/02-32.jpg)

![](/img-post/2020-08-04-springcloudalibaba/02-33.jpg)

**在配置列表中切换到 dev 命名空间**

![](/img-post/2020-08-04-springcloudalibaba/02-34.jpg)

**新建配置**

nacos-config-client-dev.yaml, DEFAULT_GROUP

```yaml
config:
    info: dev, d7f4255f-df66-4034-8866-4285ea45918b, DEFAULT_GROUP, nacos-config-client-dev.yaml
```

nacos-config-client-dev.yaml, DEV_GROUP

```yaml
config:
    info: dev, d7f4255f-df66-4034-8866-4285ea45918b, DEV_GROUP, nacos-config-client-dev.yaml
```

nacos-config-client-dev.yaml, TEST_GROUP

```yaml
config:
    info: dev, d7f4255f-df66-4034-8866-4285ea45918b, TEST_GROUP, nacos-config-client-dev.yaml
```

![](/img-post/2020-08-04-springcloudalibaba/02-35.jpg)

**在配置文件中新增 spring.cloud.nacos.config.namespace 配置项**

bootstrap.yaml

```yaml
# nacos配置
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
        group: DEV_GROUP
        namespace: d7f4255f-df66-4034-8866-4285ea45918b

```

application.yaml

```yaml
spring:
  profiles:
    active: dev # 表示开发环境
    #active: test # 表示测试环境
    #active: info

```

**测试**

http://localhost:3377/config/info

返回结果

```
dev, d7f4255f-df66-4034-8866-4285ea45918b, DEV_GROUP, nacos-config-client-dev.yaml
```



## Nacos集群_架构说明

**集群部署架构图**

开源的时候推荐用户把所有服务列表放到一个 VIP (Virtual IP Address) 下面，然后挂到一个域名下面

[http://ip1](http://ip1/):port/openAPI —— 直连 ip 模式，机器挂则需要修改 ip 才可以使用。

[http://VIP](http://vip/):port/openAPI —— 挂载 VIP (Virtual IP Address) 模式，直连 VIP (Virtual IP Address) 即可，下面挂 server 真实 ip，可读性不好。

[http://nacos.com](http://nacos.com/):port/openAPI —— 域名 +  VIP (Virtual IP Address) 模式，可读性好，而且换 ip 方便，推荐模式。

![](/img-post/2020-08-04-springcloudalibaba/02-37.jpeg)

参考：

https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html

**实际集群部署架构图**

![](/img-post/2020-08-04-springcloudalibaba/02-36.jpg)

默认 Nacos 使用嵌入式数据库（Apache Derby）实现数据的存储。所以，如果启动多个默认配置下的 Nacos 节点，数据存储是存在一致性问题的。

为了解决这个问题，Nacos 采用了集中式存储的方式来支持集群化部署，目前**只支持 MySQL 的存储**。

**Nacos支持三种部署模式**

- 单机模式 - 用于测试和单机试用。
- 集群模式 - 用于生产环境，确保高可用。
- 多集群模式 - 用于多数据中心场景。

**单机模式支持 MySQL**

在 0.7 版本之前，在单机模式时 Nacos 使用嵌入式数据库实现数据的存储，不方便观察数据存储的基本情况。0.7 版本增加了支持 MySQL 数据源能力，具体的操作步骤：

- 1.安装数据库，版本要求：5.6.5+
- 2.初始化 MySQL 数据库，数据库初始化文件：nacos-mysql.sql
- 3.修改 conf/application.properties 文件，增加支持 MySQL 数据源配置（目前只支持 MySQL），添加 MySQL 数据源的 url、用户名和密码。

```properties
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://11.162.196.16:3306/nacos_devtest?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=nacos_devtest
db.password=youdontknow
```

再以单机模式启动 Nacos，Nacos 所有写嵌入式数据库的数据都写到了 MySQL。

## Nacos持久化切换配置

Nacos 默认自带的是嵌入式数据库 Derby。

来源：

https://github.com/alibaba/nacos/blob/develop/config/pom.xml

```xml
<dependency>
    <groupId>org.apache.derby</groupId>
    <artifactId>derby</artifactId>
</dependency>
```

### Derby 切换到 MySQL 的配置步骤

**1、在 nacos\conf 目录下找到 nacos-mysql.sql，执行该脚本**

**2、在 nacos\conf 目录下找到 application.properties 进行配置**

在 application.properties 中添加以下信息：

```properties
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://192.168.25.158:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=RzMCMsJ&*4oi09Kc
```

**3、重新启动 Nacos**

**4、测试**

在配置列表中新建配置，发布成功后，可以在数据库的 config_info 表中看到对应的记录。

![](/img-post/2020-08-04-springcloudalibaba/02-38.jpg)

![](/img-post/2020-08-04-springcloudalibaba/02-39.jpg)



## 
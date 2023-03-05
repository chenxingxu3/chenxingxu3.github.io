---
layout:     post
title:      16.SpringCloud学习笔记
subtitle:   Config分布式配置中心
date:       2020-08-16
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloud
---
# 16.SpringCloud学习笔记--Config分布式配置中心

## Config分布式配置中心介绍

### 分布式系统面临的配置问题

微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以**一套集中式的、动态的配置管理设施**是必不可少的。如果没有的话，每个微服务自己带着一个application.yml，上百个配置文件的管理特别繁琐。

Spring Cloud提供了ConfigServer来解决这个问题。

### Spring Cloud Config

![](/img-post/2020-08-16-springcloud/16-01.jpg)

Spring Cloud Config为微服务架构中的微服务**提供集中化的外部配置支持**，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。Spring Cloud Config分为服务端和客户端两部分。

服务端也称为分布式配置中心，它是一个**独立的微服务应用， 用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口。**

客户端则是**通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息。配置服务器默认采用git来存储配置信息**，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容。

### 作用

- 集中管理配置文件
- 不同环境不同配置，动态化的配置更新，分环境部署比如dev/test/prod/beta/release
- 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息
- 当配置发生变动时，服务不需要重启即可感知到配置的变化并应用新的配置
- 将配置信息以REST接口的形式暴露，post、curl访问刷新均可

### 与 GitHub 整合配置

由于 Spring Cloud Config 默认使用 Git 来存储配置文件（也有其他方式，不如支持 SVN 和本地文件），但最推荐的还是 Git，而且使用的是 http/ https 访问的形式。

### 官网

https://cloud.spring.io/spring-cloud-static/spring-cloud-config/2.2.1.RELEASE/reference/html/

## Config配置总控中心搭建

### 与Github整合配置

**1、在Github上新建一个名为springcloud-config的新Repository**

**2、把仓库 clone 下来本地，命令为：`git clone 地址`**

```bash
git clone git@github.com:telyfox/springcloud-config.git
```

**3、创建config-dev.yml，config-prod.yml，config-test.yml**

表示在 dev、prod、test 三种环境下的配置文件。

保存格式必须为 UTF-8

***注意文件格式必须为 xxx-xxx.yml格式***

`config-dev.yml` 内容为

```yaml
config:
  info: "master branch,springcloud-config/config-dev.yml version=7" 
```

`config-prod.yml`内容为

```yaml
config:
  info: "master branch,springcloud-config/config-prod.yml version=1" 
```

`config-test.yml`内容为

```yaml
config:
  info: "master branch,springcloud-config/config-test.yml version=1" 
```

**4、然后执行下面的命令，一步一步来**

```bash
git stage .
git commit -m"init config"
git push origin master
```

**5、创建新分支dev**

**6、将dev分支clone到本地**

```bash
git clone -b dev git@github.com:telyfox/springcloud-config.git
```

**7、创建config-dev.yml，config-prod.yml，config-test.yml，config-aaa-dev.yml**

`config-dev.yml` 内容为

```yaml
config:
    info: "dev branch,springcloud-config/config-dev.yml version=1" 
```

`config-prod.yml`内容为

```yaml
config:
    info: "dev branch,springcloud-config/config-prod.yml version=1" 
```

`config-test.yml`内容为

```yaml
config:
  info: "dev branch, springcloud-config/config-test.yml,version=1"
```

`config-aaa-dev.yml`内容为

```yaml
config:
  info: "dev branch dev branch,springcloud-config/config-aaa-dev.yml,version = 11." 
```

**8、然后执行下面的命令，一步一步来**

```bash
git stage .
git commit -m"dev config"
git push origin dev
```



### 创建cloud-config-center-3344

**1、建Module**

cloud-config-center-3344

**2、改POM**

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

    <artifactId>cloud-config-center-3344</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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

**3、写YML**

推荐spring.cloud.config.server.git.uri使用https地址

```yaml
server:
  port: 3344

spring:
  application:
    name:  cloud-config-center #注册进Eureka服务器的微服务名
  cloud:
    config:
      server:
        git:
          uri: https://github.com/telyfox/springcloud-config.git #GitHub上面的git仓库名字
          ####搜索目录
          search-paths:
            - springcloud-config

      ####读取分支
      label: master

#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka

```

使用ssh地址有以下注意事项：

本地ssh私钥要以-----BEGIN RSA PRIVATE KEY-----开头，如果是以-----BEGIN OPENSSH PRIVATE KEY-----会因为不支持而报Property 'spring.cloud.config.server.git.privateKey' is not a valid private key错误。

如何生成-----BEGIN RSA PRIVATE KEY-----开头的私钥和公钥可以参考：

https://www.cnblogs.com/wwct/p/12488951.html

如何配置多个ssh可以参考：

https://www.jianshu.com/p/a481837c9de2

https://blog.csdn.net/weixin_34315189/article/details/88040658

将私钥配置到YML文件中（不推荐，有安全风险）可以参考：

https://blog.csdn.net/sinat_38843093/article/details/79917102

```yaml
server:
  port: 3344

spring:
  application:
    name:  cloud-config-center #注册进Eureka服务器的微服务名
  cloud:
    config:
      server:
        git:
          uri: git@config.github.com:telyfox/springcloud-config.git #GitHub上面的git仓库名字
          ####搜索目录
          search-paths:
            - springcloud-config
        #配置仓库默认的分支,配置后生效
        default-label: master

      ####读取分支，配置后无效，原因不明
      label: master


#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka

```

**4、主启动类**

ConfigCenterMain3344

```java
package demo.yangxu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigCenterMain3344 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigCenterMain3344.class, args);
    }
}

```

**5、Windows 下修改 hosts 文件，增加映射**

```
127.0.0.1 config-3344.com
```

**6、测试通过 Config 微服务是否可以从 GitHub 上获取配置内容**

启动：

- cloud-eureka-server7001（EurekaMain7001）

- cloud-config-center-3344（ConfigCenterMain3344）


访问 `http://config-3344.com:3344/master/config-dev.yml`

### 配置读取规则

The HTTP service has resources in the following form:

```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

- label：分支（branch）
- application：服务名
- profiles：环境（dev / test / prod）

参考：

https://cloud.spring.io/spring-cloud-static/spring-cloud-config/2.2.1.RELEASE/reference/html/

#### /{label}/{application}-{profile}.yml（推荐）

实例：

master 分支：

```
http://config-3344.com:3344/master/config-dev.yml
http://config-3344.com:3344/master/config-test.yml
http://config-3344.com:3344/master/config-prod.yml
```

dev 分支：

```
http://config-3344.com:3344/dev/config-dev.yml
http://config-3344.com:3344/dev/config-test.yml
http://config-3344.com:3344/dev/config-prod.yml
```

#### /{application}-{profile}.yml

默认为`spring.cloud.config.default-label`所配置的分支

```
http://config-3344.com:3344/config-dev.yml
http://config-3344.com:3344/config-test.yml
http://config-3344.com:3344/config-test.yml
#不存在的配置
http://config-3344.com:3344/config-xxxx.yml
```

如果**访问一个不存在的配置**，如果上面的 GitHub 配置成功，则会**返回 {}**

#### /{application}-{profile}[/{label}]

以 json 格式返回原文件中的内容

```
http://config-3344.com:3344/config/dev/master
http://config-3344.com:3344/config/test/master
http://config-3344.com:3344/config/prod/master
```

以`http://config-3344.com:3344/config/dev/master`为例

```json
{
  "name": "config",
  "profiles": [
    "dev"
  ],
  "label": "master",
  "version": "e7654c21e20b2ff8a6c7e5f64e36003902a91f99",
  "state": null,
  "propertySources": [
    {
      "name": "git@config.github.com:telyfox/springcloud-config.git/config-dev.yml",
      "source": {
        "config.info": "master branch,springcloud-config/config-dev.yml version=7"
      }
    }
  ]
}
```

## Config客户端配置与测试

**1、建Module**

cloud-config-client-3355

**2、改POM**

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

    <artifactId>cloud-config-client-3355</artifactId>

    <dependencies>
        
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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

**3、写YML**

- applicaiton.yml是**用户级的资源配置项**
- bootstrap.yml是**系统级的，优先级更加高**

Spring Cloud会创建一个"Bootstrap Context" ，作为Spring应用的Application Context的父上下文。初始化的时候，"Bootstrap Context"**负责从外部源加载配置属性并解析配置**。这两个上下文共享一个从外部获取的Environment。

Bootstrap 属性有高优先级，**默认情况下，它们不会被本地配置覆盖**。Bootstrap context和Application Context有着不同的约定，所以新增了一个bootstrap.ym|文件，**保证Bootstrap Context和Application Context配置的分离**。

要将Client模块下的application.yml文件改为bootstrap.yml，这是很关键的，因为bootstrap.yml是比application.yml先加载的。bootstrap.ymI优先级高 于application.yml。

bootstrap.yml

```yaml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    #Config客户端配置
    config:
      label: master #分支名称
      name: config #配置文件名称
      profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
      uri: http://config-3344.com:3344 #配置中心地址


#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka


```

![](/img-post/2020-08-16-springcloud/16-02.jpg)

**4、修改config-dev.yml配置并提交到GitHub**

比如加个变量age或者版本号version

**5、主启动**

ConfigClientMain3355

```java
package demo.yangxu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class ConfigClientMain3355 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClientMain3355.class, args);
    }
}

```

**6、业务类**

controller

ConfigClientController

```java
package demo.yangxu.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo()
    {
        return configInfo;
    }
}

```

**7、测试**

启动：

- cloud-eureka-server7001（EurekaMain7001）
- cloud-config-center-3344（ConfigCenterMain3344）
- cloud-config-client-3355（ConfigClientMain3355）

访问 `http://localhost:3355/configInfo`

```
master branch,springcloud-config/config-dev.yml version=7
```

成功实现了客户端 3355 访问 Spring Cloud Config 3344 通过 GitHub 获取配置信息

**分布式配置的动态刷新**

1. Linux运维修改GitHub上的配置文件内容做调整
2. 刷新3344，发现ConfigServer配置中心立刻响应`http://config-3344.com:3344/master/config-dev.yml`
3. 刷新3355，发现ConfigServer客户端没任何响应`http://localhost:3355/configInfo`
4. 3355没有变化**除非自己重启或者重新加载**

难道每次运维修改配置文件，客户端都需要重启？

## Config动态刷新之手动版

### 配置

客户端的 pom 必须依赖 spring 的健康检查 actuator 监控。

修改cloud-config-client-3355的bootstrap.yml文件，暴露监控端口

```yaml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    #Config客户端配置
    config:
      label: master #分支名称
      name: config #配置文件名称
      profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
      uri: http://config-3344.com:3344 #配置中心地址


#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka

# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

业务类Controller添加注解@RefreshScope，让每次调用之前都重新加载

```java
package demo.yangxu.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RefreshScope  //刷新获得的总体配置文件
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo()
    {
        return configInfo;
    }
}

```

### 测试

此时再次修改 GitHub上配置文件的信息，然后使用服务端 3344 `http://config-3344.com:3344/master/config-dev.yml` 自测，然后调用客户端 3355 `http://localhost:3355/configInfo` 测试，发现客户端获得的信息并没有更新，如果要获得正确的信息，还是需要重启。

这是因为这个**动态加载需要运维人员发送Post请求刷新3355才能生效。**

使用 cURL 进行测试

```bash
curl -X POST "http://192.168.25.146:3355/actuator/refresh"
```

结果：

```
["config.client.version","config.info"]
```

调用客户端 3355 `http://localhost:3355/configInfo` 测试，此时客户端的信息得到更新，避免了服务重启。

遗憾的是，每次更新配置文件后都需要发送一个刷新客户端的请求。

如果需要一次通知，处处生效，进行大范围地自动刷新，可以使用广播这种方法。或者是需要精准通知部分客户端进行刷新。这种需求需要使用**服务总线 Bus**来实现。
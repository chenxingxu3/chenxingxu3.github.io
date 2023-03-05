---
layout:     post
title:      01.Spring Cloud Alibaba学习笔记
subtitle:   简介
date:       2020-08-04
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloudAlibaba
---
# 01.Spring Cloud Alibaba学习笔记--简介

## 为什么会出现 Spring Cloud Alibaba

Spring Cloud Netflix 项目进入了维护模式。

>Spring Cloud Netflix Projects Entering Maintenance Mode
>
>Recently, Netflix [announced](https://github.com/Netflix/Hystrix#hystrix-status) that Hystrix is entering maintenance mode. Ribbon has been in a [similar state](https://github.com/Netflix/ribbon#project-status-on-maintenance) since 2016. Although Hystrix and Ribbon are now in maintenance mode, they are still deployed at scale at Netflix.
>
>The Hystrix Dashboard and Turbine have been superseded by Atlas. The last commits to these project are 2 years and 4 years ago respectively. Zuul 1 and Archaius 1 have both been superseded by later versions that are not backward compatible.
>
>The following Spring Cloud Netflix modules and corresponding starters will be placed into maintenance mode:
>
>1. spring-cloud-netflix-archaius
>2. spring-cloud-netflix-hystrix-contract
>3. spring-cloud-netflix-hystrix-dashboard
>4. spring-cloud-netflix-hystrix-stream
>5. spring-cloud-netflix-hystrix
>6. spring-cloud-netflix-ribbon
>7. spring-cloud-netflix-turbine-stream
>8. spring-cloud-netflix-turbine
>9. spring-cloud-netflix-zuul
>
>This does not include the Eureka or concurrency-limits modules.
>
>What is Maintenance Mode?
>
>Placing a module in maintenance mode means that the Spring Cloud team will no longer be adding new features to the module. We will fix blocker bugs and security issues, and we will also consider and review small pull requests from the community.
>
>We intend to continue to support these modules for a period of at least a year from the general availability of the Greenwich release train.

参考：

https://spring.io/blog/2018/12/12/spring-cloud-greenwich-rc1-available-now

## 主要功能

- **服务限流降级**：默认支持 WebServlet、WebFlux, OpenFeign、RestTemplate、Spring Cloud Gateway, Zuul, Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。
- **服务注册与发现**：适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。
- **分布式配置管理**：支持分布式系统中的外部化配置，配置更改时自动刷新。
- **消息驱动能力**：基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。
- **分布式事务**：使用 @GlobalTransactional 注解， 高效并且对业务零侵入地解决分布式事务问题。。
- **阿里云对象存储**：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- **分布式任务调度**：提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行。
- **阿里云短信服务**：覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。



## 组件

- Sentinel：把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

- Nacos：一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

- RocketMQ：一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。

- Dubbo：Apache Dubbo™ 是一款高性能 Java RPC 框架。

- Seata：阿里巴巴开源产品，一个易于使用的高性能微服务分布式事务解决方案。

- Alibaba Cloud ACM：一款在分布式架构环境中对应用配置进行集中管理和推送的应用配置中心产品。

- Alibaba Cloud OSS: 阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。

- Alibaba Cloud SchedulerX: 阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。

- Alibaba Cloud SMS: 覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。

## 如何引入依赖

如果需要使用已发布的版本，在 `dependencyManagement` 中添加如下配置。

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.0.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

然后在 `dependencies` 中添加自己所需使用的依赖即可使用。

参考：

https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md

## 相关资料

https://spring.io/projects/spring-cloud-alibaba

https://github.com/alibaba/spring-cloud-alibaba

https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html

https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md
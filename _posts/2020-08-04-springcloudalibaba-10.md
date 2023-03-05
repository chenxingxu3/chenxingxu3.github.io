---
layout:     post
title:      10.Spring Cloud Alibaba学习笔记
subtitle:   Sentinel持久化规则
date:       2020-08-04
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloudAlibaba
---
# 10.Spring Cloud Alibaba学习笔记--Sentinel持久化规则

## 问题描述

一旦重启应用，Sentinel 规则消失。然而在生产环境需要将配置规则进行持久化。

## 持久化方法

将限流规则持久进 Nacos 保存，只要刷新 cloudalibaba-sentinel-server8401 某个rest 地址，Sentinel 控制台的流控规则就能看到，只要 Nacos 里面的配置不删除，针对 cloudalibaba-sentinel-server8401 上的流控规则持续有效。

## 步骤

### POM

确保添加了以下依赖：

```xml
<!--SpringCloud ailibaba sentinel-datasource-nacos 持久化-->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

### YAML

添加 Nacos 数据源配置

```yaml
server:
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
    sentinel:
      transport:
        dashboard: localhost:8080 #配置Sentinel dashboard地址
        #默认8719端口，假如被占用会自动从8719开始依次+1扫描
        #直至找到未被占用的端口
        port: 8719
      datasource:
        ds1:
          nacos:
            server-addr: localhost:8848
            dataId: cloudalibaba-sentinel-service
            groupId: DEFAULT_GROUP
            data-type: json
            #规则类型: 流控规则
            rule-type: flow

management:
  endpoints:
    web:
      exposure:
        include: '*'

feign:
  sentinel:
    enabled: true # 激活Sentinel对Feign的支持

```

spring.cloud.sentinel.datasource.ds.nacos.rule-type 的全部类型可以参考：

com.alibaba.cloud.sentinel.datasource.RuleType

```java
/**
* flow 流控规则
*/
FLOW("flow", FlowRule.class),
/**
* degrade 降级规则
*/
DEGRADE("degrade", DegradeRule.class),
/**
* param flow 热点规则
*/
PARAM_FLOW("param-flow", ParamFlowRule.class),
/**
* system 系统规则
*/
SYSTEM("system", SystemRule.class),
/**
* authority 授权规则
*/
AUTHORITY("authority", AuthorityRule.class),
/**
* gateway flow 网关限流规则
*/
GW_FLOW("gw-flow",
    "com.alibaba.csp.sentinel.adapter.gateway.common.rule.GatewayFlowRule"),
/**
* api 用户自定义的 API 定义分组，可以看做是一些 URL 匹配的组合
*/
GW_API_GROUP("gw-api-group",
    "com.alibaba.csp.sentinel.adapter.gateway.common.api.ApiDefinition");
```

### 添加 Nacos 业务规则配置

![](/img-post/2020-08-04-springcloudalibaba/08-05.png)

![](/img-post/2020-08-04-springcloudalibaba/08-06.png)

```json
[
  {
    "resource": "/rateLimit/byUrl",
    "limitApp": "default",
    "grade": 1,
    "count": 1,
    "strategy": 0,
    "controlBehavior": 0,
    "clusterMode": false
  }
]
```

参数说明：

- resource: 资源名称
- limitApp: 来源应用
- grade: 阈值类型，0 为线程数，1 为 QPS
- count: 单机阈值
- strategy: 流控模式，0 为直接，1 为关联，2 为链路
- controlBehavior: 流控效果，0 为快速失败，1 为 Warm Up，2 为排队等待
- clusterMode: 是否为集群

### 启动8401刷新Sentinel

访问

`http://localhost:8401/rateLimit/byUrl`

刷新 Sentinel

可以看到业务规则有了。

![](/img-post/2020-08-04-springcloudalibaba/08-07.png)

### 停止8401刷新Sentinel

![](/img-post/2020-08-04-springcloudalibaba/08-08.png)

流控规则已经没有了。

### 重新启动8401刷新Sentinel

访问

`http://localhost:8401/rateLimit/byUrl`

刷新 Sentinel

可以看到业务规则又有了，持久化验证通过。


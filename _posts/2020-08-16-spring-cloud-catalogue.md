---
layout:     post
title:      01.SpringCloud学习笔记
subtitle:   Boot和Cloud版本选择
date:       2020-08-16
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloud
---
# 01.SpringCloud学习笔记--Boot和Cloud版本选择

## 官网相关链接

### Spring Boot 

**git 源码地址**

https://github.com/spring-projects/spring-boot/releases

**Spring Boot 2.0 Release Notes**

https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Release-Notes

通过此链接的内容，可以得知：

> If you’re currently running with an earlier version of Spring Boot, we strongly recommend that you upgrade to Spring Boot 1.5 before migrating to Spring Boot 2.0.

**官网地址**

https://spring.io/projects/spring-boot

### Spring Cloud

**git 源码地址**

https://github.com/spring-projects/spring-cloud

**官网地址**

https://spring.io/projects/spring-cloud

## 版本选择

### 依据

| Release Train | Boot Version |
| :------------ | :----------- |
| Hoxton        | 2.2.x        |
| Greenwich     | 2.1.x        |
| Finchley      | 2.0.x        |
| Edgware       | 1.5.x        |
| Dalston       | 1.5.x        |

参考：https://spring.io/projects/spring-cloud#overview

```json
{
  "git": {
    "branch": "c68416883aa87c01ec06e873c69509264d043cbe",
    "commit": {
      "id": "c684168",
      "time": "2020-06-25T07:55:52Z"
    }
  },
  "build": {
    "version": "0.0.1-SNAPSHOT",
    "artifact": "start-site",
    "versions": {
      "spring-boot": "2.3.1.RELEASE",
      "initializr": "0.9.0.BUILD-SNAPSHOT"
    },
    "name": "start.spring.io website",
    "time": "2020-06-25T13:30:32.867Z",
    "group": "io.spring.start"
  },
  "bom-ranges": {
    "azure": {
      "2.0.10": "Spring Boot >=2.0.0.RELEASE and <2.1.0.RELEASE",
      "2.1.10": "Spring Boot >=2.1.0.RELEASE and <2.2.0.M1",
      "2.2.4": "Spring Boot >=2.2.0.M1 and <2.3.0.M1",
      "2.3.1": "Spring Boot >=2.3.0.M1"
    },
    "codecentric-spring-boot-admin": {
      "2.0.6": "Spring Boot >=2.0.0.M1 and <2.1.0.M1",
      "2.1.6": "Spring Boot >=2.1.0.M1 and <2.2.0.M1",
      "2.2.3": "Spring Boot >=2.2.0.M1"
    },
    "solace-spring-boot": {
      "1.0.0": "Spring Boot >=2.2.0.RELEASE and <2.3.0.M1",
      "1.1.0": "Spring Boot >=2.3.0.M1"
    },
    "solace-spring-cloud": {
      "1.0.0": "Spring Boot >=2.2.0.RELEASE and <2.3.0.M1"
    },
    "spring-cloud": {
      "Finchley.M2": "Spring Boot >=2.0.0.M3 and <2.0.0.M5",
      "Finchley.M3": "Spring Boot >=2.0.0.M5 and <=2.0.0.M5",
      "Finchley.M4": "Spring Boot >=2.0.0.M6 and <=2.0.0.M6",
      "Finchley.M5": "Spring Boot >=2.0.0.M7 and <=2.0.0.M7",
      "Finchley.M6": "Spring Boot >=2.0.0.RC1 and <=2.0.0.RC1",
      "Finchley.M7": "Spring Boot >=2.0.0.RC2 and <=2.0.0.RC2",
      "Finchley.M9": "Spring Boot >=2.0.0.RELEASE and <=2.0.0.RELEASE",
      "Finchley.RC1": "Spring Boot >=2.0.1.RELEASE and <2.0.2.RELEASE",
      "Finchley.RC2": "Spring Boot >=2.0.2.RELEASE and <2.0.3.RELEASE",
      "Finchley.SR4": "Spring Boot >=2.0.3.RELEASE and <2.0.999.BUILD-SNAPSHOT",
      "Finchley.BUILD-SNAPSHOT": "Spring Boot >=2.0.999.BUILD-SNAPSHOT and <2.1.0.M3",
      "Greenwich.M1": "Spring Boot >=2.1.0.M3 and <2.1.0.RELEASE",
      "Greenwich.SR6": "Spring Boot >=2.1.0.RELEASE and <2.1.16.BUILD-SNAPSHOT",
      "Greenwich.BUILD-SNAPSHOT": "Spring Boot >=2.1.16.BUILD-SNAPSHOT and <2.2.0.M4",
      "Hoxton.SR6": "Spring Boot >=2.2.0.M4 and <2.3.2.BUILD-SNAPSHOT",
      "Hoxton.BUILD-SNAPSHOT": "Spring Boot >=2.3.2.BUILD-SNAPSHOT and <2.4.0.M1",
      "2020.0.0-SNAPSHOT": "Spring Boot >=2.4.0.M1"
    },
    "spring-cloud-alibaba": {
      "2.2.1.RELEASE": "Spring Boot >=2.2.0.RELEASE and <2.3.0.M1"
    },
    "spring-cloud-services": {
      "2.0.3.RELEASE": "Spring Boot >=2.0.0.RELEASE and <2.1.0.RELEASE",
      "2.1.7.RELEASE": "Spring Boot >=2.1.0.RELEASE and <2.2.0.RELEASE",
      "2.2.3.RELEASE": "Spring Boot >=2.2.0.RELEASE and <2.3.0.M1"
    },
    "spring-statemachine": {
      "2.0.0.M4": "Spring Boot >=2.0.0.RC1 and <=2.0.0.RC1",
      "2.0.0.M5": "Spring Boot >=2.0.0.RC2 and <=2.0.0.RC2",
      "2.0.1.RELEASE": "Spring Boot >=2.0.0.RELEASE"
    },
    "vaadin": {
      "10.0.17": "Spring Boot >=2.0.0.M1 and <2.1.0.M1",
      "14.2.2": "Spring Boot >=2.1.0.M1 and <2.4.0-M1"
    },
    "wavefront": {
      "2.0.0-SNAPSHOT": "Spring Boot >=2.1.0.RELEASE"
    }
  },
  "dependency-ranges": {
    "okta": {
      "1.2.1": "Spring Boot >=2.1.2.RELEASE and <2.2.0.M1",
      "1.4.0": "Spring Boot >=2.2.0.M1 and <2.4.0-M1"
    },
    "mybatis": {
      "2.0.1": "Spring Boot >=2.0.0.RELEASE and <2.1.0.RELEASE",
      "2.1.3": "Spring Boot >=2.1.0.RELEASE and <2.4.0-M1"
    },
    "geode": {
      "1.2.8.RELEASE": "Spring Boot >=2.2.0.M5 and <2.3.0.M1",
      "1.3.0.RELEASE": "Spring Boot >=2.3.0.M1 and <2.4.0-M1"
    },
    "camel": {
      "2.22.4": "Spring Boot >=2.0.0.M1 and <2.1.0.M1",
      "2.25.1": "Spring Boot >=2.1.0.M1 and <2.2.0.M1",
      "3.3.0": "Spring Boot >=2.2.0.M1 and <2.3.0.M1",
      "3.4.0": "Spring Boot >=2.3.0.M1 and <2.4.0-M1"
    },
    "open-service-broker": {
      "2.1.3.RELEASE": "Spring Boot >=2.0.0.RELEASE and <2.1.0.M1",
      "3.0.4.RELEASE": "Spring Boot >=2.1.0.M1 and <2.2.0.M1",
      "3.1.1.RELEASE": "Spring Boot >=2.2.0.M1 and <2.4.0-M1"
    }
  }
}
```

参考：https://start.spring.io/actuator/info

>Spring Cloud provides tools for developers to quickly build some of the common patterns in distributed systems (e.g. configuration management, service discovery, circuit breakers, intelligent routing, micro-proxy, control bus). Coordination of distributed systems leads to boiler plate patterns, and using Spring Cloud developers can quickly stand up services and applications that implement those patterns. They will work well in any distributed environment, including the developer’s own laptop, bare metal data centres, and managed platforms such as Cloud Foundry.
>
>Release Train Version: **Hoxton.SR1**
>
>Supported Boot Version: **2.2.2.RELEASE**

参考：https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/

本笔记使用的版本

- Cloud ----- Hoxton.SR1
- Boot ----- 2.2.2.RELEASE
- Cloud Alibaba ----- 2.1.0.RELEASE
- Java ----- Java 8
- MySQL ----- 5.7 及以上
- Maven ----- 3.5 及以上
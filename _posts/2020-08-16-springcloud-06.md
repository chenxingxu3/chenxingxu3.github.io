---
layout:     post
title:      06.SpringCloud学习笔记
subtitle:   热部署Devtools
date:       2020-08-16
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloud
---
# 06.SpringCloud学习笔记--热部署Devtools

仅建议在开发阶段开启热部署，实际上线后，生产环境中应关闭热部署。

## 步骤

**1、Adding devtools to your project**

cloud2020\cloud-provider-payment8001\pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

**2、Adding plugin to your pom.xml**

cloud2020\pom.xml

```xml
<build>
    <finalName>你自己的工程名字</finalName>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
          <fork>true</fork>
          <addResources>true</addResources>
        </configuration>
      </plugin>
    </plugins>
</build>
```

**3、Enabling automatic build**

![](/img-post/2020-08-16-springcloud/06-01.jpg)

**4、Update the value of**

`ctrl+shift+alt+/` --> Registry

- compiler.automake.allow.when.app.running
- actionSystem.assertFocusAccessFromEdt

![](/img-post/2020-08-16-springcloud/06-02.jpg)

**5、重启 IDEA**

## 解决 spring boot devtool 热部署后出现访问 404 问题

DevTools 的检测时间和 idea 的编译所需时间存在差异。在 idea 还没完成编译工作前，DevTools 就开始进行重启和加载，导致 @RequestMapping 没有被全部正常处理。最简单的方法：牺牲一点时间，去加长 devtools 的轮询时间，增大等待时间，如下：

```yaml
spring:
  devtools:
    restart:
      poll-interval: 3000ms
      quiet-period: 2999ms
```

```properties
spring.devtools.restart.poll-interval=3000ms
spring.devtools.restart.quiet-period=2999ms
```

参考：https://stackoverflow.com/questions/39019938/springboot-devtools-restcontroller-not-always-mapped-when-rebuild-project
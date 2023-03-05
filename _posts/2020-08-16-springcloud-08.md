---
layout:     post
title:      08.SpringCloud学习笔记
subtitle:   工程重构
date:       2020-08-16
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloud
---
# 08.SpringCloud学习笔记--工程重构

## 开发流程

新建 cloud-api-commons

pom.xml

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

    <artifactId>cloud-api-commons</artifactId>

    <dependencies>
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
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.1.0</version>
        </dependency>
    </dependencies>
</project>
```

新建包 demo.yangxu.springcloud.entities

将 CommonResult 和 Payment 拷贝到 demo.yangxu.springcloud.entities 下

将 cloud-api-commons 安装到 Maven 仓库中，先 clean 再 install

删除 cloud-consumer-order80 和 cloud-provider-payment8001 下的 entities

引入自定义的 api 通用包，可以使用 Payment 支付 Entity

```xml
<dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
    <groupId>demo.yangxu.springcloud</groupId>
    <artifactId>cloud-api-commons</artifactId>
    <version>${project.version}</version>
</dependency>
```


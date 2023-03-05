---
layout:     post
title:      07.SpringCloud学习笔记
subtitle:   消费者订单模块
date:       2020-08-16
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloud
---
# 07.SpringCloud学习笔记--消费者订单模块

## RestTemplate

### 简介

RestTemplate 提供了多种便捷访问远程 HTTP 服务的方法，是一种简单便捷的访问 RESTful 服务模板类，是 Spring 提供的用于访问 REST 服务的客户端模板工具集。

### 官网及使用

https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html

使用 RestTemplate 访问 RESTful 接口非常简单粗暴。

(url, requestMap, ResponseBean.class) 这三个参数分别代表 REST 请求地址、请求参数、HTTP 响应转换被转换成的对象类型。

## 开发流程

新建 cloud-consumer-order80

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

    <artifactId>cloud-consumer-order80</artifactId>

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

resources\application.yml

```yaml
server:
  port: 80
```

demo.yangxu.springcloud.OrderMain80

```java
package demo.yangxu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class,args);
    }
}
```

demo.yangxu.springcloud.entities.Payment

```java
package demo.yangxu.springcloud.entities;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Payment implements Serializable {
    private Long id;
    private String serial;
}
```

demo.yangxu.springcloud.entities.CommonResult

```java
package demo.yangxu.springcloud.entities;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

//<T> 如果是 Payment 返回的就是 Payment
//如果是 Order 返回的就是Order
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CommonResult<T>
{
    private Integer code;
    private String  message;
    private T       data;

    public CommonResult(Integer code,String message)
    {
        this(code,message,null);
    }
}


```

demo.yangxu.springcloud.config.ApplicationContextConfig

```java
package demo.yangxu.springcloud.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContextConfig {
    
    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}

```

demo.yangxu.springcloud.controller.OrderController

```java
package demo.yangxu.springcloud.controller;

import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderController {
    
    public static final String PAYMENT_URL = "http://localhost:8001";
    
    @Resource
    private RestTemplate restTemplate;
    
    @GetMapping("/consumer/payment/create")
    public CommonResult<Payment> create(Payment payment){
        return restTemplate.postForObject(PAYMENT_URL + "/payment/create",payment,CommonResult.class);
        
    }
    
    @GetMapping("/consumer/payment/get/{id}")
    public CommonResult<Payment> getPayment(@PathVariable("id") Long id){
        return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id,CommonResult.class);
    }
}

```

测试

启动

- cloud-provider-payment8001
- cloud-consumer-order80

访问

> http://localhost/consumer/payment/get/31

获得正确结果

```json
{"code":200,"message":"查询成功","data":{"id":31,"serial":"aaabbb01"}}
```

解决插入数据只有 id 没有 serial 的问题

cloud-provider-payment8001 : demo.yangxu.springcloud.controller.PaymentController#create

```java
@PostMapping(value = "/payment/create")
public CommonResult create(@RequestBody Payment payment){
    int result = paymentService.create(payment);
    log.info("*****插入结果：" + result);

    if(result > 0){
        return new CommonResult(200,"插入数据库成功",result);
    }else{
        return new CommonResult(444,"插入数据库失败",null);
    }
}
```

测试

访问

> http://localhost/consumer/payment/create?serial=11111

## 解决 Run Dashboard 出不来的问题

### IDEA 2017

通过修改 IDEA 的 workspace.xml 来快速打开 Run Dashboard

![](/img-post/2020-08-16-springcloud/07-01.jpg)

![](/img-post/2020-08-16-springcloud/07-02.jpg)

在其中增加如下组件

```xml
<component name="RunDashboard">
<option name="configurationTypes">
  <set>
    <option value="SpringBootApplicationConfigurationType" />
  </set>
</option>
<option name="ruleStates">
  <list>
    <RuleState>
      <option name="name" value="ConfigurationTypeDashboardGroupingRule" />
    </RuleState>
    <RuleState>
      <option name="name" value="StatusDashboardGroupingRule" />
    </RuleState>
  </list>
</option>
</component>
```

重启 IDEA

在 View -> Tool Windows 中可以看到 Run Dashborad

![](/img-post/2020-08-16-springcloud/07-03.jpg)

### IDEA 2020

点击右上方的搜索图标

![](/img-post/2020-08-16-springcloud/07-04.jpg)

搜索 Run Dashboard

![](/img-post/2020-08-16-springcloud/07-05.jpg)
---
layout:     post
title:      19.SpringCloud学习笔记
subtitle:   Sleuth分布式请求链路跟踪
date:       2020-08-16
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloud
---
# 19.SpringCloud学习笔记--Sleuth分布式请求链路跟踪

## Sleuth 是什么

Spring Cloud Sleuth 官网：

https://docs.spring.io/spring-cloud-sleuth/docs/current-SNAPSHOT/reference/html/

https://github.com/spring-cloud/spring-cloud-sleuth

在微服务框架中，一个由客户端发起的请求，在后端系统中，会经过多个不同的的服务节点调用，来协同产生最后的请求结果。每个前段请求都会形成一个复杂的分布式服务调用链路，链路中的任何一环出现高延时或错误，都会引起整个请求最后的失败。

![](/img-post/2020-08-16-springcloud/19-01.jpg)

![](/img-post/2020-08-16-springcloud/19-02.jpg)

**Spring Cloud Sleuth 提供了一套完整的服务跟踪的解决方案**，在分布式系统中提供追踪解决方案并且兼容支持了zipkin。

![](/img-post/2020-08-16-springcloud/19-03.jpg)

## Sleuth 之 zipkin 搭建安装

Spring Cloud 从 F 版起不再需要自行构建 Zipkin server 了，只需要调用 jar 包即可。

### 下载

Zipkin 下载地址：

https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/2.12.9/

选择 zipkin-server-2.12.9-exec.jar 进行下载。

### 运行 jar

Zipkin 运行：

```bash
java -jar Zipkin-server-2.12.9-exec.jar
```

![](/img-post/2020-08-16-springcloud/19-04.jpg)

### 运行控制台

在浏览器中访问：

http://localhost:9411/zipkin

### 完整的调用链路

一条链路通过 Trace Id 唯一标识，Span 标识发起的请求信息，各 Span 通过 parent id 关联起来。

![](/img-post/2020-08-16-springcloud/19-05.jpg)

![](/img-post/2020-08-16-springcloud/19-06.jpg)

整个链路的依赖关系如下：

![](/img-post/2020-08-16-springcloud/19-07.jpg)

- Trace: 类似于树结构的 Span 集合，表示一条调用链路，存在唯一标识
- Span: 表示调用链路来源，通俗的理解 Span 就是一次请求信息

## Sleuth链路监控展现

### 服务提供者

cloud-provider-payment8001

#### POM

添加依赖：

```xml
<!--包含了sleuth+zipkin-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```



#### YML

```yaml
server:
  port: 8001

spring:
  application:
    name: cloud-payment-service
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
    #采样率值介于 0 到 1 之间，1 则表示全部采集
    #一般采用0.5就够用了
    probability: 1
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包
    url: jdbc:mysql://mysql:3306/db2019?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: root

  devtools:
    restart:
      poll-interval: 3000ms
      quiet-period: 2999ms

eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
      #单机版
      defaultZone: http://localhost:7001/eureka
      # 集群版
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka

  instance:
    instance-id: payment8001
    #访问路径可以显示IP地址
    prefer-ip-address: true
    #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
    lease-renewal-interval-in-seconds: 1
    #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
    lease-expiration-duration-in-seconds: 2


mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: demo.yangxu.springcloud.entities    # 所有Entity别名类所在包


```



#### 业务类

PaymentController

```java
package demo.yangxu.springcloud.controller;

import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import demo.yangxu.springcloud.service.PaymentService;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;
import java.util.List;
import java.util.concurrent.TimeUnit;

@RestController
@Slf4j
public class PaymentController {

    @Resource
    private PaymentService paymentService;

    @Resource
    private DiscoveryClient discoveryClient;

    @Value("${server.port}")
    private String serverPort;

    @PostMapping(value = "/payment/create")
    public CommonResult create(@RequestBody Payment payment){
        int result = paymentService.create(payment);
        log.info("*****插入结果：" + result);

        if(result > 0){
            return new CommonResult(200,"插入数据库成功,serverPort:"+serverPort,result);
        }else{
            return new CommonResult(444,"插入数据库失败",null);
        }
    }

    @GetMapping(value = "/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
        Payment payment = paymentService.getPaymentById(id);
        log.info("*****查询结果：" + payment + "热部署测试123456");

        if(payment != null){
            return new CommonResult(200,"查询成功,serverPort:"+serverPort,payment);
        }else{
            return new CommonResult(444,"没有对应记录，查询ID："+id,null);
        }
    }

    @GetMapping(value="/payment/discovery")
    public Object discovery(){
        //服务清单列表
        List<String> services = discoveryClient.getServices();
        for (String element : services) {
            log.info("*******element: "+element);
        }

        //一个微服务下的全部实例
        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        for (ServiceInstance instance : instances) {
            log.info(instance.getServiceId()+"\t"+instance.getHost()+"\t"+instance.getPort()+"\t"+instance.getUri());
        }

        return this.discoveryClient;
    }

    @GetMapping(value = "/payment/lb")
    public String getPaymentLB(){
        return serverPort;
    }

    @GetMapping(value = "/payment/feign/timeout")
    public String paymentFeignTimeout(){
        try {
            TimeUnit.SECONDS.sleep(3);
        }catch (InterruptedException e){
            e.printStackTrace();
        }

        return serverPort;
    }

    @GetMapping("/payment/zipkin")
    public String paymentZipkin()
    {
        return "hi ,i'am paymentzipkin server fall back，welcome to https://java2016.blog.csdn.net/，O(∩_∩)O哈哈~";
    }

}

```

### 服务消费者

cloud-consumer-order80

#### POM

添加依赖：

```xml
<!--包含了sleuth+zipkin-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

#### YML

```yaml
server:
  port: 80

spring:
  application:
    name: cloud-order-service

  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
      #采样率值介于 0 到 1 之间，1 则表示全部采集
      #一般采用0.5就够用了
      probability: 1

  devtools:
    restart:
      poll-interval: 3000ms
      quiet-period: 2999ms


eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
      #单机
      defaultZone: http://localhost:7001/eureka
      # 集群版
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka

```

#### 业务类

OrderController

```java
package demo.yangxu.springcloud.controller;

import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import demo.yangxu.springcloud.lb.LoadBalancer;
import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;
import java.net.URI;
import java.util.List;

@RestController
@Slf4j
public class OrderController {

    public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";

    @Resource
    private RestTemplate restTemplate;

    @Resource
    private LoadBalancer loadBalancer;

    @Resource
    private DiscoveryClient discoveryClient;

    @GetMapping("/consumer/payment/create")
    public CommonResult<Payment> create(Payment payment){
        return restTemplate.postForObject(PAYMENT_URL + "/payment/create",payment,CommonResult.class);

    }

    @GetMapping("/consumer/payment/postForEntity/create")
    public CommonResult<Payment> create2(Payment payment){
        return restTemplate.postForEntity(PAYMENT_URL + "/payment/create",payment,CommonResult.class).getBody();

    }

    @GetMapping("/consumer/payment/get/{id}")
    public CommonResult<Payment> getPayment(@PathVariable("id") Long id){
        return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id,CommonResult.class);
    }

    @GetMapping("/consumer/payment/getForEntity/{id}")
    public CommonResult<Payment> getPayment2(@PathVariable("id") Long id) {
        ResponseEntity<CommonResult> entity = restTemplate.getForEntity(PAYMENT_URL+"/payment/get/"+id,CommonResult.class);

        if(entity.getStatusCode().is2xxSuccessful()){
            log.info(entity.getStatusCode()+"\t" + entity.getHeaders());
            return entity.getBody();
        }else{
            return new CommonResult<>(444,"操作失败");
        }
    }

    @GetMapping(value = "/consumer/payment/lb")
    public String getPaymentLB(){
        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");

        if(instances == null || instances.size() <= 0){
            return null;
        }

        ServiceInstance serviceInstance = loadBalancer.instances(instances);
        URI uri = serviceInstance.getUri();

        return restTemplate.getForObject(uri+"/payment/lb", String.class);
    }

    // ====================> zipkin+sleuth
    @GetMapping("/consumer/payment/zipkin")
    public String paymentZipkin()
    {
        String result = restTemplate.getForObject("http://localhost:8001"+"/payment/zipkin/", String.class);
        return result;
    }

}

```



### 测试

启动：

- cloud-eureka-server7001
- cloud-provider-payment8001
- cloud-consumer-order80

80 调用 8001 几次测试下。

浏览器访问：

http://localhost/consumer/payment/zipkin

浏览器访问：

http://localhost:9411/zipkin/

可以看到以下信息：

![](/img-post/2020-08-16-springcloud/19-08.jpg)

![](/img-post/2020-08-16-springcloud/19-09.jpg)

![](/img-post/2020-08-16-springcloud/19-10.jpg)
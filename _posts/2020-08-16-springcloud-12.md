---
layout:     post
title:      12.SpringCloud学习笔记
subtitle:   Ribbon
date:       2020-08-16
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloud
---
# 12.SpringCloud学习笔记--Ribbon

## Ribbon入门介绍

### 是什么

 Spring Cloud Ribbon是基于Netflix Ribbon实现的一套客户端负载均衡的工具。

简单的说，Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法，将Netflix的中间层服务连接在一起。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们也很容易使用Ribbon实现自定义的负载均衡算法。

### 官网资料

https://github.com/Netflix/ribbon/wiki/Getting-Started

https://github.com/Netflix/ribbon

>Project Status: On Maintenance
>
>Ribbon comprises of multiple components some of which are used in production internally and some of which were replaced by non-OSS solutions over time. This is because Netflix started moving into a more componentized architecture for RPC with a focus on single-responsibility modules. So each Ribbon component gets a different level of attention at this moment.
>
>More specifically, here are the components of Ribbon and their level of attention by our teams:
>
>- ribbon-core: **deployed at scale in production**
>- ribbon-eureka: **deployed at scale in production**
>- ribbon-evcache: **not used**
>- ribbon-guice: **not used**
>- ribbon-httpclient: **we use everything not under com.netflix.http4.ssl. Instead, we use an internal solution developed by our cloud security team**
>- ribbon-loadbalancer: **deployed at scale in production**
>- ribbon-test: **this is just an internal integration test suite**
>- ribbon-transport: **not used**
>- ribbon: **not used**

### 用途

**负载均衡LB（Load Banlance）是什么？**

简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA（高可用）。

常见的负载均衡有软件Nginx，LVS，硬件 F5等。

**Ribbon本地负载均衡客户端与Nginx服务器端负载均衡区别**

- Nginx是服务器负载均衡，客户端所有请求交给 Nginx，然后由 Nginx 实现转发请求。即负载均衡是由服务器端实现，集中式LB。
- Ribbon是本地负载均衡，在调用微服务接口时候，会在注册中心上获取注册信息服务列表之后缓存到 JVM 本地，从而在本地实现 RPC 远程服务调用技术。即在客户端实现负载均衡，进程内LB。

由上面的区别可以将负载均衡分为两种：

- 集中式LB：在服务的消费方和提供方之间使用独立的LB设施（Nginx）
- 进程内LB：将LB的逻辑继承到消费方，消费方从服务注册中心获知哪些提供方地址可用，然后自己进行选择（Ribbon）

Ribbon就是一个软负载均衡的客户端组件，就是负载均衡+RestTemplate调用

## Ribbon的负载均衡和Rest调用

Ribbon就是一个软负载均衡的客户端组件，他可以和其他所需请求的客户端结合使用，和eureka结合只是其中的一个实例。

![](/img-post/2020-08-16-springcloud/12-01.png)

Ribbon在工作时分成两步：

第一步先选择 EurekaServer ,它优先选择在同一个区域内负载较少的server；

第二步再根据用户指定的策略，在从server取到的服务注册列表中选择一个地址。

其中Ribbon提供了多种策略：比如轮询、随机和根据响应时间加权。

之前实现了负载均衡就是基于 Ribbon，其实在使用eureka的新版本时，默认就集成了Ribbon。

```xml
<!--eureka client-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

```

单独引入ribbon

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

### RestTemplate的使用

官网：https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html

| ` <T> ResponseEntity<T>` | `getForEntity(String url, Class responseType, Map uriVariables)`Retrieve a representation by doing a GET on the URI template. |
| ------------------------ | ------------------------------------------------------------ |
| ` <T> ResponseEntity<T>` | `getForEntity(String url, Class responseType, Object... uriVariables)`Retrieve an entity by doing a GET on the specified URL. |
| ` <T> ResponseEntity<T>` | `getForEntity(URI url, Class responseType)`Retrieve a representation by doing a GET on the URL . |
| ` T`                     | `getForObject(String url, Class responseType, Map uriVariables)`Retrieve a representation by doing a GET on the URI template. |
| ` T`                     | `getForObject(String url, Class responseType, Object... uriVariables)`Retrieve a representation by doing a GET on the specified URL. |
| ` T`                     | `getForObject(URI url, Class responseType)`Retrieve a representation by doing a GET on the URL . |

| ` <T> ResponseEntity<T>` | `postForEntity(String url, Object request, Class responseType, Map uriVariables)`Create a new resource by POSTing the given object to the URI template, and returns the response as [`HttpEntity`](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/http/HttpEntity.html). |
| ------------------------ | ------------------------------------------------------------ |
| ` <T> ResponseEntity<T>` | `postForEntity(String url, Object request, Class responseType, Object... uriVariables)`Create a new resource by POSTing the given object to the URI template, and returns the response as [`ResponseEntity`](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/http/ResponseEntity.html). |
| ` <T> ResponseEntity<T>` | `postForEntity(URI url, Object request, Class responseType)`Create a new resource by POSTing the given object to the URL, and returns the response as [`ResponseEntity`](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/http/ResponseEntity.html). |
| ` T`                     | `postForObject(String url, Object request, Class responseType, Map uriVariables)`Create a new resource by POSTing the given object to the URI template, and returns the representation found in the response. |
| ` T`                     | `postForObject(String url, Object request, Class responseType, Object... uriVariables)`Create a new resource by POSTing the given object to the URI template, and returns the representation found in the response. |
| ` T`                     | `postForObject(URI url, Object request, Class responseType)`Create a new resource by POSTing the given object to the URL, and returns the representation found in the response. |

**getForObject / postForObject（推荐使用）**

返回对象为响应体中数据转化成的对象，基本上可以理解为Json

```java
@GetMapping("/consumer/payment/get/{id}")
public CommonResult<Payment> getPayment(@PathVariable("id") Long id){
    return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id,CommonResult.class);
}

```

```java
@GetMapping("/consumer/payment/create")
public CommonResult<Payment> create(Payment payment){
    return restTemplate.postForObject(PAYMENT_URL + "/payment/create",payment,CommonResult.class);

}
```

**getForEntity / postForEntity**

返回对象为ResponseEntity对象，包含了响应中的一些重要信息，比如响应头、相应状态码，响应体等

```java
@GetMapping("/consumer/payment/getForEntity/{id}")
public CommonResult<Payment> getPayment2(@PathVariable("id") Long id)
{
    ResponseEntity<CommonResult> entity = restTemplate.getForEntity(PAYMENT_URL+"/payment/get/"+id,CommonResult.class);

    if(entity.getStatusCode().is2xxSuccessful()){
        return entity.getBody();
    }else{
        return new CommonResult<>(444,"操作失败");
    }
}
```

```java
@GetMapping("/consumer/payment/postForEntity/create")
public CommonResult<Payment> create2(Payment payment){
    return restTemplate.postForEntity(PAYMENT_URL + "/payment/create",payment,CommonResult.class).getBody();

}
```

修改后的cloud-consumer-order80\src\main\java\demo\yangxu\springcloud\controller\OrderController.java

```java
package demo.yangxu.springcloud.controller;

import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderController {

    public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";

    @Resource
    private RestTemplate restTemplate;

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
}

```

测试

http://localhost/consumer/payment/getForEntity/31

```json
{"code":200,"message":"查询成功,serverPort:8002","data":{"id":31,"serial":"aaabbb01"}}
```

日志信息

```
2020-07-23 20:08:10.760  INFO 5608 --- [p-nio-80-exec-1] d.y.s.controller.OrderController         : 200 OK	[Content-Type:"application/json", Transfer-Encoding:"chunked", Date:"Thu, 23 Jul 2020 12:08:10 GMT", Keep-Alive:"timeout=60", Connection:"keep-alive"]
```

## Ribbon默认自带的负载规则

IRule: 根据特定算法中从服务列表中选取一个要访问的服务

![](/img-post/2020-08-16-springcloud/12-02.jpg)

- com.netflix.loadbalancer.RoundRobinRule：轮询
- com.netflix.loadbalancer.RandomRule：随机
- com.netflix.loadbalancer.RetryRule：先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试，获取可用的服务
- WeightedResponseTimeRule：对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择
- BestAvailableRule：会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
- AvailabilityFilteringRule：先过滤掉故障实例，再选择并发较小的实例
- ZoneAvoidanceRule：默认规则，复合判断server所在区域的性能和server的可用性选择服务器

## Ribbon负载规则替换

### 修改cloud-consumer-order80

### 注意配置细节

规则：这个自定义配置类不能放在 @ComponentScan 所扫描的当前包下以及子包下，否则自定义的配置类就会被所有的 Ribbon 客户端所共享，达不到特殊化定制的目的了。而 @ComponentScan 所扫描的当前包下以及子包下则就是 Spring Boot 主程序所在的包下，因为 @SpringBootApplication 注解里就包含 @ComponentScan。


>Customizing the Ribbon Client
>
>You can configure some bits of a Ribbon client using external properties in `<client>.ribbon.*`, which is no different than using the Netflix APIs natively, except that you can use Spring Boot configuration files. The native options can be inspected as static fields in `CommonClientConfigKey` (part of ribbon-core).
>
>Spring Cloud also lets you take full control of the client by declaring additional configuration (on top of the `RibbonClientConfiguration`) using `@RibbonClient`. Example:
>
>```java
>@Configuration
>@RibbonClient(name = "foo", configuration = FooConfiguration.class)
>public class TestConfiguration {
>}
>```
>
>In this case the client is composed from the components already in `RibbonClientConfiguration` together with any in `FooConfiguration` (where the latter generally will override the former).
>
>| Warning | The `FooConfiguration` has to be `@Configuration` but take care that it is not in a `@ComponentScan` for the main application context, otherwise it will be shared by all the `@RibbonClients`. If you use `@ComponentScan` (or `@SpringBootApplication`) you need to take steps to avoid it being included (for instance put it in a separate, non-overlapping package, or specify the packages to scan explicitly in the `@ComponentScan`). |
>| ------- | ------------------------------------------------------------ |
>|         |                                                              |
>
>

参考：https://bushkarl.gitbooks.io/spring-cloud/content/spring_cloud_netflix/client_side_load_balancer_ribbon.html

### 新建package: demo.yangxu.myrule

### 上面的包下新建MySelfRule规则类

demo.yangxu.myrule.MySelfRule

```java
package demo.yangxu.myrule;

import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MySelfRule {

    @Bean
    public IRule myRule(){
        
        //轮询
        //return new RoundRobinRule();
        
        //随机
        return new RandomRule();
        
        //先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试，获取可用的服务
        //return new RetryRule();
        
        //对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择
        //return new WeightedResponseTimeRule();
        
        //会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
        //return new BestAvailableRule();
        
        //先过滤掉故障实例，再选择并发较小的实例
        //return new AvailabilityFilteringRule();
        
        //默认规则，复合判断server所在区域的性能和server的可用性选择服务器
        //return new ZoneAvoidanceRule();
    }
}

```



### 主启动类添加@RibbonClient

demo.yangxu.springcloud.OrderMain80

```java
package demo.yangxu.springcloud;

import demo.yangxu.myrule.MySelfRule;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.ribbon.RibbonClient;

@SpringBootApplication
@EnableEurekaClient
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MySelfRule.class)
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class,args);
    }
}

```



### 测试

启动

- EurekaMain7001
- EurekaMain7002
- PaymentMain8001
- PaymentMain8002
- OrderMain80

访问：http://localhost/consumer/payment/get/31

使用RandomRule的参考结果：

```json
{"code":200,"message":"查询成功,serverPort:8002","data":{"id":31,"serial":"aaabbb01"}}
{"code":200,"message":"查询成功,serverPort:8001","data":{"id":31,"serial":"aaabbb01"}}
{"code":200,"message":"查询成功,serverPort:8002","data":{"id":31,"serial":"aaabbb01"}}
{"code":200,"message":"查询成功,serverPort:8001","data":{"id":31,"serial":"aaabbb01"}}
{"code":200,"message":"查询成功,serverPort:8001","data":{"id":31,"serial":"aaabbb01"}}
{"code":200,"message":"查询成功,serverPort:8001","data":{"id":31,"serial":"aaabbb01"}}
{"code":200,"message":"查询成功,serverPort:8002","data":{"id":31,"serial":"aaabbb01"}}{"code":200,"message":"查询成功,serverPort:8002","data":{"id":31,"serial":"aaabbb01"}}
{"code":200,"message":"查询成功,serverPort:8001","data":{"id":31,"serial":"aaabbb01"}}
{"code":200,"message":"查询成功,serverPort:8002","data":{"id":31,"serial":"aaabbb01"}}
```

## Ribbon默认负载轮训算法原理

### 负载均衡算法

轮询算法：

**rest 接口第几次请求数 % 服务器集群总数量 = 时机调用服务器位置下标**

每次服务重启后 rest 接口计数从 1 开始。

```java
List<ServiceInstance> instances = discoverClient.getInstances("CLOUD-PROVIDER-SERVICE");
```

如： List [0] instances = 127.0.0.1:8002

```
List[1] instances = 127.0.0.1:8001
```

8001 + 8002 组合为集群，共计 2 台服务器，集群总数为 2 ， 按照轮询算法原理：

当请求总数为 1 时：1%2 = 1, 对应下标位置为 1， 则获得服务地址为 127.0.0.1：8001
当请求总数为 2 时：2%2 = 0, 对应下标位置为 0， 则获得服务地址为 127.0.0.1：8002
当请求总数为 3 时：2%2 = 1, 对应下标位置为 1， 则获得服务地址为 127.0.0.1：8001

**例如我们现在有两台服务器用于负载均衡**

| 请求次数 | 计算公式 | 取得下标            |
| -------- | -------- | ------------------- |
| 1        | 1%2=1    | 对应 127.0.0.1:8001 |
| 2        | 2%2=0    | 对应 127.0.0.1:8002 |
| 3        | 3%2=1    | 对应 127.0.0.1:8001 |
| …        | …        | …                   |

## RoundRobinRule源码

```java
/*
 *
 * Copyright 2013 Netflix, Inc.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 *
 */
package com.netflix.loadbalancer;

import com.netflix.client.config.IClientConfig;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * The most well known and basic load balancing strategy, i.e. Round Robin Rule.
 *
 * @author stonse
 * @author Nikos Michalakis <nikos@netflix.com>
 *
 */
public class RoundRobinRule extends AbstractLoadBalancerRule {

    private AtomicInteger nextServerCyclicCounter;
    private static final boolean AVAILABLE_ONLY_SERVERS = true;
    private static final boolean ALL_SERVERS = false;

    private static Logger log = LoggerFactory.getLogger(RoundRobinRule.class);

    public RoundRobinRule() {
        nextServerCyclicCounter = new AtomicInteger(0);
    }

    public RoundRobinRule(ILoadBalancer lb) {
        this();
        setLoadBalancer(lb);
    }

    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            log.warn("no load balancer");
            return null;
        }

        Server server = null;
        int count = 0;
        while (server == null && count++ < 10) {
            List<Server> reachableServers = lb.getReachableServers();
            List<Server> allServers = lb.getAllServers();
            int upCount = reachableServers.size();
            int serverCount = allServers.size();

            if ((upCount == 0) || (serverCount == 0)) {
                log.warn("No up servers available from load balancer: " + lb);
                return null;
            }

            int nextServerIndex = incrementAndGetModulo(serverCount);
            server = allServers.get(nextServerIndex);

            if (server == null) {
                /* Transient. */
                Thread.yield();
                continue;
            }

            if (server.isAlive() && (server.isReadyToServe())) {
                return (server);
            }

            // Next.
            server = null;
        }

        if (count >= 10) {
            log.warn("No available alive servers after 10 tries from load balancer: "
                    + lb);
        }
        return server;
    }

    /**
     * Inspired by the implementation of {@link AtomicInteger#incrementAndGet()}.
     *
     * @param modulo The modulo to bound the value of the counter.
     * @return The next value.
     */
    private int incrementAndGetModulo(int modulo) {
        for (;;) {
            int current = nextServerCyclicCounter.get();
            int next = (current + 1) % modulo;
            if (nextServerCyclicCounter.compareAndSet(current, next))
                return next;
        }
    }

    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(), key);
    }

    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
    }
}

```

## Ribbon之手写轮询算法

### 7001 / 7002 集群启动

### 8001 / 8002 微服务改造

**Controller**

demo.yangxu.springcloud.controller.PaymentController

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

}

```



### 80订单微服务改造

#### ApplicationContextBean去掉注解@LoadBalanced

demo.yangxu.springcloud.config.ApplicationContextConfig

#### LoadBalancer接口

demo.yangxu.springcloud.lb.LoadBalancer

```java
package demo.yangxu.springcloud.lb;

import org.springframework.cloud.client.ServiceInstance;

import java.util.List;

public interface LoadBalancer {
    ServiceInstance instances(List<ServiceInstance> serviceInstances);
}

```



#### MyLB

demo.yangxu.springcloud.lb.MyLB

```java
package demo.yangxu.springcloud.lb;

import org.springframework.cloud.client.ServiceInstance;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;

@Component
public class MyLB implements LoadBalancer {

    private AtomicInteger atomicInteger = new AtomicInteger(0);

    public final int getAndIncrement(){
        int current;
        int next;

        do {
            current = this.atomicInteger.get();
            next = current >= 2147483647 ? 0 : current+1;
        }while(!this.atomicInteger.compareAndSet(current,next));
        System.out.println("*****第几次访问，次数next: "+next);
        return next;
    }

    //负载均衡算法：rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标  ，每次服务重启动后rest接口计数从1开始。
    @Override
    public ServiceInstance instances(List<ServiceInstance> serviceInstances) {
        int index = getAndIncrement() % serviceInstances.size();
        return serviceInstances.get(index);
    }
}

```



#### OrderController

demo.yangxu.springcloud.controller.OrderController

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
}

```



#### 测试

http://localhost/consumer/payment/lb
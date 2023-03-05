---
layout:     post
title:      09.Spring Cloud Alibaba学习笔记
subtitle:   Sentinel服务熔断
date:       2020-08-04
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloudAlibaba
---
# 09.Spring Cloud Alibaba学习笔记--Sentinel服务熔断

## 熔断框架比较

|                | Sentinel                        | Hystrix                | resilience4j           |
| -------------- | ------------------------------- | ---------------------- | ---------------------- |
| 隔离策略       | 信号量隔离（并发线程数限流）    | 线程池隔离/信号量隔离  | 信号量隔离             |
| 熔断降级策略   | 基于时间响应、异常比率、异常数  | 基于异常比率           | 基于异常比率、响应时间 |
| 实时统计实现   | 滑动窗口（LeapArray）           | 滑动窗口（基于RxJava） | Ring Bit Buffer        |
| 动态规则配置   | 支持多种数据源                  | 支持多种数据源         | 有限支持               |
| 扩展性         | 多个扩展点                      | 插件形式               | 接口形式               |
| 基于注解的支持 | 支持                            | 支持                   | 支持                   |
| 限流           | 基于QPS、支持基于调用关系的限流 | 有限的支持             | Rate Limiter           |

## Sentinel整合Ribbon

### 启动

- Nacos
- Sentinel

### 创建提供者 9003 / 9004 工程

#### 建 Module

- cloudalibaba-provider-payment9003
- cloudalibaba-provider-payment9004

#### POM

以 9003 为例，9004 内容类似。

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

    <artifactId>cloudalibaba-provider-payment9003</artifactId>

    <dependencies>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
            <groupId>demo.yangxu.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
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

#### YAML

以 9003 为例，9004 内容类似。

application.yaml

```yaml
server:
  port: 9003

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

#### 主启动

以 9003 为例，9004 内容类似。

```java
package demo.yangxu.springcloud.alibaba;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain9003 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain9003.class, args);
    }
}

```

#### 业务类

以 9003 为例，9004 内容类似。

PaymentController

```java
package demo.yangxu.springcloud.alibaba.controller;

import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;

@RestController
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    public static HashMap<Long,Payment> hashMap = new HashMap<>();

    static {
        hashMap.put(1L,new Payment(1L,"ae9efb59248d2cdc623eaf0b3f935e65"));
        hashMap.put(2L,new Payment(2L,"ecda3786504e1547b33d167f26a561d3"));
        hashMap.put(3L,new Payment(3L,"f3d1fc3dc268db5f9cacb289295286e6"));
    }

    @GetMapping(value = "/paymentSQL/{id}")
    public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id) {
        Payment payment = hashMap.get(id);
        CommonResult<Payment> result = new CommonResult(200,"from mysql,serverPort:  "+serverPort,payment);
        return result;
    }
}

```

### 创建消费者 84 工程

#### 建 Module

cloudalibaba-consumer-nacos-order84

#### POM

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

    <artifactId>cloudalibaba-consumer-nacos-order84</artifactId>

    <dependencies>
        <!--SpringCloud openfeign -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>demo.yangxu.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
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

#### YAML

application.yaml

```yaml
server:
  port: 84


spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    sentinel:
      transport:
        #配置Sentinel dashboard地址
        dashboard: localhost:8080
        #默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
        port: 8719

#消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
service-url:
  nacos-user-service: http://nacos-payment-provider

# 激活Sentinel对Feign的支持
feign:
  sentinel:
    enabled: true
```

#### 主启动

OrderNacosMain84

```java
package demo.yangxu.springcloud.alibaba;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@EnableDiscoveryClient
@SpringBootApplication
@EnableFeignClients
public class OrderNacosMain84 {
    public static void main(String[] args) {
        SpringApplication.run(OrderNacosMain84.class,args);
    }
}

```

#### 业务类

ApplicationContextConfig

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

### Sentinel服务熔断无配置

#### 业务类

CircleBreakerController

```java
package demo.yangxu.springcloud.alibaba.controller;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class CircleBreakerController {

    @Value("${service-url.nacos-user-service}")
    private String SERVICE_URL;

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback") //没有配置
    public CommonResult<Payment> fallback(@PathVariable Long id)
    {
        CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/"+id, CommonResult.class,id);

        if (id == 4) {
            throw new IllegalArgumentException ("IllegalArgumentException,非法参数异常....");
        }else if (result.getData() == null) {
            throw new NullPointerException ("NullPointerException,该ID没有对应记录,空指针异常");
        }

        return result;
    }

}

```

#### 测试

启动

- cloudalibaba-provider-payment9003
- cloudalibaba-provider-payment9004
- cloudalibaba-consumer-nacos-order84

访问

`http://localhost:84/consumer/fallback/1`

返回

```json
{"code":200,"message":"from mysql,serverPort:  9003","data":{"id":1,"serial":"ae9efb59248d2cdc623eaf0b3f935e65"}}
//负载均衡
{"code":200,"message":"from mysql,serverPort:  9004","data":{"id":1,"serial":"ae9efb59248d2cdc623eaf0b3f935e65"}}
```

访问

`http://localhost:84/consumer/fallback/4`

返回

`Whitelabel Error Page`: java.lang.IllegalArgumentException: IllegalArgumentException,非法参数异常....

访问

`http://localhost:84/consumer/fallback/5`

返回

`Whitelabel Error Page`: java.lang.NullPointerException: NullPointerException,该ID没有对应记录,空指针异常

### Sentinel服务熔断只配置fallback

#### 业务类

CircleBreakerController

```java
package demo.yangxu.springcloud.alibaba.controller;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class CircleBreakerController {

    @Value("${service-url.nacos-user-service}")
    private String SERVICE_URL;

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback",fallback = "handlerFallback") //fallback只负责业务异常
    public CommonResult<Payment> fallback(@PathVariable Long id) {
        CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/"+id, CommonResult.class,id);

        if (id == 4) {
            throw new IllegalArgumentException ("IllegalArgumentException,非法参数异常....");
        }else if (result.getData() == null) {
            throw new NullPointerException ("NullPointerException,该ID没有对应记录,空指针异常");
        }

        return result;
    }

    //本例是fallback
    public CommonResult handlerFallback(@PathVariable  Long id,Throwable e) {
        Payment payment = new Payment(id,"null");
        return new CommonResult<>(444,"兜底异常handlerFallback,exception内容  "+e.getMessage(),payment);
    }

}

```

#### 测试

启动

- cloudalibaba-provider-payment9003
- cloudalibaba-provider-payment9004
- cloudalibaba-consumer-nacos-order84

访问

`http://localhost:84/consumer/fallback/1`

返回

```json
{"code":200,"message":"from mysql,serverPort:  9003","data":{"id":1,"serial":"ae9efb59248d2cdc623eaf0b3f935e65"}}
//负载均衡
{"code":200,"message":"from mysql,serverPort:  9004","data":{"id":1,"serial":"ae9efb59248d2cdc623eaf0b3f935e65"}}
```

访问

`http://localhost:84/consumer/fallback/4`

返回

```json
{"code":444,"message":"兜底异常handlerFallback,exception内容  IllegalArgumentException,非法参数异常....","data":{"id":4,"serial":"null"}}
```

访问

`http://localhost:84/consumer/fallback/5`

返回

```json
{"code":444,"message":"兜底异常handlerFallback,exception内容  NullPointerException,该ID没有对应记录,空指针异常","data":{"id":5,"serial":"null"}}
```

### Sentinel服务熔断只配置blockHandler

#### 业务类

CircleBreakerController

```java
package demo.yangxu.springcloud.alibaba.controller;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class CircleBreakerController {

    @Value("${service-url.nacos-user-service}")
    private String SERVICE_URL;

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback",blockHandler = "blockHandler") //blockHandler只负责sentinel控制台配置违规
    public CommonResult<Payment> fallback(@PathVariable Long id) {
        CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/"+id, CommonResult.class,id);

        if (id == 4) {
            throw new IllegalArgumentException ("IllegalArgumentException,非法参数异常....");
        }else if (result.getData() == null) {
            throw new NullPointerException ("NullPointerException,该ID没有对应记录,空指针异常");
        }

        return result;
    }

    //本例是blockHandler
    public CommonResult blockHandler(@PathVariable  Long id, BlockException blockException) {
        Payment payment = new Payment(id,"null");
        return new CommonResult<>(445,"blockHandler-sentinel限流,无此流水: blockException  "+blockException.getMessage(),payment);
    }

}

```

#### Sentinel配置

启动

- cloudalibaba-provider-payment9003
- cloudalibaba-provider-payment9004
- cloudalibaba-consumer-nacos-order84

访问

`http://localhost:84/consumer/fallback/1`

配置降级规则

![](/img-post/2020-08-04-springcloudalibaba/09-01.png)

#### 测试

1、访问

`http://localhost:84/consumer/fallback/1`

返回

```json
{"code":200,"message":"from mysql,serverPort:  9003","data":{"id":1,"serial":"ae9efb59248d2cdc623eaf0b3f935e65"}}
//负载均衡
{"code":200,"message":"from mysql,serverPort:  9004","data":{"id":1,"serial":"ae9efb59248d2cdc623eaf0b3f935e65"}}
```

2、访问

`http://localhost:84/consumer/fallback/4`

返回

`Whitelabel Error Page`: java.lang.IllegalArgumentException: IllegalArgumentException,非法参数异常....

快速刷新

`http://localhost:84/consumer/fallback/4`

返回

```json
{"code":445,"message":"blockHandler-sentinel限流,无此流水: blockException  null","data":{"id":4,"serial":"null"}}
```

3、访问

`http://localhost:84/consumer/fallback/5`

返回

`Whitelabel Error Page`: java.lang.NullPointerException: NullPointerException,该ID没有对应记录,空指针异常

快速刷新

`http://localhost:84/consumer/fallback/5`

返回

```json
{"code":445,"message":"blockHandler-sentinel限流,无此流水: blockException  null","data":{"id":5,"serial":"null"}}
```

### Sentinel服务熔断fallback和blockHandle

#### 业务类

CircleBreakerController

```java
package demo.yangxu.springcloud.alibaba.controller;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class CircleBreakerController {

    @Value("${service-url.nacos-user-service}")
    private String SERVICE_URL;

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback",fallback = "handlerFallback",blockHandler = "blockHandler")
    public CommonResult<Payment> fallback(@PathVariable Long id) {
        CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/"+id, CommonResult.class,id);

        if (id == 4) {
            throw new IllegalArgumentException ("IllegalArgumentException,非法参数异常....");
        }else if (result.getData() == null) {
            throw new NullPointerException ("NullPointerException,该ID没有对应记录,空指针异常");
        }

        return result;
    }

    //本例是fallback
    public CommonResult handlerFallback(@PathVariable  Long id,Throwable e) {
        Payment payment = new Payment(id,"null");
        return new CommonResult<>(444,"兜底异常handlerFallback,exception内容  "+e.getMessage(),payment);
    }

    //本例是blockHandler
    public CommonResult blockHandler(@PathVariable  Long id, BlockException blockException) {
        Payment payment = new Payment(id,"null");
        return new CommonResult<>(445,"blockHandler-sentinel限流,无此流水: blockException  "+blockException.getMessage(),payment);
    }

}

```

#### Sentinel配置

启动

- cloudalibaba-provider-payment9003
- cloudalibaba-provider-payment9004
- cloudalibaba-consumer-nacos-order84

访问

`http://localhost:84/consumer/fallback/1`

配置流控规则

![](/img-post/2020-08-04-springcloudalibaba/09-02.png)

#### 测试

1、访问

`http://localhost:84/consumer/fallback/1`

返回

```json
{"code":200,"message":"from mysql,serverPort:  9003","data":{"id":1,"serial":"ae9efb59248d2cdc623eaf0b3f935e65"}}
//负载均衡
{"code":200,"message":"from mysql,serverPort:  9004","data":{"id":1,"serial":"ae9efb59248d2cdc623eaf0b3f935e65"}}
```

快速刷新

`http://localhost:84/consumer/fallback/1`

返回

```json
{"code":445,"message":"blockHandler-sentinel限流,无此流水: blockException  null","data":{"id":1,"serial":"null"}}
```

2、访问

`http://localhost:84/consumer/fallback/4`

返回

```json
{"code":444,"message":"兜底异常handlerFallback,exception内容  IllegalArgumentException,非法参数异常....","data":{"id":4,"serial":"null"}}
```

快速刷新

`http://localhost:84/consumer/fallback/4`

返回

```json
{"code":445,"message":"blockHandler-sentinel限流,无此流水: blockException  null","data":{"id":1,"serial":"null"}}
```

3、访问

`http://localhost:84/consumer/fallback/5`

返回

```json
{"code":444,"message":"兜底异常handlerFallback,exception内容  NullPointerException,该ID没有对应记录,空指针异常","data":{"id":5,"serial":"null"}}
```

快速刷新

`http://localhost:84/consumer/fallback/5`

返回

```json
{"code":445,"message":"blockHandler-sentinel限流,无此流水: blockException  null","data":{"id":5,"serial":"null"}}
```

### Sentinel服务熔断exceptionsToIgnore

![](/img-post/2020-08-04-springcloudalibaba/09-03.png)

#### 业务类

CircleBreakerController

```java
package demo.yangxu.springcloud.alibaba.controller;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class CircleBreakerController {

    @Value("${service-url.nacos-user-service}")
    private String SERVICE_URL;

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback",fallback = "handlerFallback",blockHandler = "blockHandler",
            exceptionsToIgnore = {IllegalArgumentException.class})
    public CommonResult<Payment> fallback(@PathVariable Long id) {
        CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/"+id, CommonResult.class,id);

        if (id == 4) {
            throw new IllegalArgumentException ("IllegalArgumentException,非法参数异常....");
        }else if (result.getData() == null) {
            throw new NullPointerException ("NullPointerException,该ID没有对应记录,空指针异常");
        }

        return result;
    }

    //本例是fallback
    public CommonResult handlerFallback(@PathVariable  Long id,Throwable e) {
        Payment payment = new Payment(id,"null");
        return new CommonResult<>(444,"兜底异常handlerFallback,exception内容  "+e.getMessage(),payment);
    }

    //本例是blockHandler
    public CommonResult blockHandler(@PathVariable  Long id, BlockException blockException) {
        Payment payment = new Payment(id,"null");
        return new CommonResult<>(445,"blockHandler-sentinel限流,无此流水: blockException  "+blockException.getMessage(),payment);
    }

}

```

#### 测试

启动

- cloudalibaba-provider-payment9003
- cloudalibaba-provider-payment9004
- cloudalibaba-consumer-nacos-order84

1、访问

`http://localhost:84/consumer/fallback/4`

返回

`Whitelabel Error Page`: java.lang.IllegalArgumentException: IllegalArgumentException,非法参数异常....

2、访问

`http://localhost:84/consumer/fallback/5`

返回

```json
{"code":444,"message":"兜底异常handlerFallback,exception内容  NullPointerException,该ID没有对应记录,空指针异常","data":{"id":5,"serial":"null"}}
```



### 总结

fallback 管运行时异常，blockHandler 管配置违规。

若 blockHandler 和 fallback 都进行了配置，则被限流降级而抛出 BlockException 时只会进入 blockHandler 处理逻辑。

## Sentinel整合OpenFeign

### 修改 cloudalibaba-consumer-nacos-order84

84 消费端调用提供者 9003 / 9004，OpenFeign 组件一般是消费侧。

#### POM

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

    <artifactId>cloudalibaba-consumer-nacos-order84</artifactId>

    <dependencies>
        <!--SpringCloud openfeign -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>demo.yangxu.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
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

#### YAML

application.yaml

```yaml
server:
  port: 84


spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    sentinel:
      transport:
        #配置Sentinel dashboard地址
        dashboard: localhost:8080
        #默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
        port: 8719

#消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
service-url:
  nacos-user-service: http://nacos-payment-provider

# 激活Sentinel对Feign的支持
feign:
  sentinel:
    enabled: true
```

#### 主启动

OrderNacosMain84

```java
package demo.yangxu.springcloud.alibaba;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@EnableDiscoveryClient
@SpringBootApplication
@EnableFeignClients
public class OrderNacosMain84 {
    public static void main(String[] args) {
        SpringApplication.run(OrderNacosMain84.class,args);
    }
}

```



#### 业务类

PaymentService

带 @FeignClient 注解的业务接口

```java
package demo.yangxu.springcloud.alibaba.service;

import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(value = "nacos-payment-provider",fallback = PaymentFallbackService.class)
public interface PaymentService {
    @GetMapping(value = "/paymentSQL/{id}")
    public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id);
}

```

PaymentFallbackService

```java
package demo.yangxu.springcloud.alibaba.service;

import demo.yangxu.springcloud.entities.CommonResult;
import demo.yangxu.springcloud.entities.Payment;
import org.springframework.stereotype.Component;

@Component
public class PaymentFallbackService implements PaymentService {
    @Override
    public CommonResult<Payment> paymentSQL(Long id) {
        return new CommonResult<>(44444,
                "服务降级返回,---PaymentFallbackService",
                new Payment(id,"errorSerial"));
    }
}

```

CircleBreakerController

添加以下代码：

```java
//==================OpenFeign
@Resource
private PaymentService paymentService;

@GetMapping(value = "/consumer/paymentSQL/{id}")
public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id) {
    return paymentService.paymentSQL(id);
}
```

### 测试

访问

`http://localhost:84/consumer/paymentSQL/1`

返回

```json
{"code":200,"message":"from mysql,serverPort:  9003","data":{"id":1,"serial":"ae9efb59248d2cdc623eaf0b3f935e65"}}
//负载均衡
{"code":200,"message":"from mysql,serverPort:  9004","data":{"id":1,"serial":"ae9efb59248d2cdc623eaf0b3f935e65"}}
```

关闭 9003 / 9004 微服务提供者

访问

`http://localhost:84/consumer/paymentSQL/1`

返回

```json
{"code":44444,"message":"服务降级返回,---PaymentFallbackService","data":{"id":1,"serial":"errorSerial"}}
```

84 消费侧会自动降级，不会被耗死。


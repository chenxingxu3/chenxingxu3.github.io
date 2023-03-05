---
layout:     post
title:      06.Spring Cloud Alibaba学习笔记
subtitle:   Sentinel降级
date:       2020-08-04
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloudAlibaba
---
# 06.Spring Cloud Alibaba学习笔记--Sentinel降级

## Sentinel降级简介

### 概述

除了流量控制以外，对调用链路中不稳定的资源进行熔断降级也是保障高可用的重要措施之一。由于调用关系的复杂性，如果调用链路中的某个资源不稳定，最终会导致请求发生堆积。Sentinel **熔断降级**会在调用链路中某个资源出现不稳定状态时（例如调用超时或异常比例升高），对这个资源的调用进行限制，让请求快速失败，避免影响到其它的资源而导致级联错误。当资源被降级后，在接下来的降级时间窗口之内，对该资源的调用都自动熔断（默认行为是抛出 `DegradeException`）。

参考：

[https://github.com/alibaba/Sentinel/wiki/%E7%86%94%E6%96%AD%E9%99%8D%E7%BA%A7](https://github.com/alibaba/Sentinel/wiki/熔断降级)

### 降级策略

![](/img-post/2020-08-04-springcloudalibaba/06-01.png)

![](/img-post/2020-08-04-springcloudalibaba/06-02.png)

![](/img-post/2020-08-04-springcloudalibaba/06-03.png)

我们通常用以下几种方式来衡量资源是否处于稳定的状态：

- 平均响应时间 (`DEGRADE_GRADE_RT`)：当 1s 内持续进入 N 个请求，对应时刻的平均响应时间（秒级）均超过阈值（`count`，以 ms 为单位），那么在接下的时间窗口（`DegradeRule` 中的 `timeWindow`，以 s 为单位）之内，对这个方法的调用都会自动地熔断（抛出 `DegradeException`）。注意 Sentinel 默认统计的 RT 上限是 4900 ms，**超出此阈值的都会算作 4900 ms**，若需要变更此上限可以通过启动配置项 `-Dcsp.sentinel.statistic.max.rt=xxx` 来配置。
- 异常比例 (`DEGRADE_GRADE_EXCEPTION_RATIO`)：当资源的每秒请求量 >= N（可配置），并且每秒异常总数占通过量的比值超过阈值（`DegradeRule` 中的 `count`）之后，资源进入降级状态，即在接下的时间窗口（`DegradeRule` 中的 `timeWindow`，以 s 为单位）之内，对这个方法的调用都会自动地返回。异常比率的阈值范围是 `[0.0, 1.0]`，代表 0% - 100%。
- 异常数 (`DEGRADE_GRADE_EXCEPTION_COUNT`)：当资源近 1 分钟的异常数目超过阈值之后会进行熔断。注意由于统计时间窗口是分钟级别的，若 `timeWindow` 小于 60s，则结束熔断状态后仍可能再进入熔断状态。

简单来说：

- 平均响应时间 (RT，秒级)：平均响应时间超出阈值且在时间窗口内通过的请求 >= 5，两个条件同时满足后触发降级，窗口期过后关闭断路器。RT 最大 4900（更大的需要通过 `-Dcsp.sentinel.statistic.max.rt=xxx` 才能生效）。
- 异常比例 (秒级)：QPS >= 5 且异常比例（秒级统计）超过阈值时，触发降级；时间窗口结束后，关闭降级。
- 异常数 (分钟级)：异常数（分钟统计）超过阈值时，触发降级；时间窗口结束后，关闭降级。

### Sentinel的断路器没有半开状态

半开的状态是系统自动去检测是否请求异常，没有异常就关闭断路器恢复使用，有异常则继续打开断路器。具体可以参考 Hystrix。

![](/img-post/2020-08-04-springcloudalibaba/06-04.jpg)

Sentinel 熔断降级会在调用链路中某个资源出现不稳定状态时（例如调用超时或异常比例升高），对这个资源进行限制，让请求快速失败，避免影响到其他的资源而导致级联错误。

当资源被降级后，在接下来的降级时间窗口之内，对该资源的调用都自动熔断（默认行为是抛出 DegradeException）。

## Sentinel降级-RT

### 概述

RT —— Response Time

平均响应时间 (`DEGRADE_GRADE_RT`)：当 1s 内持续进入 N 个请求，对应时刻的平均响应时间（秒级）均超过阈值（`count`，以 ms 为单位），那么在接下的时间窗口（`DegradeRule` 中的 `timeWindow`，以 s 为单位）之内，对这个方法的调用都会自动地熔断（抛出 `DegradeException`）。注意 Sentinel 默认统计的 RT 上限是 4900 ms，**超出此阈值的都会算作 4900 ms**，若需要变更此上限可以通过启动配置项 `-Dcsp.sentinel.statistic.max.rt=xxx` 来配置。

![](/img-post/2020-08-04-springcloudalibaba/06-05.jpg)

### 测试

#### 代码

FlowLimitController

```java
package demo.yangxu.springcloud.alibaba.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.TimeUnit;


@RestController
@Slf4j
public class FlowLimitController {

    @GetMapping("/testA")
    public String testA()
    {
        return "------testA";
    }

    @GetMapping("/testB")
    public String testB()
    {
        log.info(Thread.currentThread().getName()+"\t"+"...testB");
        return "------testB";
    }

    @GetMapping("/testD")
    public String testD()
    {
        //暂停1秒钟线程 
        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
        log.info("testD 测试RT");
        return "------testD";
    }
}

```

#### 配置

![](/img-post/2020-08-04-springcloudalibaba/06-08.png)

配置说明：

要求 200 毫秒以内处理完本次任务。如果超过 200 毫秒还没处理完，在未来 1 秒钟的时间窗口内，断路器打开（保险丝跳闸），微服务不可用。1 秒过后，如果 200 毫秒以内可以处理完本次任务，微服务恢复。

#### JMeter 压测

![](/img-post/2020-08-04-springcloudalibaba/06-09.png)

![](/img-post/2020-08-04-springcloudalibaba/06-10.png)

配置说明：

1 秒钟内有 10 个线程调用 testD，满足大于 5 个请求的条件。

#### 结果

访问

`http://localhost:8401/testD`

返回结果

`Blocked by Sentinel (flow limiting)`

关闭 JMeter 压测 1 秒后恢复正常。

## Sentinel降级-异常比例

### 概述

异常比例 (DEGRADE_GRADE_EXCEPTION_RATIO)：当资源的每秒请求量 >= N（可配置），并且每秒异常总数占通过量的比值超过阈值（DegradeRule 中的 count）之后，资源进入降级状态，即在接下的时间窗口（DegradeRule 中的 timeWindow，以 s 为单位）之内，对这个方法的调用都会自动地返回。异常比率的阈值范围是 [0.0, 1.0]，代表 0% - 100%。

![](/img-post/2020-08-04-springcloudalibaba/06-06.jpg)

### 测试

#### 代码

FlowLimitController

```java
package demo.yangxu.springcloud.alibaba.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.TimeUnit;


@RestController
@Slf4j
public class FlowLimitController {

    @GetMapping("/testA")
    public String testA()
    {
        return "------testA";
    }

    @GetMapping("/testB")
    public String testB()
    {
        log.info(Thread.currentThread().getName()+"\t"+"...testB");
        return "------testB";
    }

    @GetMapping("/testD")
    public String testD()
    {
        //暂停1秒钟线程
//        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
//        log.info("testD 测试RT");

        log.info("testD 异常比例");
        int age = 10/0;
        return "------testD";
    }
}

```

#### 配置

![](/img-post/2020-08-04-springcloudalibaba/06-11.png)

配置说明：

要求异常比例为 20%，异常比例超过 20% 之后，在未来 3 秒钟的时间窗口内，断路器打开（保险丝跳闸），微服务不可用。3 秒过后，如果异常比例低于 20%，微服务恢复。

#### JMeter 压测

![](/img-post/2020-08-04-springcloudalibaba/06-12.png)

配置说明：

1 秒钟内有 10 个线程调用 testD，满足大于 5 个请求的条件。

#### 结果

访问

`http://localhost:8401/testD`

返回结果

`Blocked by Sentinel (flow limiting)`

关闭 JMeter 压测 3 秒后单独访问，会报 `java.lang.ArithmeticException: / by zero` 异常。因为资源的每秒请求量为 1，不满足大于等于 5 的条件，所以 Sentinel 不会进行降级处理。

## Sentinel降级-异常数

### 概述

异常数 (`DEGRADE_GRADE_EXCEPTION_COUNT`)：当资源近 1 分钟的异常数目超过阈值之后会进行熔断。注意由于统计时间窗口是分钟级别的，若 `timeWindow` 小于 60s，则结束熔断状态后仍可能再进入熔断状态。

即时间窗口一定要大于等于 60 秒。

![](/img-post/2020-08-04-springcloudalibaba/06-07.jpg)

### 测试

#### 代码

修改 FlowLimitController，添加以下代码：

```java
@GetMapping("/testE")
public String testE()
{
    log.info("testE 测试异常数");
    int age = 10/0;
    return "------testE 测试异常数";
}
```



#### 配置

![](/img-post/2020-08-04-springcloudalibaba/06-13.png)

#### 结果

访问

`http://localhost:8401/testE`

异常数没有超过 5 的时候返回 `Whitelabel Error Page`，当异常数（分钟统计）超过 5 之后，触发降级，返回 `Blocked by Sentinel (flow limiting)` 。
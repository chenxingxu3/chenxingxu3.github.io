---
layout:     post
title:      07.Spring Cloud Alibaba学习笔记
subtitle:   Sentinel热点key限流及系统规则
date:       2020-08-04
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloudAlibaba
---
# 07.Spring Cloud Alibaba学习笔记--Sentinel热点key限流及系统规则

## 热点key限流

### 概述

何为热点？热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：

- 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
- 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制

热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。

![Sentinel Parameter Flow Control](/img-post/2020-08-04-springcloudalibaba/07-01.png)

Sentinel 利用 LRU 策略统计最近最常访问的热点参数，结合令牌桶算法来进行参数级别的流控。热点参数限流支持集群模式。

参考：

[https://github.com/alibaba/Sentinel/wiki/%E7%83%AD%E7%82%B9%E5%8F%82%E6%95%B0%E9%99%90%E6%B5%81](https://github.com/alibaba/Sentinel/wiki/热点参数限流)

### 服务降级

**分为系统默认和客户自定义两种**，之前的例子在限流出问题后，都是用 Sentinel 系统默认的提示：`Blocked by Sentinel (flow limiting)`

服务降级也可以自定义。类似 Hystrix，某个方法出问题了，就找对应的兜底降级方法。在 Hystrix 中使用 HystrixCommand，在 Sentinel 中使用 @SentinelResource。

### 源码参考

com.alibaba.csp.sentinel.slots.block.BlockException

```java
/*
 * Copyright 1999-2018 Alibaba Group Holding Ltd.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.alibaba.csp.sentinel.slots.block;

/**
 * Abstract exception indicating blocked by Sentinel due to flow control,
 * circuit breaking or system protection triggered.
 *
 * @author youji.zj
 */
public abstract class BlockException extends Exception {

    public static final String BLOCK_EXCEPTION_FLAG = "SentinelBlockException";
    /**
     * <p>this constant RuntimeException has no stack trace, just has a message
     * {@link #BLOCK_EXCEPTION_FLAG} that marks its name.
     * </p>
     * <p>
     * Use {@link #isBlockException(Throwable)} to check whether one Exception
     * Sentinel Blocked Exception.
     * </p>
     */
    public static RuntimeException THROW_OUT_EXCEPTION = new RuntimeException(BLOCK_EXCEPTION_FLAG);

    public static StackTraceElement[] sentinelStackTrace = new StackTraceElement[] {
        new StackTraceElement(BlockException.class.getName(), "block", "BlockException", 0)
    };

    static {
        THROW_OUT_EXCEPTION.setStackTrace(sentinelStackTrace);
    }

    protected AbstractRule rule;
    private String ruleLimitApp;

    public BlockException(String ruleLimitApp) {
        super();
        this.ruleLimitApp = ruleLimitApp;
    }

    public BlockException(String ruleLimitApp, AbstractRule rule) {
        super();
        this.ruleLimitApp = ruleLimitApp;
        this.rule = rule;
    }

    public BlockException(String message, Throwable cause) {
        super(message, cause);
    }

    public BlockException(String ruleLimitApp, String message) {
        super(message);
        this.ruleLimitApp = ruleLimitApp;
    }

    public BlockException(String ruleLimitApp, String message, AbstractRule rule) {
        super(message);
        this.ruleLimitApp = ruleLimitApp;
        this.rule = rule;
    }

    @Override
    public Throwable fillInStackTrace() {
        return this;
    }

    public String getRuleLimitApp() {
        return ruleLimitApp;
    }

    public void setRuleLimitApp(String ruleLimitApp) {
        this.ruleLimitApp = ruleLimitApp;
    }

    /**
     * Check whether the exception is sentinel blocked exception. One exception is sentinel blocked
     * exception only when:
     * <ul>
     * <li>the exception or its (sub-)cause is {@link BlockException}, or</li>
     * <li>the exception's message is or any of its sub-cause's message equals to {@link #BLOCK_EXCEPTION_FLAG}</li>
     * </ul>
     *
     * @param t the exception.
     * @return return true if the exception marks sentinel blocked exception.
     */
    public static boolean isBlockException(Throwable t) {
        if (null == t) {
            return false;
        }

        int counter = 0;
        Throwable cause = t;
        while (cause != null && counter++ < 50) {
            if ((cause instanceof BlockException) || (BLOCK_EXCEPTION_FLAG.equals(cause.getMessage()))) {
                return true;
            }
            cause = cause.getCause();
        }

        return false;
    }

    public AbstractRule getRule() {
        return rule;
    }
}
```

参考：

[https://github.com/alibaba/Sentinel/blob/master/sentinel-core/src/main/java/com/alibaba/csp/sentinel/slots/block/BlockException.java](https://github.com/alibaba/Sentinel/blob/master/sentinel-core/src/main/java/com/alibaba/csp/sentinel/slots/block/BlockException.java)

### 测试 1

#### 代码

FlowLimitController

添加以下内容：

```java
@GetMapping("/testHotKey")
@SentinelResource(value = "testHotKey",blockHandler = "deal_testHotKey")
public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                         @RequestParam(value = "p2",required = false) String p2) {

    return "------testHotKey";
}

public String deal_testHotKey (String p1, String p2, BlockException exception) {
    return "------deal_testHotKey,o(╥﹏╥)o";  
}
```

#### 配置

![](/img-post/2020-08-04-springcloudalibaba/07-02.png)

#### 结果

快速刷新

`http://localhost:8401/testHotKey?p1=a`

返回结果

`------deal_testHotKey,o(╥﹏╥)o`

方法 testHotKey 里面第一个参数只要 QPS 超过每秒 1 次，马上降级处理。

### 测试 2

#### 代码

FlowLimitController

`@SentinelResource` 中去掉 `blockHandler = "deal_testHotKey"`：

```java
@GetMapping("/testHotKey")
@SentinelResource(value = "testHotKey")
public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                         @RequestParam(value = "p2",required = false) String p2) {

    return "------testHotKey";
}

public String deal_testHotKey (String p1, String p2, BlockException exception) {
    return "------deal_testHotKey,o(╥﹏╥)o";  
}
```

#### 配置

![](/img-post/2020-08-04-springcloudalibaba/07-02.png)

#### 结果

快速刷新

`http://localhost:8401/testHotKey?p1=a`

返回错误页面

`Whitelabel Error Page`

异常显示在前台的用户界面，不友好。

### 测试 3

#### 代码

FlowLimitController

```java
@GetMapping("/testHotKey")
@SentinelResource(value = "testHotKey",blockHandler = "deal_testHotKey")
public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                         @RequestParam(value = "p2",required = false) String p2) {

    return "------testHotKey";
}

public String deal_testHotKey (String p1, String p2, BlockException exception) {
    return "------deal_testHotKey,o(╥﹏╥)o";
}
```

#### 配置

![](/img-post/2020-08-04-springcloudalibaba/07-02.png)

#### 结果

快速刷新

`http://localhost:8401/testHotKey?p1=a`

返回结果

`------deal_testHotKey,o(╥﹏╥)o`

快速刷新

`http://localhost:8401/testHotKey?p1=a&p2=b`

返回结果

`------deal_testHotKey,o(╥﹏╥)o`

快速刷新

`http://localhost:8401/testHotKey?p2=b`

服务降级没有生效。

### 参数例外项

一般情况下，对第一个参数 p1 设置了热点 key 限流，当 QPS 超过 1 秒 1 次的点击后，马上被限流。

如果期望 p1 参数为某个特殊值时，它的限流值和平时不一样，例如当 p1 的值等于 5 时，它的阈值可以达到 200（特例）。

### 测试 4

#### 代码

FlowLimitController

```java
@GetMapping("/testHotKey")
@SentinelResource(value = "testHotKey",blockHandler = "deal_testHotKey")
public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                         @RequestParam(value = "p2",required = false) String p2) {

    return "------testHotKey";
}

public String deal_testHotKey (String p1, String p2, BlockException exception) {
    return "------deal_testHotKey,o(╥﹏╥)o";
}
```

#### 配置

热点参数注意点：参数必须是基本类型或者 String。

要记得点 `添加` 按钮。

![](/img-post/2020-08-04-springcloudalibaba/07-03.png)

![](/img-post/2020-08-04-springcloudalibaba/07-04.png)

#### 结果

快速刷新

`http://localhost:8401/testHotKey?p1=a`

返回结果

`------deal_testHotKey,o(╥﹏╥)o`

当 p1 不等于 5 时，阈值变为平常的 1。

快速刷新

`http://localhost:8401/testHotKey?p1=5`

返回结果

`------testHotKey`

当 p1 等于 5 时，阈值变为 200。

### 测试 5

测试 testHotKey 中存在异常的情况。

#### 代码

FlowLimitController

```java
@GetMapping("/testHotKey")
@SentinelResource(value = "testHotKey",blockHandler = "deal_testHotKey")
public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                         @RequestParam(value = "p2",required = false) String p2) {
    int age = 10/0;
    return "------testHotKey";
}

public String deal_testHotKey (String p1, String p2, BlockException exception) {
    return "------deal_testHotKey,o(╥﹏╥)o";
}
```

#### 配置

![](/img-post/2020-08-04-springcloudalibaba/07-03.png)

![](/img-post/2020-08-04-springcloudalibaba/07-04.png)

#### 结果

访问

`http://localhost:8401/testHotKey?p1=5`

返回错误页面

`Whitelabel Error Page`

**@SentinelResource**

处理的是 Sentinel 控制台配置的违规情况，有 blockHandler 方法配置的兜底处理。

**RuntimeException** 

int age = 10/0，这个是 Java 运行时报出的运行时异常 RunTimeException，@SentinelResource 不管。

@SentinelResource 管理的是配置出错，运行出错该走异常走异常。

## 系统规则

### 概述

Sentinel 系统自适应限流从整体维度对应用入口流量进行控制，结合应用的 Load、CPU 使用率、总体平均 RT、入口 QPS 和并发线程数等几个维度的监控指标，通过自适应的流控策略，让系统的入口流量和系统的负载达到一个平衡，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

系统保护规则是从应用级别的入口流量进行控制，从单台机器的 load、CPU 使用率、平均 RT、入口 QPS 和并发线程数等几个维度监控应用指标，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

系统保护规则是应用整体维度的，而不是资源维度的，并且**仅对入口流量生效**。入口流量指的是进入应用的流量（`EntryType.IN`），比如 Web 服务或 Dubbo 服务端接收的请求，都属于入口流量。

系统规则支持以下的模式：

- **Load 自适应**（仅对 Linux/Unix-like 机器生效）：系统的 load1 作为启发指标，进行自适应系统保护。当系统 load1 超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR 阶段）。系统容量由系统的 `maxQps * minRt` 估算得出。设定参考值一般是 `CPU cores * 2.5`。
- **CPU usage**（1.5.0+ 版本）：当系统 CPU 使用率超过阈值即触发系统保护（取值范围 0.0-1.0），比较灵敏。
- **平均 RT**：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒。
- **并发线程数**：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护。
- **入口 QPS**：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护。

参考：

[https://github.com/alibaba/Sentinel/wiki/%E7%B3%BB%E7%BB%9F%E8%87%AA%E9%80%82%E5%BA%94%E9%99%90%E6%B5%81](https://github.com/alibaba/Sentinel/wiki/系统自适应限流)

### 测试

演示系统规则的入口 QPS 模式的使用方法。

#### 代码

FlowLimitController

```java
package demo.yangxu.springcloud.alibaba.controller;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
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
    public String testB() {
        log.info(Thread.currentThread().getName()+"\t"+"...testB");
        return "------testB";
    }

    @GetMapping("/testD")
    public String testD() {
        //暂停1秒钟线程
//        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
//        log.info("testD 测试RT");

//        log.info("testD 异常比例");
//        int age = 10/0;
        return "------testD";
    }

    @GetMapping("/testE")
    public String testE() {
        log.info("testE 测试异常数");
        //int age = 10/0;
        return "------testE 测试异常数";
    }

    @GetMapping("/testHotKey")
    @SentinelResource(value = "testHotKey",blockHandler = "deal_testHotKey")
    public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                             @RequestParam(value = "p2",required = false) String p2) {
        //int age = 10/0;
        return "------testHotKey";
    }

    public String deal_testHotKey (String p1, String p2, BlockException exception) {
        return "------deal_testHotKey,o(╥﹏╥)o";
    }
}

```

#### 配置

![](/img-post/2020-08-04-springcloudalibaba/07-05.png)



#### 结果

访问

FlowLimitController 中所有的 @GetMapping，均返回

`Blocked by Sentinel (flow limiting)`
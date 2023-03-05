---
layout:     post
title:      05.Spring Cloud Alibaba学习笔记
subtitle:   Sentinel流控
date:       2020-08-04
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloudAlibaba
---
# 05.Spring Cloud Alibaba学习笔记--Sentinel流控

## Sentinel流控规则简介

流控规则指的是流量限制控制规则。

![](/img-post/2020-08-04-springcloudalibaba/04-05.png)

- 资源名：唯一名称，默认请求路径
- 针对来源：Sentinel 可以针对调用者进行限流，填写微服务名，默认 default （不区分来源）
- 阈值类型 / 单机阈值：
  - QPS（每秒钟的请求数量）：当调用该 api 的 QPS 达到阈值的时候，进行限流
  - 线程数：当调用该 api 的线程数达到阈值的时候，进行限流
- 是否集群：不需要集群
- 流控模式：
  - 直接：api 达到限流条件时，直接限流
  - 关联：当关联的资源达到阈值时，限流自己
  - 链路：只记录指定链路上的流量（指定资源从入口资源进来的流量，如果达到阈值，就进行限流）【api 级别的针对来源】
- 流控效果：
  - 快速失败：直接失败，抛异常
  - Warm Up：根据 codeFactor （冷加载因子，默认为 3）的值，从阈值 / codeFactor，经过预热时长，才达到设置的 QPS 阈值
  - 排队等待：匀速排队，让请求以匀速的速度通过，阈值类型必须设置为 QPS，否则无效

## Sentinel流控-QPS直接失败

### 流控模式

#### 直接（默认）

**系统默认：直接 -> 快速失败**

添加方式一：

![](/img-post/2020-08-04-springcloudalibaba/04-06.png)

添加方式二：

![](/img-post/2020-08-04-springcloudalibaba/04-07.png)

**配置说明**:

1 秒钟内查询 1 次就是 OK，若超过次数 1，就直接快速失败，报默认错误。

**测试:**

快速点击访问`http://localhost:8401/testA`

结果：

`Blocked by Sentinel (flow limiting)`

直接调用默认报错信息。

## Sentinel流控-线程数直接失败

修改流控规则：

![](/img-post/2020-08-04-springcloudalibaba/04-08.png)

修改 FlowLimitController

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
        //============新增begin==========
        try { TimeUnit.MILLISECONDS.sleep(2000); } catch (InterruptedException e) { e.printStackTrace(); }
        //============新增end==========
        return "------testA";
    }

    @GetMapping("/testB")
    public String testB()
    {
        log.info(Thread.currentThread().getName()+"\t"+"...testB");
        return "------testB";
    }
}

```

测试：

在浏览器中开启多个标签页访问`http://localhost:8401/testA`

返回结果：

`Blocked by Sentinel (flow limiting)`

## Sentinel流控-关联

### 流控模式

#### 关联

**当关联的资源达到阈值时，就限流自己**。

比如当与 A 关联的资源 B 达到阈值后，就限流 A 自己（B 惹事，A 挂了）。

**应用场景: 比如支付接口达到阈值,就要限流下订单的接口,防止一直有订单**

**准备工作**

FlowLimitController

```java
package demo.yangxu.springcloud.alibaba.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;


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
}

```

**配置流控规则**

![](/img-post/2020-08-04-springcloudalibaba/04-09.png)

配置说明：

当关联资源 /testB 的 QPS 阀值超过 1 时，就限流 /testA 的 Rest 访问地址，**即当关联资源达到阈值后限制配置好的资源名。**

**使用 Postman 模拟并发密集访问 testB**

1、在 Postman 中测试访问`http://localhost:8401/testB`

![](/img-post/2020-08-04-springcloudalibaba/04-10.png)

2、保存到 Collections 中的某个 Collection 中

![](/img-post/2020-08-04-springcloudalibaba/04-11.png)

3、配置相关参数，运行测试

![](/img-post/2020-08-04-springcloudalibaba/04-12.png)

4、访问 `http://localhost:8401/testA`

结果：

`Blocked by Sentinel (flow limiting)`

![](common/04-13.png)

## Sentinel流控-链路

### 流控模式

#### 链路

多个请求调用了同一个微服务，根据调用链路入口限流。

`NodeSelectorSlot` 中记录了资源之间的调用链路，这些资源通过调用关系，相互之间构成一棵调用树。这棵树的根节点是一个名字为 `machine-root` 的虚拟节点，调用链的入口都是这个虚节点的子节点。

一棵典型的调用树如下图所示：

```
     	          machine-root
                    /       \
                   /         \
             Entrance1     Entrance2
                /             \
               /               \
      DefaultNode(nodeA)   DefaultNode(nodeA)
```

上图中来自入口 `Entrance1` 和 `Entrance2` 的请求都调用到了资源 `NodeA`，Sentinel 允许只根据某个入口的统计信息对资源限流。比如我们可以设置 `strategy` 为 `RuleConstant.STRATEGY_CHAIN`，同时设置 `refResource` 为 `Entrance1` 来表示只有从入口 `Entrance1` 的调用才会记录到 `NodeA` 的限流统计当中，而不关心经 `Entrance2` 到来的调用。

调用链的入口（上下文）是通过 API 方法 `ContextUtil.enter(contextName)` 定义的，其中 contextName 即对应调用链路入口名称。详情可以参考 [ContextUtil 文档](https://github.com/alibaba/Sentinel/wiki/如何使用#上下文工具类-contextutil)。

参考：

[https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6](https://github.com/alibaba/Sentinel/wiki/流量控制)

**配置思路**

首先查看 cloudalibaba-sentinel-service 的簇点链路：

![](common/04-15.png)

调用树如下图所示：

```
     	          machine-root
                    /        \
                   /          \
              /testA         /testB
                /               \
               /                 \
sentinel_web_servlet_context   sentinel_web_servlet_context
```

根据上图的调用树，可以参考的一个配置案例：

设置 `入口资源` 为 `/testA` 来表示只有从入口 `/testA` 的调用才会记录到 `sentinel_web_servlet_context` 的限流统计当中，而不关心经 `/testB` 到来的调用。

**具体配置案例**

为 `sentinel_web_servlet_context` 新增流控规则

![](common/04-16.png)

![](common/04-17.png)

**测试**

同时快速刷新

- http://localhost:8401/testA
- http://localhost:8401/testB

`/testA` 返回 `Blocked by Sentinel (flow limiting)`，而 `/testB` 访问正常。

![](common/04-14.png)

## Sentinel流控-预热

### 流控效果

#### 直接 -> 快速失败（默认的流控处理）

直接失败，抛出异常：`Blocked by Sentinel (flow limiting)`

具体的例子参见 [FlowQpsDemo](https://github.com/alibaba/Sentinel/blob/master/sentinel-demo/sentinel-demo-basic/src/main/java/com/alibaba/csp/sentinel/demo/flow/FlowQpsDemo.java)

#### 预热（Warm Up）

##### 概述

公式：阈值除以 coldFactor (默认值为 3)，经过预热时长后才会达到阈值。

即请求 QPS 从 threshold / 3 开始，经预热时长逐渐升至设定的 QPS 阈值。

当流量突然增大的时候，我们常常会希望系统从空闲状态到繁忙状态的切换的时间长一些。即如果系统在此之前长期处于空闲的状态，我们希望处理请求的数量是缓步的增多，经过预期的时间以后，到达系统处理请求个数的最大值。

例如，秒杀系统在开启的瞬间，会有很多流量上来，很有可能把系统打死。

Warm Up（冷启动，预热）模式就是为了实现这个目的。

预热方式就是把为了保护系统，慢慢地把流量放进来，慢慢地把阈值增长到设置的阈值。

预热方式也可用于启动需要额外开销的场景，例如建立数据库连接等。

它的实现是在 [Guava](https://github.com/google/guava/blob/master/guava/src/com/google/common/util/concurrent/SmoothRateLimiter.java) 的算法的基础上实现的。然而，和 Guava 的场景不同，Guava 的场景主要用于调节请求的间隔，即 [Leaky Bucket](https://en.wikipedia.org/wiki/Leaky_bucket)，而 Sentinel 则主要用于控制每秒的 QPS，即我们满足每秒通过的 QPS 即可，我们不需要关注每个请求的间隔，换言之，我们更像一个 [Token Bucket](https://en.wikipedia.org/wiki/Token_bucket)。

我们用桶里剩余的令牌来量化系统的使用率。假设系统每秒的处理能力为 b,系统每处理一个请求，就从桶中取走一个令牌；每秒这个令牌桶会自动掉落b个令牌。令牌桶越满，则说明系统的利用率越低；当令牌桶里的令牌高于某个阈值之后，我们称之为令牌桶"饱和"。

当令牌桶饱和的时候，基于 Guava 的计算上，我们可以推出下面两个公式:

```
rate(c)=m*c+ coldrate
```

其中，rate 为当前请求和上一个请求的间隔时间，而 rate 是和令牌桶中的高于阈值的令牌数量成线形关系的。cold rate 则为当桶满的时候，请求和请求的最大间隔。通常是 `coldFactor * rate(stable)`。

通常冷启动的过程系统允许通过的 QPS 曲线如下图所示：

![冷启动过程 QPS 曲线](common/04-18.gif)

默认 `coldFactor` 为 3，即请求 QPS 从 `threshold / 3` 开始，经预热时长逐渐升至设定的 QPS 阈值。

参考：

[https://github.com/alibaba/Sentinel/wiki/%E9%99%90%E6%B5%81---%E5%86%B7%E5%90%AF%E5%8A%A8](https://github.com/alibaba/Sentinel/wiki/限流---冷启动)

##### 源码

```java
public WarmUpController(double count, int warmUpPeriodInSec) {
    construct(count, warmUpPeriodInSec, 3);
}
```

参考：

https://github.com/alibaba/Sentinel/blob/master/sentinel-core/src/main/java/com/alibaba/csp/sentinel/slots/block/flow/controller/WarmUpController.java

##### 案例

阈值为 10+ 预热时长设置为 5 秒。

系统初始化的阈值为 10 / 3 约等于 3，即阈值刚开始为 3；然后过了 5 秒后阈值才慢慢升高恢复到 10。

![](common/04-19.png)

##### 测试

快速刷新：

http://localhost:8401/testB

最初的 5 秒系统会频繁提示 `Blocked by Sentinel (flow limiting)`，5 秒之后系统提示 `Blocked by Sentinel (flow limiting)`的频率大幅降低了。

## Sentinel流控-排队等待

### 流控效果

#### 排队等待

##### 概述

匀速排队，让请求以均匀的速度通过，阈值类型必须设成 QPS，否则无效。

匀速排队（`RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER`）方式会严格控制请求通过的间隔时间，也即是让请求以均匀的速度通过，对应的是漏桶算法。详细文档可以参考 [流量控制 - 匀速器模式](https://github.com/alibaba/Sentinel/wiki/流量控制-匀速排队模式)，具体的例子可以参见 [PaceFlowDemo](https://github.com/alibaba/Sentinel/blob/master/sentinel-demo/sentinel-demo-basic/src/main/java/com/alibaba/csp/sentinel/demo/flow/PaceFlowDemo.java)。

该方式的作用如下图所示：

![image](common/04-20.png)

这种方式主要用于处理间隔性突发的流量，例如消息队列。想象一下这样的场景，在某一秒有大量的请求到来，而接下来的几秒则处于空闲状态，我们希望系统能够在接下来的空闲期间逐渐处理这些请求，而不是在第一秒直接拒绝多余的请求。

> 注意：匀速排队模式暂时不支持 QPS > 1000 的场景。

参考：

[https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6](https://github.com/alibaba/Sentinel/wiki/流量控制)

##### 源码

https://github.com/alibaba/Sentinel/blob/master/sentinel-core/src/main/java/com/alibaba/csp/sentinel/slots/block/flow/controller/RateLimiterController.java

##### 修改 FlowLimitController

```java
package demo.yangxu.springcloud.alibaba.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;


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
}

```

##### 案例

![](common/04-21.png)

设置含义：

/testB 每秒 1 次请求，超过的话就排队等待，等待的超时时间为 20000 毫秒。

##### 测试

1、配置 Postman 并运行测试

![](common/04-22.png)

2、通过 IDEA 控制台输出的信息可以看到效果

```
2020-08-08 17:28:12.488  INFO 8876 --- [nio-8401-exec-3] d.y.s.a.controller.FlowLimitController   : http-nio-8401-exec-3	...testB
2020-08-08 17:28:13.590  INFO 8876 --- [nio-8401-exec-4] d.y.s.a.controller.FlowLimitController   : http-nio-8401-exec-4	...testB
2020-08-08 17:28:14.640  INFO 8876 --- [nio-8401-exec-5] d.y.s.a.controller.FlowLimitController   : http-nio-8401-exec-5	...testB
2020-08-08 17:28:15.680  INFO 8876 --- [nio-8401-exec-6] d.y.s.a.controller.FlowLimitController   : http-nio-8401-exec-6	...testB
2020-08-08 17:28:16.735  INFO 8876 --- [nio-8401-exec-7] d.y.s.a.controller.FlowLimitController   : http-nio-8401-exec-7	...testB
2020-08-08 17:28:17.832  INFO 8876 --- [nio-8401-exec-8] d.y.s.a.controller.FlowLimitController   : http-nio-8401-exec-8	...testB
2020-08-08 17:28:18.871  INFO 8876 --- [nio-8401-exec-9] d.y.s.a.controller.FlowLimitController   : http-nio-8401-exec-9	...testB
2020-08-08 17:28:19.918  INFO 8876 --- [io-8401-exec-10] d.y.s.a.controller.FlowLimitController   : http-nio-8401-exec-10	...testB
2020-08-08 17:28:20.960  INFO 8876 --- [nio-8401-exec-1] d.y.s.a.controller.FlowLimitController   : http-nio-8401-exec-1	...testB
2020-08-08 17:28:21.998  INFO 8876 --- [nio-8401-exec-2] d.y.s.a.controller.FlowLimitController   : http-nio-8401-exec-2	...testB

```

## 
---
layout:     post
title:      15.Spring Cloud Alibaba学习笔记
subtitle:   Seata案例--测试
date:       2020-08-04
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloudAlibaba
---
# 15.Spring Cloud Alibaba学习笔记--Seata案例--测试

## 数据库初始情况

![](/img-post/2020-08-04-springcloudalibaba/15-01.png)

## 正常下单

`http://localhost:2001/order/create?userId=1&productId=1&count=10&money=100`

运行结果

```json
{"code":200,"message":"订单创建成功","data":{"id":null,"userId":1,"productId":1,"count":10,"money":100,"status":null}}
```

数据库情况

![](/img-post/2020-08-04-springcloudalibaba/15-02.png)

## 没加 @GlobalTransactional 出现超时异常

### AccountServiceImpl 添加超时

```java
@Override
public void decrease(Long userId, BigDecimal money) {
    log.info("------->account-service中扣减账户余额开始");
    //模拟超时异常，全局事务回滚
    //暂停几秒钟线程
    try { TimeUnit.SECONDS.sleep(20); } catch (InterruptedException e) { e.printStackTrace(); }
    accountDao.decrease(userId, money);
    log.info("------->account-service中扣减账户余额结束");
}
```

`http://localhost:2001/order/create?userId=1&productId=1&count=10&money=100`

运行错误

`There was an unexpected error (type=Internal Server Error, status=500).
Read timed out executing POST http://seata-account-service/account/decrease?userId=1&money=100`

### 数据库情况

![](/img-post/2020-08-04-springcloudalibaba/15-03.png)

### 故障情况

当库存和账户金额扣减后，订单状态并没有设置为已经完成，没有从零改为 1。

而且由于 feign 的重试机制，账户余额还有可能被多次扣减。

## 添加 @GlobalTransactional 出现超时异常

### AccountServiceImpl 添加超时

```java
@Override
public void decrease(Long userId, BigDecimal money) {
    log.info("------->account-service中扣减账户余额开始");
    //模拟超时异常，全局事务回滚
    //暂停几秒钟线程
    try { TimeUnit.SECONDS.sleep(20); } catch (InterruptedException e) { e.printStackTrace(); }
    accountDao.decrease(userId, money);
    log.info("------->account-service中扣减账户余额结束");
}
```



### 添加 @GlobalTransactional

OrderServiceImpl#create 

```java
@Override
//name只要唯一即可，可以起任意值
//rollbackFor 发生任何异常均回滚
@GlobalTransactional(name = "fsp-create-order",rollbackFor = Exception.class)
public void create(Order order) {
    //1 新建订单
    log.info("----->开始新建订单");
    orderDao.create(order);
    log.info("----->新建订单完成");

    //2 扣减库存
    log.info("----->订单微服务开始调用库存，做扣减Count");
    storageService.decrease(order.getProductId(), order.getCount());
    log.info("----->库存扣减Count完成");

    //3 扣减账户
    log.info("----->订单微服务开始调用账户，做扣减Money");
    accountService.decrease(order.getUserId(), order.getMoney());
    log.info("----->账户扣减Money完成");

    //4 修改订单状态，从0到1,1代表已经完成
    log.info("----->修改订单状态开始");
    orderDao.update(order.getUserId(),0);
    log.info("----->修改订单状态结束");

    log.info("----->下订单结束了，O(∩_∩)O哈哈~");

}
```

### 恢复数据库到初始状态

![](/img-post/2020-08-04-springcloudalibaba/15-04.png)

### 访问

`http://localhost:2001/order/create?userId=1&productId=1&count=10&money=100`

结果异常

`There was an unexpected error (type=Internal Server Error, status=500).`

### 数据库情况

![](/img-post/2020-08-04-springcloudalibaba/15-05.png)
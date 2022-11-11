---
layout:     post
title:      Oracle数据库学习笔记（十二）
subtitle:   多行函数（聚合函数）
date:       2020-05-21
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（十二）--多行函数（聚合函数）

# 概念

作用于多行，返回一个值。

# 学习笔记

## 1、查询总数量

```sql
SELECT COUNT(1)
FROM emp;
```

COUNT(1) 和 COUNT(*) 底层都是一样的，推荐使用 COUNT(1)。当然，填写某个具体列名，比如 COUNT(EMPNO) 也是可以的。

## 2、查询工资总和

```sql
SELECT SUM(sal)
FROM emp;
```

## 3、查询最大工资

```sql
SELECT MAX(sal)
FROM emp;
```

## 4、查询最低工资

```sql
SELECT MIN(sal)
FROM emp;
```

## 5、查询平均工资

```sql
SELECT AVG(sal)
FROM emp;
```


---
layout:     post
title:      Oracle数据库学习笔记（十九）
subtitle:   索引
date:       2020-05-22
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（十九）--索引

# 索引的概念

索引就是在表的列上构建一棵二叉树，达到大幅度提高查询效率的目的，但是索引会影响增删改的效率。

比如说，我们将表看作是书，一开始这些书都是没有目录的，如果要查找某一段文字，需要一页一页的去翻，查询效率特别低。为了提高查找的效率，我们就在每本书上添加一个目录，而这个目录是用二叉树形式存储的，这样查询的效率就会大幅度提高。书中的目录就是表中的索引。但是索引会影响增删改的效率，因为每次执行增、删、改的操作后，都会重新去构建一棵索引的二叉树。

# 单例索引

单列索引是基于单个列所建立的索引。

**1、创建单列索引**

```sql
CREATE INDEX idx_ename ON emp(ename);
```

**2、单列索引触发规则**

条件必须是索引列中的原始值。

```sql
SELECT *
FROM emp
WHERE ename = 'SCOTT'
```

诸如单行函数 [ 比如 upper() 函数 ]、模糊查询都会影响索引的触发。

# 复合索引

复合索引是基于两个列或多个列的索引。在同一张表上可以有多个索引，但是要求列的组合必须不同。

**1、创建复合索引**

```sql
CREATE INDEX idx_enamejob ON emp(ename, job);
```

**2、复合索引触发规则**

复合索引中第一列为优先检索列，如果要触发复合索引，必须包含有优先检索列中的原始值。

触发复合索引

```sql
SELECT *
FROM emp
WHERE ename = 'SCOTT'
	AND job = 'xx';
```

不触发复合索引。用 OR 连接可以将查询语句看作是两个条件不同的查询语句，一个触发索引，另一个不触发索引，用 OR 连接后就是不触发复合索引

```sql
SELECT *
FROM emp
WHERE ename = 'SCOTT'
	OR job = 'xx';
```

如果一张表同时拥有单例索引和复合索引，使用下面的语句，触发的是单列索引

```sql
SELECT *
FROM emp
WHERE ename = 'SCOTT';
```


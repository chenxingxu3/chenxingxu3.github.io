---
layout:     post
title:      Oracle数据库学习笔记（十四）
subtitle:   多表查询中的一些概念
date:       2020-05-22
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（十四）--多表查询中的一些概念

**1、笛卡尔积**

通俗点说就是就是两张表的数据进行相乘，一张表的所有记录一一和另一张表的所有记录做匹配。

```sql
SELECT *
FROM emp e, dept d;
```

当两张表的数据量比较大，又需要连接查询时，应尽量避免笛卡尔积，因为会增加内存的开销，而且使用笛卡尔积产生的数据大多是没有用的。

**2、等值连接**

```sql
SELECT *
FROM emp e, dept d
WHERE e.deptno = d.deptno;
```

**3、内连接**

```sql
SELECT *
FROM emp e
	INNER JOIN dept d ON e.deptno = d.deptno ;
```

内连接的效果和等值连接的效果相同，不过等值连接是后来才出现的。推荐使用等值连接。

**4、查询出所有部门，以及部门下的员工信息。【外连接】**

```sql
SELECT *
FROM emp e
	RIGHT JOIN dept d ON e.deptno = d.deptno ;
```

**5、查询所有员工信息，以及员工所属部门**

```sql
SELECT *
FROM emp e
	LEFT JOIN dept d ON e.deptno = d.deptno ;
```

**6、Oracle 专用外连接**

查询出所有部门，以及部门下的员工信息

```sql
SELECT *
FROM emp e, dept d
WHERE e.deptno(+) = d.deptno;
```

与等值连接相似。要显示谁的全部数据，就在谁的对面添加 (+)。本例显示的是 dept 表的全部数据。
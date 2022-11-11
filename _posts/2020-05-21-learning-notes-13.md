---
layout:     post
title:      Oracle数据库学习笔记（十三）
subtitle:   分组查询
date:       2020-05-21
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（十三）--分组查询

# 学习笔记

## 1、查询出每个部门的平均工资

```sql
SELECT e.deptno, AVG(e.sal)
FROM emp e
GROUP BY e.deptno;
```

分组查询中，出现在 GROUP BY 后面的原始列，才能出现在 SELECT 后面，没有出现在 GROUP BY 后面的列，想在 SELECT 后面，必须加上聚合函数。聚合函数有一个特性，可以把多行记录变成一个值。

## 2、查询出平均工资高于 2000 的部门信息

```sql
SELECT e.deptno, AVG(e.sal) asal
FROM emp e
GROUP BY e.deptno
HAVING AVG(e.sal) > 2000;
```

所有条件都不能使用别名来判断，HAVING 的实行顺序先于 SELECT。比如下面的条件语句也不能使用别名当条件。WHERE 的执行顺序先于 SELECT。

```sql
SELECT ename, sal s
FROM emp
WHERE sal > 1500;
```

## 3、查询出每个部门工资高于 800 的员工的平均工资

```sql
SELECT e.deptno, AVG(e.sal) asal
FROM emp e
WHERE e.sal > 800
GROUP BY e.deptno;
```

WHERE 是过滤分组前的数据，HAVING 是过滤分组后的数据。

表现形式：WHERE 必须在 GROUP BY 之前，HAVING 是在 GROUP BY 之后。

## 4、查询出每个部门工资高于 800 的员工的平均工资，然后再查询出平均工资高于 2000 的部门。

```sql
SELECT e.deptno, AVG(e.sal) asal
FROM emp e
WHERE e.sal > 800
GROUP BY e.deptno
HAVING AVG(e.sal) > 2000;
```


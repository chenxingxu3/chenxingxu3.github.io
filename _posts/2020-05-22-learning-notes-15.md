---
layout:     post
title:      Oracle数据库学习笔记（十五）
subtitle:   自连接
date:       2020-05-22
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（十五）--自连接

自连接其实就是站在不同的角度把一张表看成多张表。

**1、查询出员工姓名，员工领导姓名**

```sql
SELECT e1.ename, e2.ename
FROM emp e1, emp e2
WHERE e1.mgr = e2.empno;
```

此时可将 e1 表看成是员工表，e2 表看成是领导表，因为 e1 的领导是 e2 的员工编号。

**2、查询出员工姓名，员工部门名称，员工领导姓名，员工领导部门名称**

```sql
SELECT e1.ename, d1.dname, e2.ename, d2.dname
FROM emp e1, emp e2, dept d1, dept d2
WHERE e1.mgr = e2.empno
	AND e1.deptno = d1.deptno
	AND e2.deptno = d2.deptno;
```

此时可将 e1 表看成是员工表，e2 表看成是领导表，d1 表看成是员工部门表，d2 表看成是领导部门表。
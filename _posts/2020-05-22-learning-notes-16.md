---
layout:     post
title:      Oracle数据库学习笔记（十六）
subtitle:   子查询
date:       2020-05-22
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（十六）--子查询

# 子查询返回一个值

**查询出工资和 SCOTT 一样的员工信息**

```sql
SELECT *
FROM emp
WHERE sal IN (
	SELECT sal
	FROM emp
	WHERE ename = 'SCOTT'
);
```

分步解析：

**1、先查询出 SCOTT 的工资**

```sql
SELECT sal
FROM emp
WHERE ename = 'SCOTT'
```

**注意：这里的员工姓名是区分大小写的。**

虽然上述语句那么长，但是实际上它就是一个值。

**2、把上面的语句用括号括起来，再查询出工资和 SCOTT 一样的员工信息**

```sql
SELECT *
FROM emp
WHERE sal = (
	SELECT sal
	FROM emp
	WHERE ename = 'SCOTT'
)
```

**注意：在 where sal = 这里写等号是有隐患的。**因为无法保证一个公司名字为 SCOTT 的员工是唯一的，有可能发生重名的现象，这样查询出来的结果就不是一个工资的值了，而是所有名字为 SCOTT 的员工的工资的集合，这样再写等号就会报错。为了避免这种情况发生，建议把等号改为 IN。

**3、更正后最终结果**

```sql
SELECT *
FROM emp
WHERE sal IN (
	SELECT sal
	FROM emp
	WHERE ename = 'SCOTT'
);
```

当然，如果是根据主键查询，返回的结果自然是唯一的，还是可以用等号的。

# 子查询返回一个集合

**查询出工资和 10 号部门任意员工一样的员工信息**

```sql
SELECT *
FROM emp
WHERE sal IN (
	SELECT sal
	FROM emp
	WHERE deptno = 10
);
```

分步解析：

**1、先查出 10 号部门所有人工资的集合**

```sql
SELECT sal
FROM emp
WHERE deptno = 10
```

**2、把上面的语句用括号括起来，再查询出工资和 10 号部门任意员工一样的员工信息**

```sql
SELECT *
FROM emp
WHERE sal IN (
	SELECT sal
	FROM emp
	WHERE deptno = 10
);
```



# 子查询返回一张表

**查询出每个部门最低工资，和最低工资员工姓名，和该员工所在部门名称**

```sql
SELECT t.deptno, t.msal, e.ename, d.dname
FROM (
	SELECT deptno, MIN(sal) AS msal
	FROM emp
	GROUP BY deptno
) t, emp e, dept d
WHERE t.deptno = e.deptno
	AND t.msal = e.sal
	AND e.deptno = d.deptno;
```

分步解析：

当前没有每个部门最低工资这张表，需要自己先查询出来。员工姓名和部门名称都可以根据 emp 表和 dept 表进行查询。

**1、先查询出每个部门最低工资**

```sql
SELECT deptno, MIN(sal) AS msal
FROM emp
GROUP BY deptno;
```

**2、三表联查，得到最终结果**

```sql
SELECT t.deptno, t.msal, e.ename, d.dname
FROM (
	SELECT deptno, MIN(sal) AS msal
	FROM emp
	GROUP BY deptno
) t, emp e, dept d
WHERE t.deptno = e.deptno
	AND t.msal = e.sal
	AND e.deptno = d.deptno;
```

msal 这个别名可以使用，因为会先执行子查询，执行完后 msal 这个别名就生效了，可以在外部的 SELECT 中使用。

t.deptno = e.deptno -- 确保在同一个部门（t 表和 emp 表联合）

t.msal = e.sal -- 通过这个条件得到最低工资员工姓名（t 表和 emp 表联合）

e.deptno = d.deptno -- 通过这个条件得到员工所在部门名称（emp 表和 dept 表联合）

t.deptno -- 部门编号

t.msal -- 最低工资

e.ename -- 员工姓名

d.dname -- 部门名称
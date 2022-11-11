---
layout:     post
title:      Oracle数据库学习笔记（九）
subtitle:   scott用户
date:       2020-05-21
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（九）--scott用户

# SCOTT 用户概述

SCOTT 是在 Oracle 数据库中，一个示例用户的名称，默认口令为 tiger，其作用是为初学者提供一些简单的应用示例。其中的表和表间的关系演示了关系型数据库的一些基本原理，Oracle 举例说明时一般都用这个用户，一些关于 Oracle 的书、教材上一般也都用这个用户来讲解。

# SCOTT 用户下的表结构

![](/img-post/2020-05-21-learning-notes-09/01.jpg)

EMP（雇员表）

| NO   | 字段       | 类型          | 描述               |
| ---- | ---------- | ------------- | ------------------ |
| 1    | EMPNO      | NUMBER(4)     | 雇员编号           |
| 2    | ENAME      | VARCHAR2(10)  | 雇员姓名           |
| 3    | JOB        | VARCHAR2(9)   | 工作职位           |
| 4    | MGR        | NUMBER(4)     | 一个雇员的领导编号 |
| 5    | HIREDATE   | DATE          | 雇佣日期           |
| 6    | SAL        | NUMBER(7,2)   | 月薪，工资         |
| 7    | COMM       | NUMBER(7,2)   | 奖金或佣金         |
| 8    | **DEPTNO** | **NUMBER(2)** | **部门编号**       |

DEPT（部门表）

| NO   | 字段       | 类型          | 描述         |
| ---- | ---------- | ------------- | ------------ |
| 1    | **DEPTNO** | **NUMBER(2)** | **部门编号** |
| 2    | DNAME      | VARCHAR2(14)  | 部门名称     |
| 3    | LOC        | VARCHAR2(13)  | 部门位置     |

BONUS（奖金表）

| NO   | 字段  | 类型         | 描述     |
| ---- | ----- | ------------ | -------- |
| 1    | ENAME | VARCHAR2(10) | 雇员姓名 |
| 2    | JOB   | VARCHAR2(9)  | 雇员工作 |
| 3    | SAL   | NUMBER       | 雇员工资 |
| 4    | COMM  | NUMBER       | 雇员奖金 |

SALGRADE（工资等级表）

| NO   | 字段  | 类型   | 描述             |
| ---- | ----- | ------ | ---------------- |
| 1    | GRADE | NUMBER | 等级名称         |
| 2    | LOSAL | NUMBER | 此等级的最低工资 |
| 3    | HISAL | NUMBER | 此等级的最高工资 |

# 解锁 SCOTT 用户

解锁 SCOTT 用户

```sql
ALTER USER scott account unlock;
```

解锁 SCOTT 用户的密码【此句也可以用来重置密码】

```sql
ALTER USER scott IDENTIFIED BY tiger;
```


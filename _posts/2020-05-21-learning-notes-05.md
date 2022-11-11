---
layout:     post
title:      Oracle数据库学习笔记（五）
subtitle:   表空间创建用户及用户授权
date:       2020-05-21
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---


# 1、使用PLSQL Developer 连接到数据库

system 用户为 DBA 角色，拥有全部特权，是系统最高权限。为了方便学习，先使用 system 用户进行登录。

![](/img-post/2020-05-21-learning-notes-05/01.jpg)

**Normal -- 普通身份，普通程序员选择这个即可**

**SYSDBA -- 系统管理员身份**

SYSDBA 权限：

1. 启动和关闭操作
2. 更改数据库状态为打开/装载/备份，更改字符集
3. 创建数据库
4. 创建服务器参数文件 SPFILE
5. 日志归档和恢复
6. 包含了“会话权限”权限

**SYSOPER -- 系统操作员身份**

SYSOPER权限：

1. 启动和关闭操作
2. 更改数据库状态为打开/装载/备份
3. 创建服务器参数文件 SPFILE
4. 日志归档和恢复
5. 包含了“会话权限”权限

打开一个 SQL Window 或者 Test Window

![](/img-post/2020-05-21-learning-notes-05/02.jpg)

# 2、学习笔记

**1、创建表空间**

```sql
create tablespace tbs_test_1
datafile 'c:\tbs_test_1.dbf'
size 100m
autoextend on
next 10m;
```

tbs_test_1 -- 为表空间名称

datafile -- 指定表空间对应的数据文件。表空间只是一个逻辑单位，真正存放数据的地方应该叫数据文件，创建完表空间后应该指定数据文件所在的位置。注意：这里指定的是 Oracle 所在服务端的目录，而不是当前连接 Oracle 数据库的客户端的目录。

size -- 定义表空间的初始大小

autoextend on -- 当表空间存储都占满时，自动增长

next -- 指定一次自动增长的大小

**2、删除表空间**

```sql
DROP TABLESPACE tbs_test_1;
```

**3、创建用户**

```sql
create user xiaoming
identified by mima
default tablespace tbs_test_1;
```

identified by -- 用户密码

default tablespace -- 表空间名称

Oracle 数据库与其它数据库产品的区别在于，表和其它的数据库对象都是存储在用户下的。

**4、给用户授权**

```sql
grant dba to xiaoming;
```

Oracle 数据库中常用角色：

1. connect -- 连接角色，基本角色
2. resource -- 开发者角色
3. dba -- 超级管理员角色

CONNECT 角色 -- 授予最终用户典型权利的最基本角色，拥有的权限如下：

1. ALTER SESSION -- 修改会话
2. CREATE CLUSTER -- 建立聚簇
3. CREATE DATABASE LINK -- 建立数据库连接
4. CREATE SEQUENCE -- 建立序列
5. CREATE SESSION -- 建立会话
6. CREATE SYNONYM -- 建立同义词
7. CREATE VIEW -- 建立视图

RESOURCE 角色 -- 授予开发人员的角色，拥有的权限如下：

1. CREATE CLUSTER -- 建立聚簇
2. CREATE PROCEDURE -- 建立过程
3. CREATE SEQUENCE -- 建立序列
4. CREATE TABLE -- 建表
5. CREATE TRIGGER -- 建立触发器
6. CREATE TYPE -- 建立类型

DBA 角色 -- 拥有全部特权，是拥有系统最高权限的角色。只有 DBA 才可以创建数据库结构，并且系统权限也需要DBA 授予，且 DBA 用户可以操作全体用户的任意基表，包括删除。



**5、切换用户**

![](/img-post/2020-05-21-learning-notes-05/03.jpg)

随便选择一个登录过的用户，然后在弹出的登录框中填写新创建的用户名和密码即可。

![](/img-post/2020-05-21-learning-notes-05/04.jpg)


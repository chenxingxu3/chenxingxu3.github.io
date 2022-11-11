---
layout:     post
title:      Oracle数据库学习笔记（二十）
subtitle:   PL/SQL编程语言定义变量
date:       2020-05-22
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（二十）--PL/SQL编程语言定义变量

# PL/SQL 编程语言概念

PL/SQL 编程语言是对 SQL 语言的扩展，使得 SQL 语言具有过程化编程的特性，比一般的过程化编程语言更加灵活高效。PL/SQL 编程语言主要用来编写存储过程和存储函数等。

**PL/SQL 程序语法：**

```
DECLARE
	说明部分 （变量说明，游标申明，例外说明〕
BEGIN
	语句序列 （DML语句〕…
EXCEPTION
    例外处理语句
END;
```



# 学习笔记

**声明方法**

```sql
DECLARE
	i number(2) := 10;
	s varchar2(10) := '小明';
	ena emp.ename%TYPE; ---引用型变量
	emprow emp%ROWTYPE; ---记录型变量
BEGIN
	dbms_output.put_line(i);
	dbms_output.put_line(s);
	SELECT ename
	INTO ena
	FROM emp
	WHERE empno = 7788;
	dbms_output.put_line(ena);
	SELECT *
	INTO emprow
	FROM emp
	WHERE empno = 7788;
	dbms_output.put_line(emprow.ename || '的工作为：' || emprow.job);
END;
```

赋值操作可以使用 := 也可以使用 into 查询语句赋值。连接符是 ||

dbms_output.put_line() -- 相当于 Java 中的System.out.println()

emp.ename%type -- emp 表中 ename 这一列的数据类型

emp%rowtype -- 可以理解为 emp 表中的每一行记录都是一个对象，记录型变量就是用于存储这个对象
---
layout:     post
title:      Oracle数据库学习笔记（二十五）
subtitle:   存储函数
date:       2020-05-22
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（二十五）--存储函数

# 存储过程和存储函数的区别

**1、语法区别**

关键字不一样，存储函数比存储过程多了两个 return。

**2、本质区别**

存储函数有返回值，而存储过程没有返回值。

如果存储过程想实现有返回值的业务，我们就必须使用 out 类型的参数。即便是存储过程使用了 out 类型的参数，其本质也不是真的有了返回值，而是在存储过程内部给 out 类型参数赋值，在执行完毕后，我们直接拿到输出类型参数的值。

我们可以使用存储函数有返回值的特性，来自定义函数，而存储过程不能用来自定义函数。

# 创建存储函数语法

```
create or replace function 函数名(Name in type, Name in type, ...) return 数据类型 is
结果变量 数据类型;
begin
return(结果变量);
end函数名;
```

# 学习笔记

**1、通过存储函数实现计算指定员工的年薪**

```sql
create or replace function f_yearsal(eno emp.empno%type) return number
is
  s number(10);     
begin
  select sal*12+nvl(comm, 0) into s from emp where empno = eno;
  return s;
end;
```

return number -- 存储函数的返回值类型不能带长度

存储过程和存储函数的参数都不能带长度。

eno emp.empno%type -- 引用型变量如果原数据类型是带长度的，系统会自动去掉

**2、测试 f_yearsal**

```sql
declare
  s number(10); 
begin
  s := f_yearsal(7788);
  dbms_output.put_line(s);
end;
```

存储函数在调用的时候，返回值需要接收。

**3、查询出员工姓名，员工所在部门名称**

准备工作：把 scott 用户下的 dept 表复制到当前用户下

```sql
CREATE TABLE dept
AS
SELECT *
FROM scott.dept;
```

使用传统方式来实现案例需求

```sql
select e.ename, d.dname
from emp e, dept d
where e.deptno=d.deptno;
```

使用存储函数来实现提供一个部门编号，输出一个部门名称

```sql
create or replace function fdna(dno dept.deptno%type) return dept.dname%type
is
  dna dept.dname%type;
begin
  select dname into dna from dept where deptno = dno;
  return dna;
end;
```

使用 fdna 存储函数来实现案例需求：查询出员工姓名，员工所在部门名称

```sql
SELECT e.ename, fdna(e.deptno)
FROM emp e;
```


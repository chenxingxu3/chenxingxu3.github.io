---
layout:     post
title:      Oracle数据库学习笔记（二十四）
subtitle:   存储过程和out类型参数
date:       2020-05-22
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（二十四）--存储过程和out类型参数

# 存储过程概念

存储过程就是提前把已经编译好的一段 PL/SQL 语言，放置在数据库端，可以直接被调用。这一段 PL/SQL 一般都是固定步骤的业务。

**创建存储过程语法：**

```
create [or replace] PROCEDURE 过程名[(参数名 in/out 数据类型)]
AS
begin
PLSQL子程序体；
End;
```

或者

```
create [or replace] PROCEDURE 过程名[(参数名 in/out 数据类型)]
is
begin
PLSQL子程序体；
End 过程名;
```



# 学习笔记

**1、创建一个给指定员工涨 100 块钱的存储过程**

```sql
create or replace procedure p1(eno emp.empno%type)
is

begin
   update emp set sal=sal+100 where empno = eno;
   commit;
end;
```

**2、测试 p1存储过程**

```sql
declare

begin
  p1(7788);
end;
```

**3、使用查询语句验证结果**

```sql
SELECT *
FROM emp
WHERE empno = 7788;
```

**4、使用存储过程来算年薪**

```sql
create or replace procedure p_yearsal(eno emp.empno%type, yearsal out number)
is
   s number(10);
   c emp.comm%type;
begin
   select sal*12, nvl(comm, 0) into s, c from emp where empno = eno;
   yearsal := s+c;
end;
```

**5、测试 p_yearsal**

```sql
declare
  yearsal number(10);
begin
  p_yearsal(7788, yearsal);
  dbms_output.put_line(yearsal);
end;
```

**6、in 和 out 类型参数的区别是什么？**

凡是涉及到 into 查询语句赋值或者 := 赋值操作的参数，都必须使用 out 来修饰。
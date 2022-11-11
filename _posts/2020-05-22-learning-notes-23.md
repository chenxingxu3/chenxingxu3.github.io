---
layout:     post
title:      Oracle数据库学习笔记（二十三）
subtitle:   -PL/SQL中的游标
date:       2020-05-22
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（二十三）--PL/SQL中的游标

# 游标的概念

类似于 Java 中的集合，可以存放多个对象，多行记录。

# 学习笔记

**1、输出 emp 表中所有员工的姓名**

```sql
declare
  cursor c1 is select * from emp;
  emprow emp%rowtype; --记录型变量
begin
  open c1;
     loop
         fetch c1 into emprow;
         exit when c1%notfound;
         dbms_output.put_line(emprow.ename);
     end loop;
  close c1;
end;
```

**2、给指定部门员工涨工资**

```sql
declare
  cursor c2(eno emp.deptno%type) 
  is select empno from emp where deptno = eno;
  en emp.empno%type;
begin
  open c2(10);
     loop
        fetch c2 into en;
        exit when c2%notfound;
        update emp set sal=sal+100 where empno=en;
        commit;
     end loop;  
  close c2;
end;
```

cursor c2(eno emp.deptno%type) -- 定义了一个带参数的游标，使用的时候可以传入参数。

**3、查询 10 号部门员工信息**

```sql
SELECT *
FROM emp
WHERE deptno = 10;
```


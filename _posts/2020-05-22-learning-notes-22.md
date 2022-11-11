---
layout:     post
title:      Oracle数据库学习笔记（二十二）
subtitle:   PL/SQL中的循环
date:       2020-05-22
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（二十二）--PL/SQL中的循环

# PL/SQL 中的 loop 循环

用三种方式输出 1 到 10 这十个数字

## while 循环

```sql
declare
  i number(2) := 1;
begin
  while i<11 loop
     dbms_output.put_line(i);
     i := i+1;
  end loop;  
end;
```



## exit 循环【重点掌握】

```sql
declare
  i number(2) := 1;
begin
  loop
    exit when i>10;
    dbms_output.put_line(i);
    i := i+1;
  end loop;
end;
```



## for 循环

```sql
declare

begin
  for i in 1..10 loop
     dbms_output.put_line(i);  
  end loop;
end;

```


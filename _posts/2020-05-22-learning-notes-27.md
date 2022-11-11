---
layout:     post
title:      Oracle数据库学习笔记（二十七）
subtitle:   触发器实现主键自增
date:       2020-05-22
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（二十七）--触发器实现主键自增

# 分析

在用户做插入操作的之前，拿到即将插入的数据，给该数据中的主键列赋值。使用的行级触发器。

# 实现

**1、创建触发器**

```sql
create or replace trigger auid
before
insert
on person
for each row
declare

begin
  select s_person.nextval into :new.pid from dual;
end;
```

**2、查询 person 表数据**

```sql
SELECT *
FROM person;
```

**3、使用 auid 实现主键自增**

```sql
insert into person (pname) values ('a');
commit;
insert into person values (1, 'b');
commit;
```

insert into person values (1, 'b') -- 表中已经有主键为 1 的记录了，执行完这条语句后原主键为 1 的记录不会被覆盖掉，而 b 值使用了新的序列所分配的值作为 pid 插入到了表中。
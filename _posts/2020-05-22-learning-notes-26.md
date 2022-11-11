---
layout:     post
title:      Oracle数据库学习笔记（二十六）
subtitle:   触发器的概念和分类
date:       2020-05-22
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（二十六）--触发器的概念和分类

# 触发器的概念

触发器就是制定一个规则，当我们做增删改操作的时候，只要满足该规则，自动触发，无需调用。

# 触发器的分类

1. 语句级触发器：不包含有 for each row 的触发器。
2. 行级触发器：包含有 for each row 的就是行级触发器。

加 for each row 是为了使用 :old 或者 :new 的对象/一行记录。

# 在触发器中触发语句与伪记录变量的值

| 触发语句 | :old                    | :new                    |
| -------- | ----------------------- | ----------------------- |
| Insert   | 所有字段都是空 ( null ) | 将要插入的数据          |
| Update   | 更新以前该行的值        | 更新后的值              |
| Delete   | 删除以前该行的值        | 所有字段都是空 ( null ) |



# 学习笔记

**1、插入一条记录，输出一个新员工入职【语句级触发器】**

```sql
create or replace trigger t1
after
insert
on person
declare

begin
  dbms_output.put_line('一个新员工入职');
end;
```

触发t1

```sql
INSERT INTO person
VALUES (1, '小红');

COMMIT;

SELECT *
FROM person;
```

**2、不能给员工降薪【行级别触发器】**

```sql
create or replace trigger t2
before
update
on emp
for each row
declare

begin
  if :old.sal>:new.sal then
     raise_application_error(-20001, '不能给员工降薪');
  end if;
end;
```

自定义异常：raise_application_error(-20001~-20999之间, '错误提示信息');

触发t2

```sql
SELECT *
FROM emp
WHERE empno = 7788;

UPDATE emp
SET sal = sal - 1
WHERE empno = 7788;

COMMIT;
```


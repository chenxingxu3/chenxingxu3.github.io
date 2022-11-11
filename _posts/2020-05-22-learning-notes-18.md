---
layout:     post
title:      Oracle数据库学习笔记（十八）
subtitle:   视图
date:       2020-05-22
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（十八）--视图

# 视图概念

视图就是提供一个查询的窗口，里面没有存放数据，所有数据都来自于原表。

视图的作用：

1. **视图可以屏蔽掉一些敏感字段。**比如为了防止员工看到工资，可以创建一个视图，把工资那一列屏蔽掉。
2. **保证总部和分部数据及时统一。**比如总部和分部共同销售一种商品，总部这边销售一件出去了，分部通过查询视图可以及时更新商品的库存数据。如果分部与总部各自查询独立的表，就会发生总部库存卖空后，分部这边依旧接单的情况，而总部创建一个视图供分部使用就可以避免这种情况的发生。

# 学习笔记

**1、创建视图**

必须有 dba 权限，需要切换到具有 dba 权限的用户。

切换完成后，使用查询语句创建表。

```sql
CREATE TABLE emp
AS
SELECT *
FROM scott.emp;
```

然后完成创建视图的操作。

```sql
CREATE VIEW v_emp
AS
SELECT ename, job
FROM emp;
```

**2、查询视图**

```sql
SELECT *
FROM v_emp;
```

**3、修改视图**

```sql
UPDATE v_emp
SET job = 'CLERK'
WHERE ename = 'ALLEN';
COMMIT;
```

原表的数据也被修改了，所以不推荐对视图进行修改。实际开发过程中为了防止修改视图，公司会创建只读视图供开发人员使用。

**4、创建只读视图**

```sql
CREATE VIEW v_emp1
AS
SELECT ename, job
FROM emp
WITH READ ONLY;
```


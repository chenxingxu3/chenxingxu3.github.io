---
layout:     post
title:      Oracle数据库学习笔记（六）
subtitle:   数据类型介绍及表的创建
date:       2020-05-21
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（六）--数据类型介绍及表的创建

# 一、Oracle 数据类型

| No   | 数据类型           | 描述                                                    |
| ---- | ------------------ | ------------------------------------------------------- |
| 1    | VARCHAR， VARCHAR2 | 表示一个字符串                                          |
| 2    | NUMBER             | NUMBER(n)表示一个整数，长度是n                          |
| 3    | NUMBER             | NUMBER(m,n):表示一个小数，总长度是m，小数是n，整数是m-n |
| 4    | DATE               | 表示日期类型                                            |
| 5    | CLOB               | 大对象，表示大文本数据类型，可存 4 G                    |
| 6    | BLOB               | 大对象，表示二进制数据，可存 4 G                        |

比较常用的是 VARCHAR2 ，可变长度的字符串。如果实际所存的字符串长度比规定的要小的话，会自动截断，但是实际所存的字符串长度比规定的要大的话，不会自动扩展。

NUMBER(2) 表示最大两位数，也就是 99。NUMBER(4,2) 表示总长度为 4，占 2 位小数。也就是最大可以存 99.99。

Oracle 中也有 INTEGER 整数类型，但是只能存整数不能存小数，所以一般情况下都使用 NUMBER 类型。

DATE 相当于 MySQL 中的 DATETIME 类型。

BLOB 用于存储二进制数据。比如存取视频，应该选取 BLOB 数据类型。

# 二、学习笔记

**1、创建一个 person 表**

```sql
CREATE TABLE person (
	pid NUMBER(20),
	pname VARCHAR2(10)
);
```

**2、修改表结构**

添加一列

```sql
ALTER TABLE person
	ADD COLUMN gender NUMBER(1);
```

修改列类型

```sql
ALTER TABLE person
	MODIFY COLUMN gender CHAR(1);
```

修改列名称

```sql
ALTER TABLE person
	RENAME COLUMN gender TO sex;
```

删除一列

```sql
ALTER TABLE person
	DROP COLUMN sex;
```


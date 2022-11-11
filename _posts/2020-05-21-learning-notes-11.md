---
layout:     post
title:      Oracle数据库学习笔记（十一）
subtitle:   条件表达式
date:       2020-05-21
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（十一）--条件表达式

# 通用写法

条件表达式的通用写法在 MySQL 和 Oracle 均可正常使用。建议使用通用写法，保证可重用性。因为一旦公司因为某些原因要更换数据库，使用通用写法移植成本会适当降低。

1、给 emp 表中员工起中文名

```sql
SELECT e.ename
	, CASE e.ename
		WHEN 'SMITH' THEN '史密斯'
		WHEN 'ALLEN' THEN '艾伦'
		WHEN 'WARD' THEN '沃德'
		ELSE '无名'
	END
FROM emp e;
```

如果想要让其余的名字变为 null，可以参考下面的写法，不写 else 语句。

```sql
SELECT e.ename
	, CASE e.ename
		WHEN 'SMITH' THEN '史密斯'
		WHEN 'ALLEN' THEN '艾伦'
		WHEN 'WARD' THEN '沃德'
	END
FROM emp e;
```

2、判断 emp 表中员工工资。如果高于 3000 显示“高收入”，如果高于 1500 低于 3000 显示“中等收入”，其余显示“低收入”。

```sql
SELECT e.sal
	, CASE 
		WHEN e.sal > 3000 THEN '高收入'
		WHEN e.sal > 1500 THEN '中等收入'
		ELSE '低收入'
	END
FROM emp e;
```

# Oracle 专用条件表达式

Oracle 中除了起别名，都用单引号。所以下面的别名使用了双引号，即"中文名"。加了单引号的话会报错。

给 emp 表中员工起中文名

```sql
SELECT e.ename, 
        decode(e.ename,
          'SMITH',  '史密斯',
          'ALLEN',  '艾伦',
          'WARD',  '沃德',
          '无名') "中文名"             
FROM emp e;
```

别名不加双引号，也不加单引号也是可以的。

```sql
SELECT e.ename, 
        decode(e.ename,
          'SMITH',  '史密斯',
          'ALLEN',  '艾伦',
          'WARD',  '沃德',
          '无名') 中文名             
FROM emp e;
```


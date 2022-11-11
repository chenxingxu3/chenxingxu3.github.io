---
layout:     post
title:      Oracle数据库学习笔记（十七）
subtitle:   分页查询
date:       2020-05-22
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（十七）--分页查询

# rownum行号的概念

当我们做 SELECT 操作的时候，每查询出一行记录，就会在该行上加上一个行号，行号从 1 开始，依次递增，不能跳着走。

# 学习笔记

**emp 表工资倒序排列后，每页五条记录，查询第二页**

```sql
SELECT *
FROM (
	SELECT rownum AS rn, tt.*
	FROM (
		SELECT *
		FROM emp
		ORDER BY sal DESC
	) tt
	WHERE rownum < 11
)
WHERE rn > 5;
```

可以将分页查询总结成一个**模板**来使用

```sql
SELECT *
FROM (
	SELECT rownum rn, tt.*
	FROM (
		--{查询语句}
	) tt
	WHERE rownum < --{行号}
)
WHERE rn > --{行号}
;
```

分步解析：

**1、根据 emp 表的工资进行倒序排列**

```sql
SELECT rownum, e.*
FROM emp e
ORDER BY e.sal DESC
```

rownum 不可以写成 e.rownum，因为 rownum 不属于 emp 这张表，但是每张表都可以用。

查询出结果后，可以看到 rownum 这一列是乱序的，因为 SELECT 会先执行，然后再执行 ORDER BY 排序。排序操作会影响 rownum 的顺序，可以考虑先排序再加行号，具体操作就是嵌套查询。

**2、如果涉及到排序，但是还要使用 rownum 的话，我们可以再次嵌套查询**

```sql
SELECT rownum, t.*
FROM (
	SELECT rownum, e.*
	FROM emp e
	ORDER BY e.sal DESC
) t;
```

**3、改写第 1 步的查询语句**

```sql
SELECT *
FROM emp
ORDER BY sal DESC
```

**4、把第 3 步的语句括起来，查询出带有正确 rownum 排序的结果**

```sql
SELECT rownum, e.*
FROM (
	SELECT *
	FROM emp
	ORDER BY sal DESC
) e
```

**5、添加 WHERE 查询条件，查出 rownum 小于 11 的结果，也就是 1 到 10 的结果** 

```sql
SELECT rownum, e.*
FROM (
	SELECT *
	FROM emp
	ORDER BY sal DESC
) e
WHERE rownum < 11
```

**6、再次添加 rownum > 5 这个条件后发现结果是不正确的**

这是因为行号从 1 开始，依次递增，不能跳着走。WHERE 先于 SELECT 执行，此时行号从 1 开始，rownum < 11这个条件，也就是 1 < 11 结果为真，rownum > 5 这个条件，也就是 1 > 5 结果为假，所以 rownum < 11 and rownum > 5 的结果为假。

rownum 行号不能添加大于一个正数这样的条件，所以要想办法间接地写大于一个正数这样的条件。

**7、将第 5 步的语句外面再嵌套一层查询语句**

```sql
SELECT *
FROM (
	SELECT rownum AS rn, e.*
	FROM (
		SELECT *
		FROM emp
		ORDER BY sal DESC
	) e
	WHERE rownum < 11
) tt
WHERE rn > 5
```

因为 rownum 不属于任何一张表，所以不可以使用 tt.rownum 这样的写法来指定某一个 rownum，只能通过为rownum 起别名的方式来指定某一个具体的 rownum。

**8、将第 7 步的语句进行进一步优化，得到最终答案**

```sql
SELECT *
FROM (
	SELECT rownum AS rn, tt.*
	FROM (
		SELECT *
		FROM emp
		ORDER BY sal DESC
	) tt
	WHERE rownum < 11
)
WHERE rn > 5;
```


---
layout:     post
title:      Oracle数据库学习笔记（八）
subtitle:   序列的使用
date:       2020-05-21
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（八）--序列的使用

# 序列的概念

在很多数据库中都存在一个自动增长的列，如果现在要想在 Oracle 中完成自动增长的功能，则只能依靠序列完成。所有的自动增长操作，需要用户手工完成处理。简单来说，序列主要用来给主键赋值使用。

序列默认从 1 开始，依次递增。序列实际上不属于任何一张表，但是可以和表做逻辑上的绑定。

# 学习笔记

## **1、序列创建语法**

```
CREATE SEQUENCE 序列名
[INCREMENT BY n]
[START WITH n]
[{MAXVALUE/ MINVALUE n|NOMAXVALUE/ NOMINVALUE}]
[{CYCLE|NOCYCLE}]
[{CACHE n|NOCACHE}];
```

[INCREMENT BY n] -- 指定每次增加多少。默认情况下是 1。比如 INCREMENT BY 2，那么序列就是 1、3、5、7……

[START WITH n] -- 指定从几开始。默认情况是从 1 开始。比如 START WITH 2，那么序列就是从 2 开始。

[{MAXVALUE n/ MINVALUE n|NOMAXVALUE/ NOMINVALUE}] -- 指定最大值、最小值。在普通开发过程中，这个属性很少被使用。可以参考以下几种用法：

- MAXVALUE 999999
- MINVALUE 1
- MINVALUE 1 NOMAXVALUE

[{CYCLE|NOCYCLE}] -- 指定是否循环。在普通开发过程中，这个属性基本不会被使用。序列最常用的使用场景就是用来给主键赋值，而主键要求是非空和唯一的，使用循环后主键的唯一性就被破坏了，自然就不能用到循环这个属性。

[{CACHE n|NOCACHE}] -- 指定缓存。比如说序列当前值为 8，指定了 CACHE 2 这个属性，Oracle 会提前把 9 和 10 缓存好，用的时候会适当提高下效率。但是实际上效果不是很明显，所以这个属性也很少被用到。

## **2、创建一个序列**

```sql
CREATE SEQUENCE s_person;
```

## **3、序列的两种操作**

序列创建完成之后,所有的自动增长应该由用户自己处理,所以在序列中提供了以下的两种操作：

1. nextval ：取得序列的下一个内容
2. currval ：取得序列的当前内容

取得序列的下一个内容

```sql
SELECT s_person.NEXTVAL
FROM dual;
```

取得序列的当前内容

```sql
SELECT s_person.CURRVAL
FROM dual;
```

其中，dual 是虚表，只是为了补全语法，没有任何意义。

注意：新创建好了一个序列之后，要先使用 nextval 操作，如果先使用 currval 操作会报错，因为如果不先执行一次 nextval 操作，那么当前值就为空。执行了一次 nextval 操作后，再执行 currval 操作就不会报错了。

## **4、在插入数据时，使用序列实现主键自增**

```sql
INSERT INTO person (pid, pname)
VALUES (s_person.NEXTVAL, '小明');
COMMIT;
```

如果执行了插入语句，没有提交，而是进行了回滚操作（此时序列值为 7），再次执行插入语句，进行提交操作（此时序列值为 8）后，此时的主键值为 8。

表中的主键不是连续的数字没有任何问题，因为在实际情况下，随时都有可能要删除记录，主键很难保证连续。只要保证主键的值是非空、唯一即可。
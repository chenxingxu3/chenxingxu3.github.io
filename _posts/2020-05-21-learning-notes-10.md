---
layout:     post
title:      Oracle数据库学习笔记（十）
subtitle:   单行函数
date:       2020-05-21
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（十）--单行函数

# 单行函数的概念

作用于一行，返回一个值。

# 字符函数

**1、小写变大写**

```sql
SELECT upper('yes')
FROM dual;
```

结果：YES

**2、大写变小写**

```sql
SELECT lower('YES')
FROM dual;
```

结果：yes

# 数值函数

**1、四舍五入，后面的参数表示保留的位数**

```sql
SELECT round(26.18, 1)
FROM dual;
```

结果：26.2

```sql
SELECT round(26.14, 1)
FROM dual;
```

结果：26.1

```sql
SELECT round(26.16, -1)
FROM dual;
```

结果：30

```sql
SELECT round(26.16, -2)
FROM dual;
```

结果：0

```sql
SELECT round(56.16, -2)
FROM dual;
```

结果：100

**2、直接截取，不在看后面位数的数字是否大于5**

```sql
SELECT trunc(56.16)
FROM dual;
```

结果：56

```sql
SELECT trunc(56.16, 1)
FROM dual;
```

结果：56.1

```sql
SELECT trunc(56.16, -1)
FROM dual;
```

结果：50

**3、求余数**

```sql
SELECT mod(10, 3)
FROM dual;
```

结果：1

# 日期函数

**1、查询出 emp 表中所有员工入职距离现在几天**

```sql
SELECT SYSDATE - e.hiredate
FROM emp e;
```

**2、算出明天此刻**

```sql
SELECT SYSDATE + 1
FROM dual;
```

**3、查询出 emp 表中所有员工入职距离现在几月**

```sql
SELECT months_between(SYSDATE, e.hiredate)
FROM emp e;
```

**注意**：只有月份有 months_between 函数，其余都没有。

**4、查询出 emp 表中所有员工入职距离现在几年**

```sql
SELECT months_between(SYSDATE, e.hiredate) / 12
FROM emp e;
```

**5、查询出 emp 表中所有员工入职距离现在几周**

```sql
SELECT round((SYSDATE - e.hiredate) / 7)
FROM emp e;
```

# 转换函数

**1、日期转字符串**

```sql
SELECT to_char(SYSDATE, 'fm yyyy-mm-dd hh24:mi:ss')
FROM dual;
```

fm 表示去除日期当中的数字 0。例如，将 2019-07-08 04:49:51 转换为 2019-7-8 4:49:51。

在 hh 后面加上 24，即 hh24 就变为 24 小时制时间了。

输出的是 CHAR 类型。

**2、字符串转日期**

```sql
SELECT to_date('2019-7-8 17:49:51', 'fm yyyy-mm-dd hh24:mi:ss')
FROM dual;
```

输出的是 DATE 类型。

# 通用函数

算出 emp 表中所有员工的年薪。奖金里面有 null 值，如果 null 值和任意数字做算术运算，结果都是 null。所以要使用 nvl(e.comm, 0) 将 null 值转换为数字 0.

nvl 函数格式：nvl(expr1,expr2)

如果第一个参数为 null，那么显示第二个参数的值；如果第一个参数的值不为 null，则显示第一个参数本来的值。

```sql
SELECT e.sal * 12 + nvl(e.comm, 0)
FROM emp e;
```




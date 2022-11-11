---
layout:     post
title:      Oracle数据库学习笔记（二十一）
subtitle:   PL/SQL中的if判断
date:       2020-05-22
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（二十一）--PL/SQL中的if判断

# 案例

**输入小于 18 的数字，输出未成年；输入大于 18 小于 40 的数字，输出中年人；输入大于 40 的数字，输出老年人**

```sql
declare
  i number(3) := &ii;
begin
  if i<18 then
    dbms_output.put_line('未成年');
  elsif i<40 then
    dbms_output.put_line('中年人');
  else
    dbms_output.put_line('老年人');
  end if;
end;
```

elsif -- 也就是else if

&ii -- 获取输入的值。ii是自定义的名称
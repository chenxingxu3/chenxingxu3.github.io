---
layout:     post
title:      07.SVN入门笔记
subtitle:   查看目录或文件日志信息
date:       2020-12-13
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SVN
---
# 07.SVN入门笔记——查看目录或文件日志信息

使用 svn log 命令

[注意：使用这个命令的前提是设置匿名访问为 none，即：anon-access = none，否则会出现“svn: E220001: Item is not readable”错误]

![](/img-post/2020-12-13-svn-ln-introduction/a-07-01.png)

执行效果如下：

```
D:\DevWorkSpace\SVNSpace\ERP>svn info test.txt
Path: test.txt
Name: test.txt
Working Copy Root Path: D:\DevWorkSpace\SVNSpace\ERP
URL: svn://192.168.25.136/ERP/test.txt
Relative URL: ^/test.txt
Repository Root: svn://192.168.25.136/ERP
Repository UUID: 39db0184-d422-af4a-8388-e037c5abf675
Revision: 2
Node Kind: file
Schedule: normal
Last Changed Rev: 2
Last Changed Date: 2020-12-12 16:39:50 +0800 (周六, 12 十二月 2020)
Text Last Updated: 2020-12-12 16:39:06 +0800 (周六, 12 十二月 2020)
Checksum: cbe88544452697fcd2050a463419ca80fb74bb08


D:\DevWorkSpace\SVNSpace\ERP>svn log test.txt
svn: E220001: Item is not readable

D:\DevWorkSpace\SVNSpace\ERP>svn log test.txt
------------------------------------------------------------------------
r2 | (no author) | 2020-12-12 16:39:50 +0800 (周六, 12 十二月 2020) | 1 line

My second commit
------------------------------------------------------------------------
r1 | (no author) | 2020-12-12 16:35:13 +0800 (周六, 12 十二月 2020) | 1 line

My first commit
------------------------------------------------------------------------
```


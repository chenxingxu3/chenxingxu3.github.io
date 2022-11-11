---
layout:     post
title:      06.SVN入门笔记
subtitle:   查看工作副本信息
date:       2020-12-13
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SVN
---
# 06.SVN入门笔记——查看工作副本信息

使用 svn info 命令

执行效果如下：

```
D:\DevWorkSpace\SVNSpace\ERP>svn info
Path: .
Working Copy Root Path: D:\DevWorkSpace\SVNSpace\ERP
URL: svn://192.168.25.136/ERP
Relative URL: ^/
Repository Root: svn://192.168.25.136/ERP
Repository UUID: 39db0184-d422-af4a-8388-e037c5abf675
Revision: 0
Node Kind: directory
Schedule: normal
Last Changed Rev: 0
Last Changed Date: 2020-12-12 15:28:08 +0800 (周六, 12 十二月 2020)
```

![](/img-post/2020-12-13-svn-ln-introduction/a-06-01.png)

对某一个文件使用 svn info 命令

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
```

![](/img-post/2020-12-13-svn-ln-introduction/a-06-02.png)
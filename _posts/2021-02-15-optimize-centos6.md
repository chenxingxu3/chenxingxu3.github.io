---
layout:     post
title:      CentOS 6 study notes (2)
subtitle:   Optimizing CentOS 6
date:       2021-02-15
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - CentOS
---

以管理员权限编辑：

vim /etc/inittab

把 id:5:initdefault: 改为 id:3:initdefault: 即可。
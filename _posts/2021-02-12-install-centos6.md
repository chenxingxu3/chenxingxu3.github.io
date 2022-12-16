---
layout:     post
title:      CentOS 6 study notes (1)
subtitle:   Install CentOS 6
date:       2021-02-12
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - CentOS
---

# 安装CentOS 

# 下载 CentOS 6 安装镜像

CentOS 官方网站：

[https://www.centos.org/](https://www.centos.org/)

然而目前官方网站已经不再提供 CentOS 6 版本的安装镜像了。

一般情况下，只下载 CentOS-6.10-x86_64-bin-DVD1.iso 就可以了。

# 创建 VMware 虚拟机

●文件 -> 新建虚拟机

![](/img-post/2021-02-12-install-centos6/01.jpg)

●典型（推荐）-> 下一步

![](/img-post/2021-02-12-install-centos6/02.jpg)

●稍后安装操作系统 -> 下一步

![](/img-post/2021-02-12-install-centos6/03.jpg)

●Linux -> CentOS 6 -> 下一步

![](/img-post/2021-02-12-install-centos6/04.jpg)

●填写好 虚拟机名称 和 位置，点击下一步

![](/img-post/2021-02-12-install-centos6/05.jpg)

●最大磁盘大小 设置为 60 GB，选择 将虚拟磁盘存储为单个文件，点击下一步

![](/img-post/2021-02-12-install-centos6/06.jpg)

●点击 自定义硬件

![](/img-post/2021-02-12-install-centos6/07.jpg)

●CD/DVD (IDE) -> 使用 ISO 映像文件 -> 浏览，选择 CentOS 的 iso 映像文件，设置完成后点击确定

![](/img-post/2021-02-12-install-centos6/08.jpg)

# 安装 CentOS

●开启刚刚创建好的虚拟机

![](/img-post/2021-02-12-install-centos6/09.jpg)

●选择第一项，Install or upgrade an existing system

![](/img-post/2021-02-12-install-centos6/10.jpg)

●等待安装程序加载

![](/img-post/2021-02-12-install-centos6/11.jpg)

●选择 Skip

![](/img-post/2021-02-12-install-centos6/12.jpg)

●选择 Next

![](/img-post/2021-02-12-install-centos6/13.jpg)

●English (English) -> Next

![](/img-post/2021-02-12-install-centos6/14.jpg)

●U.S. English -> Next

![](/img-post/2021-02-12-install-centos6/15.jpg)

●Basic Storage Devices -> Next

![](/img-post/2021-02-12-install-centos6/16.jpg)

●Yes, discard any data

![](/img-post/2021-02-12-install-centos6/17.jpg)

●填写好 Hostname，点击 Next

![](/img-post/2021-02-12-install-centos6/18.jpg)

●时区选择 Asia/Shanghai，点击 Next

![](/img-post/2021-02-12-install-centos6/19.jpg)

●设置 Root password，在 Confirm 一栏中重复输入在 Root password 设置好的密码，点击Next

![](/img-post/2021-02-12-install-centos6/20.jpg)

●点击 Use Anyway 【注意：在实际生产环境中不要使用弱密码】

![](/img-post/2021-02-12-install-centos6/21.jpg)

●Replace Existing Linux System(s) -> Next

![](/img-post/2021-02-12-install-centos6/22.jpg)

●Write changes to disk

![](/img-post/2021-02-12-install-centos6/23.jpg)

●安装正式开始，等待 CentOS 安装完成即可

![](/img-post/2021-02-12-install-centos6/24.jpg)



![](/img-post/2021-02-12-install-centos6/25.jpg)

●点击 Reboot 重启系统

![](/img-post/2021-02-12-install-centos6/26.jpg)

●等待系统重启完成

![](/img-post/2021-02-12-install-centos6/27.jpg)



![](/img-post/2021-02-12-install-centos6/28.jpg)



![](/img-post/2021-02-12-install-centos6/29.jpg)

●看到下面这个页面，说明 CentOS 正式安装完成

![](/img-post/2021-02-12-install-centos6/30.jpg)
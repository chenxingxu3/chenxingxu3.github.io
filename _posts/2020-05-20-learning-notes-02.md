---
layout:     post
title:      Oracle数据库学习笔记（二）
subtitle:   在服务端安装Oralce数据库
date:       2020-05-20
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---





# 前期准备

Oracle Database 10g Release 2 (10.2.0.1.0) 

# 安装过程

1、打开 setup.exe 出现如下的安装界面，输入 数据库口令 和 确认口令，为了方便学习，本人在此设置的密码为 password 。点击 下一步 。

![01](/img-post/2020-05-20-learning-notes-02/01.jpg)

![02](/img-post/2020-05-20-learning-notes-02/02.jpg)

2、检查先决条件，选中红框所示的选择框，如下图。然后点击 下一步 。

![03](/img-post/2020-05-20-learning-notes-02/03.jpg)

3、点击 安装 。

![04](/img-post/2020-05-20-learning-notes-02/04.jpg)

![05](/img-post/2020-05-20-learning-notes-02/05.jpg)

![06](/img-post/2020-05-20-learning-notes-02/06.jpg)

![07](/img-post/2020-05-20-learning-notes-02/07.jpg)

4、安装完成后，出现下图的界面，点击 口令管理 。

![08](/img-post/2020-05-20-learning-notes-02/08.jpg)

5、将 SCOTT 和 HR 用户的勾去掉（解锁这两个账户）。然后点击 确定 。

![09](/img-post/2020-05-20-learning-notes-02/09.jpg)

6、点击 确定 。

![10](/img-post/2020-05-20-learning-notes-02/10.jpg)

![11](/img-post/2020-05-20-learning-notes-02/11.jpg)

7、安装完成，点击 退出 。

![12](/img-post/2020-05-20-learning-notes-02/12.jpg)

![13](/img-post/2020-05-20-learning-notes-02/13.jpg)

# 测试安装结果

## 方法一：

1、安装完成后，会自动弹出一个网页，在此网页中输入 用户名：system 和刚刚安装时设置的口令。

![14](/img-post/2020-05-20-learning-notes-02/14.jpg)

2、拉到页面最下方，点击 我同意 。

![15](/img-post/2020-05-20-learning-notes-02/15.jpg)

3、出现如下页面，说明数据库安装成功。

![16](/img-post/2020-05-20-learning-notes-02/16.jpg)

## 方法二：

1、按照下图，找到 SQL Plus 。

![17](/img-post/2020-05-20-learning-notes-02/17.jpg)

2、输入 用户名：system 和刚刚安装时设置的口令。

![18](/img-post/2020-05-20-learning-notes-02/18.jpg)

3、出现如下界面，说明数据库安装成功。

![19](/img-post/2020-05-20-learning-notes-02/19.jpg)
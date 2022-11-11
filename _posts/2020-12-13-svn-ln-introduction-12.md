---
layout:     post
title:      10.SVN入门笔记
subtitle:   使用SVN独立客户端TortoiseSVN
date:       2020-12-13
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SVN
---
# 10.SVN入门笔记——使用 SVN 独立客户端TortoiseSVN

## TortoiseSVN 简介

TortoiseSVN 是一个 Windows 下的版本控制系统 Apache™ Subversion®的客户端工具。

![10-01](/img-post/2020-12-13-svn-ln-introduction/10-01.jpg)

## TortoiseSVN 的优良特性

### 外壳集成

TortoiseSVN 无缝地整合进 Windows 的外壳 (例如资源管理器)。

### 重载图标

每个版本控制的文件和目录的状态使用小的重载图标表示，可以让你立刻看出工作副本的状态。

### 图形用户界面

当你列出文件或文件夹的更改时，你可以点击任意版本查看提交注释。也可以看到更改过的文件列表 - 只要双击文件就可以查看更改内容。

提交对话框列出了本次提交将要包括的条目，每一个条目有一个复选框，所以你可以选择包括哪些条目。未被版本控制管理的文件也会被列出，以防你忘记添加新文件。

### Subversion 命令的简便访问

TortoiseSVN 将所有的 Subversion 命令添加到了资源管理器的右键菜单。

## TortoiseSVN 的历史

2002 年，Tim Kemp 发现 Subversion 是一个非常好的版本管理系统，但是缺乏一个好的图形界面客户端程序。做一个与 Windows 外壳整合的 Subversion 客户端程序的想法是受一个叫 TortoiseCVS 的 CVS 客户端程序所启发的。Tim 研究了 TortoiseCVS 的源码并在此基础上开发了 TortoiseSVN。随后他注册了域名 tortoisesvn.org 并且将源码放在了网上。

与此同时， Stefan Küng 正在寻找一个好用的并且免费的版本控制系统， 他也发现了 Subversion，同时也发现了客户端 TortoiseSVN。但是此时的 TortoiseSVN 还不能正常使用，他索性加入了项目并贡献代码。然而，他重写了现有的大部分代码，最终，最初的代码已经都被改写了。

由于 Subversion 的用户越来越多， TortoiseSVN 作为 Subversion 的图形界面客户端程序，用户数量也随之快速增长。Lübbe Onken 为 TortoiseSVN  项目提供了精美的图标，绘制了 TortoiseSVN 的标志。现在他负责照看网站和管理多语言翻译。

## TortoiseSVN 安装

下载安装程序：

http://tortoisesvn.net/downloads.html

[https://gitee.com/zhuzhulu/java-dev-software](https://gitee.com/zhuzhulu/java-dev-software)

以 Win64 位为例演示安装过程：

打开安装文件，点击 Next

![](/img-post/2020-12-13-svn-ln-introduction/a-10-01.png)

点击 Next

![](/img-post/2020-12-13-svn-ln-introduction/a-10-02.png)

这里保持默认即可，如果需要更换安装目录可以点击 Browse 设置。点击 Next

![](/img-post/2020-12-13-svn-ln-introduction/a-10-03.png)

点击 Install

![](/img-post/2020-12-13-svn-ln-introduction/a-10-04.png)

等待安装完成

![](/img-post/2020-12-13-svn-ln-introduction/a-10-05.png)

点击 Finish

![](/img-post/2020-12-13-svn-ln-introduction/a-10-06.png)



## 可能出现的问题

在 Windows 7 系统下使用 Repo-browser 编辑文件后提交会报下面的错误：

![](/img-post/2020-12-13-svn-ln-introduction/a-11-06.png)

但是提交一切正常。换成 Windows 10 下使用不会报这个错误。所以条件允许的话，最好在 Windows 10 下使用新版本的 TortoiseSVN. 但是如果不使用 Repo-browser 编辑提交文件的话，这个问题可以忽略。

## 检出

创建一个目录用来存放检出得到的文件，例如 D:\DevWorkSpace\SVNSpace\jack\OA

进入目录，点右键，点击 SVN Checkout

![](/img-post/2020-12-13-svn-ln-introduction/a-10-07.png)

填写好 URL 后点击 OK

![](/img-post/2020-12-13-svn-ln-introduction/a-10-08.png)

开始检出

![](/img-post/2020-12-13-svn-ln-introduction/a-10-09.png)

可以看到检出得到的文件

![](/img-post/2020-12-13-svn-ln-introduction/a-10-10.png)

如果文件图标上没有任何标识，可以尝试重启外壳程序——explorer.exe . 打开任务管理器，选中 explorer.exe 进程，结束进程，然后新建进程 explorer.exe 就可以了。

![](/img-post/2020-12-13-svn-ln-introduction/10-22.jpg)

![](/img-post/2020-12-13-svn-ln-introduction/10-23.jpg)

![](/img-post/2020-12-13-svn-ln-introduction/10-24.jpg)

如果一切顺利的话，你会看到文件图标变成了这样：

![](/img-post/2020-12-13-svn-ln-introduction/a-10-10.png)

TortoiseSVN 图标含义

![](/img-post/2020-12-13-svn-ln-introduction/10-26.jpg)

- ![](/img-post/2020-12-13-svn-ln-introduction/10-27.jpg)：一个新检出的工作副本使用绿色的对勾做重载。表示 Subversion 状态正常。
- ![](/img-post/2020-12-13-svn-ln-introduction/10-28.jpg)：在你开始编辑一个文件后，状态就变成了已修改，而图标重载变成了红色感叹号。通过这种方式，你可以很容易地看出哪些文件从你上次更新工作副本后被修改过，需要被提交。
- ![](/img-post/2020-12-13-svn-ln-introduction/10-29.jpg)：如果在更新的过程中出现了冲突，图标会变成黄色感叹号。
- ![](/img-post/2020-12-13-svn-ln-introduction/10-30.jpg)：如果你给一个文件设置了 svn:needs-lock 属性，Subversion 会让此文件只读，直到你获得文件锁。具有这个重载图标的文件来表示你必须在编辑之前先得到锁。
- ![](/img-post/2020-12-13-svn-ln-introduction/10-31.jpg)：如果你拥有了一个文件的锁，并且 Subversion 状态是正常，这个重载图标就提醒你如果不使用该文件的话应该释放锁，允许别人提交对该文件的修改。
- ![](/img-post/2020-12-13-svn-ln-introduction/10-32.jpg)：这个图标表示当前文件夹下的某些文件或文件夹已经被调度从版本控制中删除，或是该文件夹下某个受版本控制的文件丢失了。
- ![](/img-post/2020-12-13-svn-ln-introduction/10-33.jpg)：加号告诉你有一个文件或目录已经被调度加入版本控制。
- ![](/img-post/2020-12-13-svn-ln-introduction/10-34.jpg)：横条告诉你有一个文件或目录被版本控制系统所忽略。这个图标重载是可选的。
- ![](/img-post/2020-12-13-svn-ln-introduction/10-35.jpg)：这个图标说明文件和目录未被版本控制，但是也没有被忽略。这个图标重载是可选的。

## 纳入版本控制

新建文件 README.MD

在文件上点右键 --> Add

![](/img-post/2020-12-13-svn-ln-introduction/a-10-11.png)

添加后文件图标发生变化

![](/img-post/2020-12-13-svn-ln-introduction/a-10-12.png)

## 提交

使用 TortoiseSVN 可以提交具体某一个文件，或某一个目录下的所有改变。方法就是在想要提交的项目下点右键 --> SVN Commit...

![](/img-post/2020-12-13-svn-ln-introduction/a-10-13.png)

![](/img-post/2020-12-13-svn-ln-introduction/a-10-14.png)

日志内容如果不填，TortoiseSVN 会提交一个空字符串作为日志信息。

提交后显示信息如下：

![](/img-post/2020-12-13-svn-ln-introduction/a-10-15.png)

没有纳入版本控制的文件默认是不在提交范围内的，直接在新创建的文件上点右键只能看到 add 操作的选项。但在新创建的文件所在目录点右键选择 SVN commit...，可以看到如下界面：

![](/img-post/2020-12-13-svn-ln-introduction/a-10-16.png)

将文件 newText.txt 选中

![](/img-post/2020-12-13-svn-ln-introduction/a-10-17.png)

同样可以提交文件，TortoiseSVN 会帮我们自动将 newText.txt 纳入版本控制

![](/img-post/2020-12-13-svn-ln-introduction/a-10-18.png)

## 更新

在要更新的文件或目录上点右键→SVN Update

![](/img-post/2020-12-13-svn-ln-introduction/a-10-19.png)

![](/img-post/2020-12-13-svn-ln-introduction/a-10-20.png)

## 恢复历史版本

查看历史版本内容

1. 首先需要把对应版本库的匿名访问权限设置为 none：anon-access = none

2. 在要查看历史版本的文件上点右键→TortoiseSVN→Show log

   ![](/img-post/2020-12-13-svn-ln-introduction/a-10-21.png)

3. 在感兴趣的历史版本上点右键，可以与当前工作副本进行比较，或直接打开。

   ![](/img-post/2020-12-13-svn-ln-introduction/a-10-22.png)

   在要回复历史版本的文件上点右键→Update to revision

   ![](/img-post/2020-12-13-svn-ln-introduction/a-10-23.png)

   点击 Show log, 选择想要回到的版本即可

   ![](/img-post/2020-12-13-svn-ln-introduction/a-10-24.png)

   ![](/img-post/2020-12-13-svn-ln-introduction/a-10-25.png)

   ![](/img-post/2020-12-13-svn-ln-introduction/a-10-26.png)

## 恢复历史版本各个选项的区别

假设我们有许多个版本，版本号分别是 1-10

- Revert to this revision

  在 7 这里选择，那么 7 之后的 8, 9, 10 的操作都会被消除。

  适合永久恢复到以前的某个版本。

- Revert changes from this revision

  在 7 这里选择，那么7版本的修改将会被消除，8, 9, 10 的操作还在。

  同时选择 7, 8，那么 7 和 8 两个版本的所做的修改都会被消除，9, 10 的操作还在。

- Update item to revision

  本地更新到某历史版本，作为只读模式版本无法提交所作的更改，一般作查看历史版本用，无其它用途。

- Update to revision

  和 Revert to this revision 很像，都会融合你本地未提交的修改。它们 2 个的区别是：Revert to this revision 会把这个 revert 作为最新版本，而 Update to revision 不会。

  适合临时恢复到以前的某个版本。

  Update to revision 比 Revert to this revision 要常用得多。
---
layout:     post
title:      03.SVN入门笔记
subtitle:   使用命令行模式访问SVN服务器
date:       2020-12-13
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SVN
---

# 03.SVN入门笔记——使用命令行模式访问 SVN 服务器

## 检出

1. 首先进入自己的工作目录，例如：D:\DevWorkSpace\SVNSpace

2. 运行 svn checkout 命令，命令格式如下：

   ```shell
   svn checkout svn://SVN 服务器主机地址/具体仓库目录 保存检出内容的目录
   # 举例
   svn checkout svn://192.168.25.136/ERP ERP
   ```

   运行结果：Checked out revision 0.

   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-21.png)

3. 工作副本

   运行 checkout 命令后进入 ERP 目录，看到里面什么都没有。真的什么都没有吗？不是的。检出命令会在这一目录下创建一个隐藏目录 .svn，用来保存与服务器交互的重要信息，其中包括从服务器端取回的最新版本信息、文件状态、更新时间等。
   
   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-22.png)
   
   SVN 正是以此为依据判断当前目录中文件的状态。所以这个隐藏目录**千万不要删除或修改其中的内容**——完全无视它的存在吧。如果服务器端保存的文件可以视为一个“正本”，那么每个开发人员检出到本地目录的文件可以视为“副本”，通常称为工作副本。

## 提交

进入 D:\DevWorkSpace\SVNSpace\ERP 目录

创建一个文件 test.txt

执行 svn commit 命令，运行结果是：

```
D:\DevWorkSpace\SVNSpace\ERP>svn commit test.txt
svn: E200009: Commit failed (details follow):
svn: E200009: 'D:\DevWorkSpace\SVNSpace\ERP\test.txt' is not under version control
```

说明一个文件必须纳入版本控制才可以提交到服务器端。

执行 svn add 命令，将 test.txt 纳入版本控制

```
D:\DevWorkSpace\SVNSpace\ERP>svn add test.txt
A         test.txt
```

再次执行 svn commit 命令

```
D:\DevWorkSpace\SVNSpace\ERP>svn commit test.txt
svn: E205007: Commit failed (details follow):
svn: E205007: Could not use external editor to fetch log message; consider setting the $SVN_EDITOR environment variable or using the --message (-m) or --file (-F) options
svn: E205007: None of the environment variables SVN_EDITOR, VISUAL or EDITOR are set, and no 'editor-cmd' run-time configuration option was found
```

此时要求附加日志信息

使用-m 参数附加日志信息

```
D:\DevWorkSpace\SVNSpace\ERP>svn commit -m "My first commit" test.txt
svn: E170001: Commit failed (details follow):
svn: E170001: Authorization failed
```

原因是没有权限

暂时先开启匿名访问权限

1. 进入对应的版本库目录下的 conf 目录：D:\DevRepository\Subversion\ERP\conf
2. 打开 svnserve.conf
3. 将第 19 行的# anon-access = read 改为 anon-access = write，也就是去掉“# ”，将 read 改为 write。注意前面不要留空格，一定要顶格写。
4. 不需要重启 SVN 服务，甚至命令行窗口都不需要重新打开。

![](/img-post/2020-12-13-svn-ln-introduction/a-02-23.png)

重新执行提交命令

```
D:\DevWorkSpace\SVNSpace\ERP>svn commit -m "My first commit" test.txt
Adding         test.txt
Transmitting file data .done
Committing transaction...
Committed revision 1.
```

说明提交成功了。

其实 svn commit 命令最后可以不指定具体文件，此时表示提交当前工作副本中的所有修改。

## 更新

将服务器端文件检出到一个新的目录，模拟另外一个终端

```
D:\DevWorkSpace\SVNSpace>svn checkout svn://192.168.25.136/ERP TomERP
A    TomERP\test.txt
Checked out revision 1.
```

回到 ERP 目录，对 test.txt 文件修改后提交。

进入 TomERP 目录

执行 svn update 命令

```
D:\DevWorkSpace\SVNSpace\TomERP>svn update
Updating '.':
U    test.txt
Updated to revision 2.
```

这样我们就可以在 TomERP 目录下看到 ERP 目录下提交的修改。

更新和检出的相同点和不同点分别是什么？

|          | 检出                                   | 更新                         |
| -------- | -------------------------------------- | ---------------------------- |
| 相同点   | 从服务器端下载最新内容                 |                              |
| 不同点 1 | 下载整个项目                           | 下载与本地工作副本不同的内容 |
| 不同点 2 | 创建 .svn 目录，使检出目录成为工作副本 | 依赖 .svn 目录               |
| 不同点 3 | 只能操作 1 次                          | 可以操作多次                 |

## 工作副本中文件的几种状态

### 没有修改，现行版本

本档案在工作目录中没有被修改，而且自当前版本之后，其他终端也没有任何该文件的修改被提交到服务器，即当前工作副本的版本和服务器端最新版本是一致的。对它执行 svn commit 和 svn update 都不会发生任何事。

### 本地修改, 现行版本

这个文件被修改过，但这个修改还没有提交到服务器，而且自当前版本之后，其他终端也没有任何该文件的修改被提交到服务器，所以当前工作副本的版本和服务器端最新版本仍然是一致的。由于有尚未送交回去的本地修改，所以对它的 svn commit 会成功提交你的修改，而 svn update 则不会做任何事。

### 没有修改，过时版本

这个文件没有修改，但是版本库中有其他终端提交的修改。此时当前工作副本的版本比服务器端的版本落后了，我们称之为“过时”。对当前文件的 svn commit 不会发生任何事，而 svn update 会让工作目录中的文件更新至最新版本。

### 本地修改，过时版本

服务器端存在没有更新到本地的修改，导致当前版本过时。如果这个文件在本地有未提交的修改，则无法提交，对它执行 svn commit 会产生“out-of-date”错误。

此时应该先尝试更新本地文件。更新时 SVN 会尝试将服务器端的更新与本地文件进行合并，合并的结果有两种可能：一个是服务器端和本地修改位于文件的不同位置，合并成功；另一个是服务器端的修改正好和本地修改位于同一个位置，发生冲突。

## 将工作副本整体恢复到某一个历史版本

假设当前版本为 12，想要取回版本 9

执行 svn update 命令

格式：

```shell
svn update --revision 想要取回的版本号
```

举例：

```shell
svn update --revision 1
```

运行结果：

```
D:\DevWorkSpace\SVNSpace\TomERP>svn update --revision 1
Updating '.':
U    test.txt
Updated to revision 1.
```

这里需要注意的是，SVN 版本号并不是对某一个文件进行编号，而是对应整个版本库总体状态的一个“快照”，取回某个版本不是取回版本号对应的某个文件，而是整个项目的一个快照。

![](/img-post/2020-12-13-svn-ln-introduction/02-17.jpg)

## 将某个文件恢复到某个版本中的状态，同时不涉及其他文件

假设想要取回 pp.txt 在版本 10 时的状态

执行 svn update 命令

```shell
#格式
svn update 文件名 --revision 想要取回的版本号
#举例
svn update pp.txt –revision 10
```

运行结果：

```
Updating 'pp.txt':
U pp.txt
Updated to revision 10.
```

综合这两个例子，我们可以认为版本号和文件名构成了一个横纵坐标系，通过文件路径和版本号定位其在某一个时刻的状态。

![](/img-post/2020-12-13-svn-ln-introduction/02-18.png)
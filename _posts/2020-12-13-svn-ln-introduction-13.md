---
layout:     post
title:      11.SVN入门笔记
subtitle:   使用TortoiseSVN解决冲突
date:       2020-12-13
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SVN
---
# 11.SVN入门笔记——使用TortoiseSVN解决冲突

文件发生冲突时的状态：

![](/img-post/2020-12-13-svn-ln-introduction/a-11-01.png)

在冲突的文件上点右键→Edit Conflicts

![](/img-post/2020-12-13-svn-ln-introduction/a-11-02.png)

![](/img-post/2020-12-13-svn-ln-introduction/a-11-03.png)

在冲突行点右键

![](/img-post/2020-12-13-svn-ln-introduction/a-11-04.png)

可以选择四种操作：

1. 使用选中的这段代码
2. 使用选中的这段代码所在文件的全部代码
3. 把我的代码放在他们的代码前面
4. 把他们的代码放在我的代码前面



在冲突解决后，直接保存——这时 TortoiseSVN 自动弹出如下确认界面。如果没有问题的话，直接选择“Mark as resolved”。

![](/img-post/2020-12-13-svn-ln-introduction/a-11-05.png)

文件变为红色叹号标志，自动生成的三个文件被删除。提交修改即可。
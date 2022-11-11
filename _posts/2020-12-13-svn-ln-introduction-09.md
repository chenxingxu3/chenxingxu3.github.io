---
layout:     post
title:      08.SVN入门笔记
subtitle:   在Eclipse中安装SVN客户端插件
date:       2020-12-13
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SVN
---
# 08.SVN入门笔记——在 Eclipse 中安装 SVN 客户端插件

## Eclipse 插件应用市场

在 Eclipse 中访问 Eclipse Marketplace Client 可以搜索 Subversion，下载插件，按提示安装即可。

Help --> Eclipse Marketplace

![](/img-post/2020-12-13-svn-ln-introduction/a-08-01.png)

搜索 Subversive，点击 Install

![](/img-post/2020-12-13-svn-ln-introduction/a-08-02.png)

保持默认选项，点击 Confirm

![](/img-post/2020-12-13-svn-ln-introduction/a-08-03.png)

出现此提示框，点击 Yes

![](/img-post/2020-12-13-svn-ln-introduction/a-08-04.png)

选中 “I accept the terms of the license agreement”，点击 Finish

![](/img-post/2020-12-13-svn-ln-introduction/a-08-05.png)

Eclipse 会开始安装，耐心等待即可

![](/img-post/2020-12-13-svn-ln-introduction/a-08-06.png)

安装完成后，会弹出此提示框要求重启 Eclipse，点击 Yes

![](/img-post/2020-12-13-svn-ln-introduction/a-08-07.png)

进入 Eclipse 依次打开 Window-->Preferences-->Team-->SVN，看到如下界面即说明 SVN 插件安装成功：

![](/img-post/2020-12-13-svn-ln-introduction/a-08-08.png)

## 使用压缩包

如果不能联网，可以使用已经安装好插件的 Eclipse 压缩包。

下载地址：

[https://gitee.com/zhuzhulu/java-dev-software](https://gitee.com/zhuzhulu/java-dev-software)

找到 eclipse_javaee_mars.1_with_svn_svnkit.7z 压缩包，下载后解压缩，运行，即可直接使用带有 svn 插件的 Eclipse.



## 创建资源库位置

切换到透视图 SVN Repository Exploring

![](/img-post/2020-12-13-svn-ln-introduction/a-08-09.png)

创建资源库位置(Repository Location)

![](/img-post/2020-12-13-svn-ln-introduction/a-08-10.png)

填写相关信息，点击 Finish

![](/img-post/2020-12-13-svn-ln-introduction/a-08-11.png)

### 安装 SVN Connector

此时我这里出现了这个错误提示：

![](/img-post/2020-12-13-svn-ln-introduction/a-08-12.png)

原因是没有安装 SVN Connector。

依次打开 Window-->Preferences-->Team-->SVN, 进入 SVN Connector 标签页，点击 Get Connectors: 

![](/img-post/2020-12-13-svn-ln-introduction/a-08-21.png)

此时会让你选择一个 SVN Connector，这里我也不清楚这三个有什么区别，不过 Native JavaHL 有两个版本，对应不同的 SVN 版本，选错了应该会有麻烦，所以我就直接选了 SVN Kit.

![](/img-post/2020-12-13-svn-ln-introduction/a-08-13.png)

保持默认选项，点击 Next

![](/img-post/2020-12-13-svn-ln-introduction/a-08-14.png)

点击 Next

![](/img-post/2020-12-13-svn-ln-introduction/a-08-15.png)

选中“I accept the terms of the license agreement”，点击 Finish

![](/img-post/2020-12-13-svn-ln-introduction/a-08-16.png)

此时 Eclipse 会安装插件，耐心等待即可

![](/img-post/2020-12-13-svn-ln-introduction/a-08-17.png)

这里会出现安全警告，安装的插件包含未签名的内容，这里直接点 OK 即可

![](/img-post/2020-12-13-svn-ln-introduction/a-08-18.png)

要求重启 Eclipse, 点击 Yes

![](/img-post/2020-12-13-svn-ln-introduction/a-08-19.png)

重启后，SVN Repository Exploring 可以正常使用了

![](/img-post/2020-12-13-svn-ln-introduction/a-08-20.png)

### 查看版本库

此时可以查看版本库中的文件及目录结构

![](/img-post/2020-12-13-svn-ln-introduction/a-08-22.png)

补充：如何确定版本库地址？

![](/img-post/2020-12-13-svn-ln-introduction/02-23.jpg)



## 检出

进入 Eclipse, 在 Project Explorer 中点击鼠标右键 Import --> Import...

![](/img-post/2020-12-13-svn-ln-introduction/a-08-32.png)

SVN --> Project from SVN, 点击 Next

![](/img-post/2020-12-13-svn-ln-introduction/a-08-33.png)

填写好相关信息，点击 Next

![](/img-post/2020-12-13-svn-ln-introduction/a-08-34.png)

如果没有特殊需要的话保持默认即可，点击 Finish

![](/img-post/2020-12-13-svn-ln-introduction/a-08-35.png)

SVN 资源库中已经存在 HelloWorld 这个项目了，所以选择“Find projects in the children of the selected resource”, 点击 Finish.

![](/img-post/2020-12-13-svn-ln-introduction/a-08-36.png)

因为 HelloWorld 本身是一个 Eclipse 工程，所以选择“Check out as a projects into workspace”，点击 Next.

![](/img-post/2020-12-13-svn-ln-introduction/a-08-37.png)

保持默认即可，点击 Finish

![](/img-post/2020-12-13-svn-ln-introduction/a-08-38.png)

项目从服务器检出后，会成为一个工作副本，根目录下会自动创建.svn 隐藏目录

## 提交

新创建文件后，文件图标上会以“?”标识，表示该文件尚未纳入版本控制。

![](/img-post/2020-12-13-svn-ln-introduction/a-08-39.png)

在新创建的文件上点右键-->Team-->Add to Version Control...，这样文件图标上会显示“+”(新版本显示的是一个类似时钟的图标)，表示当前文件已纳入版本控制，但还未提交至服务器。

![](/img-post/2020-12-13-svn-ln-introduction/a-08-40.png)

![](/img-post/2020-12-13-svn-ln-introduction/a-08-41.png)

![](/img-post/2020-12-13-svn-ln-introduction/a-08-42.png)

在要提交的文件上点右键-->Team-->Commit...会提交文件，在弹出的对话框中可以不填写日志。文件提交后，图标会变为“金色的圆柱体”表示当前文件的版本和服务器端一致。

![](/img-post/2020-12-13-svn-ln-introduction/a-08-43.png)

![](/img-post/2020-12-13-svn-ln-introduction/a-08-44.png)

![](/img-post/2020-12-13-svn-ln-introduction/a-08-45.png)

文件修改后图标会变为“*”(新版本是在文件名前增加了一个 > 符号)，表示当前文件或目录包含未提交的修改。

![](/img-post/2020-12-13-svn-ln-introduction/a-08-46.png)

## 更新

更新整个项目时可以在项目上点右键-->Team-->Update

![](/img-post/2020-12-13-svn-ln-introduction/a-08-48.png)

更新某个具体的文件时，可以在文件上点右键-->Team-->Update

![](/img-post/2020-12-13-svn-ln-introduction/a-08-47.png)

## 共享项目

在 Eclipse 中创建的新项目想要发布到 SVN 服务器端，可以通过“共享”项目实现

在项目上点右键-->Team-->Share Project...-->选择一种版本控制工具

![](/img-post/2020-12-13-svn-ln-introduction/a-08-24.png)

![](/img-post/2020-12-13-svn-ln-introduction/a-08-25.png)

选择一个资源库位置。如果是初次使用，资源库位置应该是空的，这个时候应该选择 “Create a new repository location”来创建一个新的资源库位置。如果资源库位置列表中已经有了想要上传代码的资源库位置，选择“Use existing repository location”然后选中对应的资源库位置即可。

![](/img-post/2020-12-13-svn-ln-introduction/a-08-26.png)

填写相关信息，点击 Next

![](/img-post/2020-12-13-svn-ln-introduction/a-08-27.png)

这里没有特殊需要，保持默认即可，点击 Next

![](/img-post/2020-12-13-svn-ln-introduction/a-08-28.png)

这一步是在 SVN 新建一个 project，这里自动生成了 commit comment，直接点 Finish 即可。也可以把自动生成的 commit comment 删了，自己重新写一个新的。

![](/img-post/2020-12-13-svn-ln-introduction/a-08-29.png)

在 SVN 中的 project 创建好了之后，就可以往里面上传代码了，填写好 commit comment 后，选择要上传的 文件夹 和 文件，点击 OK 即可。

![](/img-post/2020-12-13-svn-ln-introduction/a-08-30.png)

在 SVN Repository Exploring 视图中可以看到刚刚上传的 project.

![](/img-post/2020-12-13-svn-ln-introduction/a-08-31.png)

## 恢复历史版本

在需要回复的文件上点右键-->Team-->Show History

得到如下界面：

![](/img-post/2020-12-13-svn-ln-introduction/a-08-49.png)

![](/img-post/2020-12-13-svn-ln-introduction/a-08-50.png)

选择某一个历史记录点右键-->Get Content, 文件就会恢复到指定版本的状态，同时图标变为“*”(新版本是在文件名前增加了一个 > 符号)。

![](/img-post/2020-12-13-svn-ln-introduction/a-08-51.png)

![](/img-post/2020-12-13-svn-ln-introduction/a-08-52.png)

![](/img-post/2020-12-13-svn-ln-introduction/a-08-55.png)

获取历史记录时，如果出现如下错误提示

![](/img-post/2020-12-13-svn-ln-introduction/a-08-54.png)

可以通过将对应版本库中的 svnserve.conf 文件中的 anon-access 设置为 none 解决。

![](/img-post/2020-12-13-svn-ln-introduction/a-08-53.png)
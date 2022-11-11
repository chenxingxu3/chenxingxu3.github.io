---
layout:     post
title:      02.SVN入门笔记
subtitle:   VisualSVN-Server安装与配置
date:       2020-12-13
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SVN
---

# 02.SVN入门笔记——VisualSVN-Server 安装与配置

## 安装服务器端程序

1. 服务器端程序版本

   VisualSVN-Server 下载地址：

   [https://www.visualsvn.com/server/download/](https://www.visualsvn.com/server/download/)

   [https://gitee.com/zhuzhulu/java-dev-software](https://gitee.com/zhuzhulu/java-dev-software)

2. 双击运行 VisualSVN-Server-4.3.1-x64.msi，点击Next

   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-01.png)

   点击Next

   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-02.png)

   此页面保持默认选项即可，Next

   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-03.png)

   Location：VisualSVN-Server 的安装目录

   Repositories：SVN 仓库

   Server Port：服务端口号。为了避免冲突，最好设置为 8443

   Backups：备份文件存放的目录

   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-04.png)

   保持默认选项：使用 Subversion 授权，Next

   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-05.png)

   Next

   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-06.png)

   等待安装完成

   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-07.png)

   Finish

   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-08.png)

   可以在开始菜单中找到 VisualSVN Server Manager

   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-10.png)

   VisualSVN Server Manager 管理页面，安装成功

   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-09.png)

5. 安装程序会自动配置 Path 环境变量

4. 验证 Path 环境变量是否配置成功

   按住 Shift 键，点击鼠标右键，点击 在此处打开命令窗口

   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-12.png)
   
   在命令行输入：

   ```shell
svn --version
   ```
   
   看到如下信息就表示服务器端程序安装成功
   
   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-11.png)

## 配置版本库

- 为什么要配置版本库？

  Subversion 是将文件数据信息保存到版本库中进行管理的，为了满足用户的不同需求，Subversion 允许用户对版本库目录进行定制。

- 在一个非中文无空格目录下创建一个文件夹，作为版本库的根目录。

  例如：D:\DevRepository\Subversion

- 在版本库根目录下创建与具体项目对应的子目录

  这样做的目的是使一个 SVN 服务器能够同时管理多个项目，而不是为每一个项目搭建一个 SVN 服务器——这显然太浪费资源了。

  例如：

  D:\DevRepository\Subversion\CRM D:\DevRepository\Subversion\ERP D:\DevRepository\Subversion\OA

- 创建版本库命令格式

  | 主命令   | 子命令 | 参数 1   |
  | -------- | ------ | -------- |
  | svnadmin | create | 仓库路径 |
  
  举例：
  
  ```shell
  svnadmin create D:\DevRepository\Subversion\CRM
  svnadmin create D:\DevRepository\Subversion\ERP
  svnadmin create D:\DevRepository\Subversion\OA
  ```
  
  ![](/img-post/2020-12-13-svn-ln-introduction/a-02-13.png)
  
- 版本库目录结构

  版本库创建成功后会在指定目录下产生如下的目录结构

  ![](/img-post/2020-12-13-svn-ln-introduction/02-10.png)

## 启动服务器端程序

SVN 服务器必须处于运行状态才能响应客户端请求，帮助我们管理项目文件。所以我们必须将 SVN 服务器启动起来。启动 SVN 服务器有两种方法，一个是命令行方式，一个是注册 Windows 服务。

### 命令行方式

命令格式：

| 主命令   | 参数 1          | 参数 2              | 参数 3                      |
| -------- | --------------- | ------------------- | --------------------------- |
| svnserve | -d 表示后台执行 | -r 表示版本库根目录 | D:\DevRepository\Subversion |

举例

```shell
svnserve -d -r D:\DevRepository\Subversion
```

![](/img-post/2020-12-13-svn-ln-introduction/a-02-14.png)

验证服务是否启动：

SVN 服务监听 3690 端口，打开一个新的 cmd 窗口 （命令窗口），使用 netstat -an 命令查看 3690 端口是否被监听。

![](/img-post/2020-12-13-svn-ln-introduction/a-02-15.png)

命令行方式的缺陷是：只要运行服务器端程序的命令行窗口关闭，服务就停止了，很不方便，而且每次开机都需要手动启动。

### 注册 Windows 服务

将SVN 服务端程序注册为 Windows 服务，就可以让 SVN 服务随系统一起启动，克服了命令行方式的不足。注册 Windows 服务需要利用 XP、2000 以上系统自带工具 Service Control，执行文件是 sc.exe，注意这个命令不是 SVN 的命令。

命令格式：

| 主命令 | 子命令 | 参数 1 | 参数 2                                                       | 参数 3                  | 参数 4                           |
| ------ | ------ | ------ | ------------------------------------------------------------ | ----------------------- | -------------------------------- |
| sc     | create | 服务名 | binpath= “运行服务所需要的二进制文件路径以及运行该二进制文件的命令行参数” | start= auto表示自动启动 | depend= Tcpip表示依赖 Tcpip 协议 |

注意：在这个命令中，等号左边都没有空格，右边都有一个空格！

binpath 组成结构说明：

| svnserve.exe 路径             | svnserve 命令参数 1                    | svnserve 命令参数 2 | svnserve 命令参数 3 |
| ----------------------------- | -------------------------------------- | ------------------- | ------------------- |
| SVN 安装目录\bin\svnserve.exe | --service表示以服务方式启动 Subversion | -r表示版本库根目录  | 版本库目录          |

关于“版本库目录”：

| 单仓库 | 指定与具体项目对应的仓库目录 | 例如：D:\DevRepository\Subversion\CRM | 只能为一个项目服务 |
| ------ | ---------------------------- | ------------------------------------- | ------------------ |
| 多仓库 | 指定版本库的根目录           | 例如：D:\DevRepository\Subversion     | 可以为多个项目服务 |

最终命令举例：

```shell
sc create MySVNService binpath= "C:\Program Files\VisualSVN Server\bin\svnserve.exe --service -r D:\DevRepository\Subversion" start= auto depend= Tcpip
```

注意：在这个命令中，等号左边都没有空格，右边都有一个空格！

在 Win7 及以上系统中，运行该命令需要管理员权限，否则会得到如下错误提示：

![](/img-post/2020-12-13-svn-ln-introduction/a-02-16.png)

解决的办法是以管理员身份运行 cmd 命令行窗口即可：

![](/img-post/2020-12-13-svn-ln-introduction/a-02-17.png)

![](/img-post/2020-12-13-svn-ln-introduction/a-02-18.png)

打开控制面板 --> 系统和安全 --> 管理工具 --> 服务

![](/img-post/2020-12-13-svn-ln-introduction/a-02-19.png)

此时查看当前系统中的服务，可以看到我们刚刚创建的服务，但此时它还没有启动，如果创建失败，需检查 sc 命令是否正确：

![](/img-post/2020-12-13-svn-ln-introduction/02-15.jpg)

启动此服务，启动服务的命令格式如下：

```shell
sc start 服务名
```

举例：

```shell
sc start MySVNService
```

![](/img-post/2020-12-13-svn-ln-introduction/a-02-20.png)

![](/img-post/2020-12-13-svn-ln-introduction/02-16.jpg)

打开命令行窗口运行 netstat -an 查看 3690 端口是否被监听。如果启动失败，那很有可能是 binpath 中的内容有错误，此时只能将已经创建的服务删除，重新创建。

删除服务之前，最好先停止服务。停止服务的命令格式如下：

```shell
sc stop 服务名
#举例
sc stop MySVNService
```

删除服务的命令格式如下：

```shell
sc delete 服务名
# 举例
sc delete MySVNService
```

删除、启动、停止服务同样需要管理员权限。
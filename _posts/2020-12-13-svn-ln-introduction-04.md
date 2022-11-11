---
layout:     post
title:      04.SVN入门笔记
subtitle:   单一版本库权限配置
date:       2020-12-13
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SVN
---
# 04.SVN入门笔记——单一版本库权限配置

## 匿名访问

前已述及。

## 授权访问

要设置授权访问就需要创建用户，并为用户设定权限。

打开授权访问的配置：

1. 打开 D:\DevRepository\Subversion\ERP\conf\svnserve.conf

2. 将第 19 行 anon-access = write 注释掉：# anon-access = write

   表明该版本库不接受匿名访问。

3. 将第 20 行# auth-access = write 注释打开：auth-access = write 表明该版本库使用授权访问。

4. 将第 27 行注释打开：password-db = passwd

   表明使用同目录下的 passwd 文件保存用户信息。

5. 将第 36 行注释打开：authz-db = authz

   表明使用同目录下的 authz 文件保存权限信息。

   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-24.png)

6. 打开 passwd 文件创建用户：

   ```shell
   userWrite01 = 123456
   userWrite02 = 123456
   userRead = 123456
   userOther = 123456
   ```

   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-25.png)

7. 打开 authz 文件：#后面注释的是例子

   1. 创建用户组：

      ```
      [groups]
      # harry_and_sally = harry,sally
      # harry_sally_and_joe = harry,sally,&joe 
      canWrite = userWrite01,userWrite02
      ```
   ```
   
   
   ```
   
2. 指定路径，给用户和用户组授权：
   
      ```
      # [/foo/bar]
      # harry = rw
      # &joe = r
      # * =  屏蔽那些未设定的用户，让它们没有任何权限
      [/]
      @canWrite = rw 
      userRead = r
      * =
   ```
   
   ![](/img-post/2020-12-13-svn-ln-introduction/a-02-26.png)
   
3. 权限的继承性：父目录设置的权限，对子目录同样有效——除非子目录进行了更为具体的设定。
   
      ```
      [/OA] 
      userOther = rw
      * =
   ```
   
   这个例子表示当前版本库下的 OA 目录只有 userOther 有读写权限，其它用户无任何权限。
   

## 访问示例

![](/img-post/2020-12-13-svn-ln-introduction/a-04-01.png)

初次访问，svn 默认使用的用户名为 Windows 系统当前登录的用户名，如果想切换成别的用户，直接回车即可，svn 会让你输入 Username 和对应的 Password.
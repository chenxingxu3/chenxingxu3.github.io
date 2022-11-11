---
layout:     post
title:      05.SVN入门笔记
subtitle:   多版本库共享配置
date:       2020-12-13
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SVN
---
# 05.SVN入门笔记——多版本库共享配置

在版本库根目录 D:\DevRepository\Subversion 下创建 commConf 目录

将未修改的 authz 和 passwd 文件拷贝到 commConf 目录下

修改需要设置权限的版本库的 svnserve.conf 文件

```
auth-access = write
password-db = ../../commConf/passwd
authz-db = ../../commConf/authz
```

![](/img-post/2020-12-13-svn-ln-introduction/a-05-01.png)

在 D:\DevRepository\Subversion\commConf\password 中创建用户

```
[users]
# harry = harryssecret
# sally = sallyssecret
userERP = 123456
userOA = 123456
userCRM = 123456
```

在 D:\DevRepository\Subversion\commConf\authz 中针对不同版本库为不同用户授予权限

```
# [repository:/baz/fuz]
# @harry_and_sally = rw
# * = r

[ERP:/]
userERP = rw
* =

[OA:/]
userOA = rw
* =

[CRM:/]
userCRM = rw
* =
```

![](/img-post/2020-12-13-svn-ln-introduction/a-05-02.png)


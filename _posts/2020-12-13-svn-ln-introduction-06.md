---
layout:     post
title:      05.01.SVN入门笔记
subtitle:   更换svn的用户名和密码
date:       2020-12-13
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SVN
---
# 05.01.SVN入门笔记——更换svn的用户名和密码

## 临时更换

在所有命令下强制加上 --username 和 --password 选项。

```
svn checkout svn://192.168.25.136/ERP JerryERP --username userERP --password 123456
```

![](/img-post/2020-12-13-svn-ln-introduction/a-05.01-01.png)

## 永久更换

进入 C:\Users\你的 Windows 当前登录用户名\Application Data\Roaming\Subversion\auth 目录，删除auth/

```
c:
cd /
cd  Users\User\AppData\Roaming\Subversion
RMDIR /Q/S auth\
mkdir auth
```

再次执行命令可以重新指定用户名和密码了。

```
D:\DevWorkSpace\SVNSpace>svn checkout svn://192.168.25.136/ERP JerryERP
Authentication realm: <svn://192.168.25.136:3690> 39db0184-d422-af4a-8388-e037c5abf675
Password for 'User':
Authentication realm: <svn://192.168.25.136:3690> 39db0184-d422-af4a-8388-e037c5abf675
Username: userERP
Password for 'userERP': ******
A    JerryERP\test.txt
Checked out revision 2.
```

![](/img-post/2020-12-13-svn-ln-introduction/a-05.01-02.png)


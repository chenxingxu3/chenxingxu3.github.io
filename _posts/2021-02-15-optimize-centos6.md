---
layout:     post
title:      CentOS 6 study notes (2)
subtitle:   Optimizing CentOS 6
date:       2021-02-15
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - CentOS
---
# CentOS 6学习笔记（二）--优化CentOS以适应开发环境 

### 修改网络配置

```bash
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```

```
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=no
BOOTPROTO=dhcp
```

```bash
service network restart
ip addr
```

### 关闭防火墙

```bash
service iptables stop
service ip6tables stop
service iptables status
service ip6tables status

chkconfig iptables off
chkconfig ip6tables off
vi /etc/selinux/config
```

```
SELINUX=disabled
```

### 配置 DNS 服务器

```bash
vi /etc/resolv.conf
```

```
nameserver 114.114.114.114
```

### 配置 repo 文件

新建文本文件，拷贝下列内容，重命名为 CentOS6-Base-163.repo，上传到  /usr/local 目录下。

```
[base]
name=CentOS-$releasever - Base - 163.com
baseurl=http://mirrors.163.com/centos/$releasever/os/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
gpgcheck=0
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-6

#released updates 
[updates]
name=CentOS-$releasever - Updates - 163.com
baseurl=http://mirrors.163.com/centos/$releasever/updates/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
gpgcheck=0
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-6

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - 163.com
baseurl=http://mirrors.163.com/centos/$releasever/extras/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
gpgcheck=0
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-6

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - 163.com
baseurl=http://mirrors.163.com/centos/$releasever/centosplus/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
gpgcheck=0
enabled=0
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-6

#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib - 163.com
baseurl=http://mirrors.163.com/centos/$releasever/contrib/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib
gpgcheck=0
enabled=0
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-6
```

将 repo 文件拷贝到 /usr/local 目录下

```bash
cd /etc/yum.repos.d/
rm -rf *
cp /usr/local/CentOS6-Base-163.repo .
```

### 配置 yum

```bash
yum clean all
yum makecache
#测试yum工作是否正常，随便安装一个东西试试
yum install telnet
```

### 配置系统时间

安装 ntpdate

```bash
yum install -y ntp
ntpdate 0.pool.ntp.org
#查看系统当前时间是否正确
date
```

```bash
#添加NTP
chkconfig --add ntpd
#开机自启动NTP
chkconfig ntpd on
```

修改 NTP 配置文件

```bash
vi /etc/ntp.conf
```

```
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 0.pool.ntp.org iburst
server 1.pool.ntp.org iburst
server 2.pool.ntp.org iburst
server 3.pool.ntp.org iburst
```

### 安装 JDK 1.7

将 jdk-7u65-linux-i586.rpm 上传到  /usr/local 目录下。

```bash
cd /usr/local
rpm -ivh jdk-7u65-linux-i586.rpm
```
配置 JDK 相关的环境变量

```bash
vi ~/.bashrc
```

```
export JAVA_HOME=/usr/java/latest
export PATH=$PATH:$JAVA_HOME/bin
```

```bash
source ~/.bashrc
#测试java是否安装成功
java -version
```

### 安装 gcc-c++

```bash
yum install gcc-c++
```

### 删除 70-persistent-net.rules

为了防止某个设备名称和mac地址不对应的现象，将 70-persistent-net.rules 这个文件删除

```bash
rm -f /etc/udev/rules.d/70-persistent-net.rules
```

### 关机或重启

```bash
#立即关机
shutdown -h now
#重启
reboot
#强制重启
reboot –f
```


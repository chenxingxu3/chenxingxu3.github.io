---
layout:     post
title:      Maven丨使用IDEA将本地的ojdbc14.Jar包添加到个人仓库中
subtitle:   Maven丨使用IDEA将本地的ojdbc14.Jar包添加到个人仓库中
date:       2020-05-18
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Maven丨使用IDEA将本地的ojdbc14.Jar包添加到个人仓库中

在学习 Oracle 数据库的过程中，使用 IntelliJ IDEA 创建了一个 Maven 工程，但是在 pom.xml 中添加 ojdbc14 依赖后，提示如下错误：

Could not find artifact com.oracle.jdbc:ojdbc14:pom:10.2.0.1.0 in central (https://repo.maven.apache.org/maven2)

原因是由于 ojdbc 是收费的，所以无法从远程仓库中获取，获取 ojdbc14 需要从本地类库获取。知道了原因所在，那么解决此问题的思路就有了。

解决方法：

1. 下载 ojdbc14.jar 包
2. 使用 IDEA 将本地的 Jar 包添加到个人的 Maven 仓库中
3. 在 pom.xml 文件中添加依赖

**1、下载 ojdbc14.jar 包**

https://306t.com/file/23243704-443891502
访问密码 665159

**2、使用 IDEA 将本地的 Jar 包添加到个人的 Maven 仓库中**

（1）点击 IDEA 右侧的 Maven 按钮

![Snipaste_2020-05-18_08-55-04](/img-post/2020-05-18-oracle-special-01/Snipaste_2020-05-18_08-55-04.jpg)

（2）点击 Execute Maven Goal 按钮

![Snipaste_2020-05-18_08-57-42](/img-post/2020-05-18-oracle-special-01/Snipaste_2020-05-18_08-57-42.jpg)

（3）在出现的 Run Anything 中 写入 Maven Goal

![Snipaste_2020-05-18_09-01-15](/img-post/2020-05-18-oracle-special-01/Snipaste_2020-05-18_09-01-15.jpg)

maven goal 的格式如下（符号："<>" 运行时去掉, -D 前面一定要有空格）：

```xml
install:install-file -Dfile=<Jar包的地址> 
                     -DgroupId=<Jar包的GroupId> 
                     -DartifactId=<Jar包的引用名称> 
                     -Dversion=<Jar包的版本> 
                     -Dpackaging=<Jar的打包方式>
```

我写的例子如下，仅供参考：

```xml
mvn install:install-file -Dfile=D:/jar/ojdbc14.jar -DgroupId=com.oracle.jdbc -DartifactId=ojdbc14 -Dversion=10.2.0.1.0 -Dpackaging=jar
```

填写完成后回车即可执行。当出现如下图的信息，说明部署安装成功：

![Snipaste_2020-05-18_09-04-37](/img-post/2020-05-18-oracle-special-01/Snipaste_2020-05-18_09-04-37.jpg)

**3、在Pom.xml文件中添加依赖**

我写的例子如下，仅供参考：

```xml
<dependency>
            <groupId>com.oracle.jdbc</groupId>
            <artifactId>ojdbc14</artifactId>
            <version>10.2.0.1.0</version>
            <scope>runtime</scope>
</dependency>
```


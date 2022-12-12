---
layout:     post
title:      Maven丨Use IDEA to add the local ojdbc14.Jar package to your personal repository
subtitle:   Maven丨Use IDEA to add the local ojdbc14.Jar package to your personal repository
date:       2020-05-18
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Maven丨Use IDEA to add the local ojdbc14.Jar package to your personal repository

In the process of learning Oracle database, I created a Maven project using IntelliJ IDEA, but after adding the ojdbc14 dependency in pom.xml, the following error was prompted:

Could not find artifact com.oracle.jdbc:ojdbc14:pom:10.2.0.1.0 in central (https://repo.maven.apache.org/maven2)

The reason is that since ojdbc is paid, it cannot be fetched from the remote repository, and getting ojdbc14 requires getting it from the local class library. Once you know what the reason is, then you have an idea to solve this problem.

Solution.

1. download the ojdbc14.jar package
2. Use IDEA to add the local Jar package to your personal Maven repository
3. add the dependencies in the pom.xml file

**1、Download ojdbc14.jar package**

https://306t.com/file/23243704-443891502
Access Password 665159

**2、Use IDEA to add local Jar packages to your personal Maven repository**

（1）Click on the Maven button on the right side of IDEA

![Snipaste_2020-05-18_08-55-04](/img-post/2020-05-18-oracle-special-01/Snipaste_2020-05-18_08-55-04.jpg)

（2）Click the Execute Maven Goal button

![Snipaste_2020-05-18_08-57-42](/img-post/2020-05-18-oracle-special-01/Snipaste_2020-05-18_08-57-42.jpg)

（3）Write Maven Goal in the Run Anything that appears

![Snipaste_2020-05-18_09-01-15](/img-post/2020-05-18-oracle-special-01/Snipaste_2020-05-18_09-01-15.jpg)

The format of the maven goal is as follows（The symbol: "<>" is removed at runtime, -D must be preceded by a space）：

```xml
install:install-file -Dfile=<Address of the Jar package> 
                     -DgroupId=<GroupId of the Jar package> 
                     -DartifactId=<Reference name of Jar package> 
                     -Dversion=<Versions of the Jar package> 
                     -Dpackaging=<Jar's packaging method>
```

I have written the following example for reference only: 

```xml
mvn install:install-file -Dfile=D:/jar/ojdbc14.jar -DgroupId=com.oracle.jdbc -DartifactId=ojdbc14 -Dversion=10.2.0.1.0 -Dpackaging=jar
```

Fill in the fields and enter to execute. When the following message appears, the deployment is successfully installed: 

![Snipaste_2020-05-18_09-04-37](/img-post/2020-05-18-oracle-special-01/Snipaste_2020-05-18_09-04-37.jpg)

**3、Add dependencies in the Pom.xml file**

I have written the following example for reference only.

```xml
<dependency>
            <groupId>com.oracle.jdbc</groupId>
            <artifactId>ojdbc14</artifactId>
            <version>10.2.0.1.0</version>
            <scope>runtime</scope>
</dependency>
```


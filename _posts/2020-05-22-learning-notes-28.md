---
layout:     post
title:      Oracle数据库学习笔记（二十八）
subtitle:   在Java项目开发中使用ojdbc连接Oracle数据库
date:       2020-05-22
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---
# Oracle数据库学习笔记（二十八）--在Java项目开发中使用ojdbc连接Oracle数据库

# 选择正确的 Jar 包

Oracle 10g 选择使用 ojdbc14.jar

Oracle 11g 选择使用 ojdbc6.jar

# 将 Jar 包安装到本地 Maven 仓库中

具体方法参考我之前发的文章：

[Maven丨使用IDEA将本地的ojdbc14.Jar包添加到个人仓库中]: https://blog.csdn.net/gaoxiaokun4282/article/details/106186136

# 创建 Maven 工程

## 在IntelliJ IDEA 中创建 Maven 工程

![](/img-post/2020-05-22-learning-notes-28/01.jpg)

![](/img-post/2020-05-22-learning-notes-28/02.jpg)

![](/img-post/2020-05-22-learning-notes-28/03.jpg)

## pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>demo.yangxu</groupId>
    <artifactId>ojdbc_oracle_demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>com.oracle.jdbc</groupId>
            <artifactId>ojdbc14</artifactId>
            <version>10.2.0.1.0</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.10</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <!-- 修改为自己使用的 JDK 版本 -->
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

# 敲代码

这个案例我上传到了码云，供大家参考。

[ojdbc_oracle_demo]: https://gitee.com/telyfox/ojdbc_oracle_demo



## Java 测试类

### OracleDemo

所属包：java.yangxu.oracle

#### javaCallOracle()

```java
    @Test
    public void javaCallOracle() throws ClassNotFoundException, SQLException {
        //加载数据库驱动
        Class.forName("oracle.jdbc.driver.OracleDriver");
        //得到Connection连接
        Connection connection = DriverManager.getConnection("jdbc:oracle:thin:@192.168.63.128:1521:orcl", "xiaoming", "mima");
        //得到预编译的Statement对象
        PreparedStatement pstm = connection.prepareStatement("SELECT * FROM emp WHERE empno = ?");
        //给参数赋值
        pstm.setObject(1,7788);
        //执行数据库查询操作
        ResultSet rs = pstm.executeQuery();
        //输出结果
        while(rs.next()){
            System.out.println(rs.getString("ename"));
        }
        //释放资源
        rs.close();
        pstm.close();
        connection.close();

    }
```

测试结果：

![](/img-post/2020-05-22-learning-notes-28/04.jpg)

#### javaCallProcedure()

```java
    /**
     * java调用存储过程
     * {call <procedure-name>[(<arg1>,<arg2>, ...)]}调用存储过程使用
     * @throws ClassNotFoundException
     * @throws SQLException
     */
    @Test
    public void javaCallProcedure() throws ClassNotFoundException, SQLException {
        //加载数据库驱动
        Class.forName("oracle.jdbc.driver.OracleDriver");
        //得到Connection连接
        Connection connection = DriverManager.getConnection("jdbc:oracle:thin:@192.168.63.128:1521:orcl", "xiaoming", "mima");
        //得到预编译的Statement对象
        //p_yearsal(eno emp.empno%type, yearsal out number)
        CallableStatement pstm = connection.prepareCall("{call p_yearsal(?, ?)}");
        //给参数赋值
        pstm.setObject(1,7788);
        pstm.registerOutParameter(2, OracleTypes.NUMBER);
        //执行数据库查询操作
        pstm.execute();
        //输出结果
        System.out.println(pstm.getObject(2));
        //释放资源
        pstm.close();
        connection.close();
    }
```

测试结果：

![](/img-post/2020-05-22-learning-notes-28/05.jpg)

#### javaCallFunction()

```java
    /**
     * java调用存储函数
     * {?= call <procedure-name>[(<arg1>,<arg2>, ...)]}   调用存储函数使用
     * @throws ClassNotFoundException
     * @throws SQLException
     */
    @Test
    public void javaCallFunction() throws ClassNotFoundException, SQLException {
        //加载数据库驱动
        Class.forName("oracle.jdbc.driver.OracleDriver");
        //得到Connection连接
        Connection connection = DriverManager.getConnection("jdbc:oracle:thin:@192.168.63.128:1521:orcl", "xiaoming", "mima");
        //得到预编译的Statement对象
        //f_yearsal(eno emp.empno%type) return number
        CallableStatement pstm = connection.prepareCall("{?= call f_yearsal(?)}");
        //给参数赋值
        pstm.setObject(2,7788);
        pstm.registerOutParameter(1, OracleTypes.NUMBER);
        //执行数据库查询操作
        pstm.execute();
        //输出结果
        System.out.println(pstm.getObject(1));
        //释放资源
        pstm.close();
        connection.close();
    }
```

测试结果：

![](/img-post/2020-05-22-learning-notes-28/06.jpg)
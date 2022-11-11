---
layout:     post
title:      09.SVN入门笔记
subtitle:   在Eclipse中使用SVN客户端插件解决冲突
date:       2020-12-13
author:     java阳旭
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SVN
---
# 09.SVN入门笔记——在 Eclipse 中使用 SVN 客户端插件解决冲突

## 什么情况下会发生冲突

![](/img-post/2020-12-13-svn-ln-introduction/02-35.jpg)

1. 两个开发人员，Harry 和 Sally，分别从服务器端下载了文件 A。

2. Harry 修改之后，A 变成了 A’，Sally 修改之后，A 变成了 A’’。

3. Harry 先一步提交，使服务器端文件的版本也变成了 A’

4. Sally 本地的文件 A’’已经过时了，此时她已无法提交文件，服务器会要求她先进行一次更新操作。

5. 此时 Sally 的更新操作有两种可能： 

   1. Sally 所做的修改与 Harry 不是同一个位置，更新操作尝试合并文件成功。
   2. Sally 所做的修改与 Harry 恰好是同一个位置，更新操作尝试合并文件失败，发生冲突。

6. 发生冲突后，本地工作副本会发生如下变化：

   1. 文件 A 中的内容发生如下改变：

      ```java
      public static void main(String[] args) { 
      	System.out.println("Edit By Command!"); 
      	System.out.println("Edit By Command!");
      <<<<<<< .mine
      	System.out.println("Edit By Eclipse!");
      =======
      	System.out.println("Edit By Command!New Edit");
      >>>>>>> .r14
      	System.out.println("Edit By Command!"); 
      	System.out.println("Edit By Command!");
      }
      ```

      其中，从<<<<<<< .mine 到=======之间是发生冲突时本地副本的内容。从=======到>>>>>>> .r14 是发生冲突时服务器端的最新内容。注意这里 r 后面的数字是发生冲突时服务器端的版本号，有可能是任何整数值，r14 只是一个例子。

      同时文件图标变成一个“黄色的！”。

   2. 与冲突文件同目录下新增文件，扩展名为.mine，其内容是发生冲突时本地副本的文件内容。

   3. 与冲突文件同目录下新增文件，扩展名为.r 小版本号，例如 MyCRM.java.r13，其内容是冲突发生之前，服务器端的文件内容，可以作为解决冲突的参照。

   4. 与冲突文件同目录下新增文件，扩展名为.r 大版本号，例如 MyCRM.java.r14，其内容是冲突发生时，服务器端的文件内容。

## 解决冲突

发生冲突的提示：

![](/img-post/2020-12-13-svn-ln-introduction/a-09-01.png)

1. 在冲突文件上点右键→Team→Edit Conflicts...→出现如下界面

   ![](/img-post/2020-12-13-svn-ln-introduction/a-09-02.png)

   ![](/img-post/2020-12-13-svn-ln-introduction/a-09-03.png)

   以对比的方式将本地内容与冲突内容显示出来，其中左侧为本地内容，右侧为冲突内容。其中本地内容是可以修改的。

2. 根据需要和实际情况将本地内容更正，这个过程很可能需要牵涉冲突的两位开发人员进行必要的沟通。

   ![](/img-post/2020-12-13-svn-ln-introduction/a-09-04.png)

   

3. 在冲突文件上点右键→Team→Mark as Merged

   ![](/img-post/2020-12-13-svn-ln-introduction/a-09-05.png)

   此时.mine 文件和.r 版本号文件都会被自动删除，冲突文件的图标变为“*”(新版本为在文件名前添加 > 符号)，表示可以提交。

4. 提交文件，文件图标变为“金色圆柱体”。

   ![](/img-post/2020-12-13-svn-ln-introduction/a-09-06.png)

   
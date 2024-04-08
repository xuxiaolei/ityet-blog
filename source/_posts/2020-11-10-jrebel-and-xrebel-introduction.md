---
layout: post
title: Jreble和Xrebel使用学习
categories: Java
description: Jreble和Xrebel使用学习
keywords:  Jreble,Xrebel
---
# 功能概括

JRebel：修改完代码，不想重启服务，就使想代码生效。
XRebel：请求中，各个部分代码性能监控。例如：方法执行时间，出现的异常，SQL执行时间，输出的Log，MQ执行时间等。

# 一、JRebel介绍

---

JRebel是一套JavaEE开发工具，可快速实现热部署，节省了大量重启时间，提高了个人开发效率。
JRebel是一款JAVA虚拟机插件，它使得JAVA程序员能在不进行重部署的情况下，即时看到代码的改变对一个应用程序带来的影响。JRebel使你能即时分别看到代码、类和资源的变化，你可以一个个地上传而不是一次性全部部署。当程序员在开发环境中对任何一个类或者资源作出修改的时候，这个变化会直接反应在部署好的应用程序上，从而跳过了构建和部署的过程，可以省去大量的部署用的时间。

JRebel是一款JVM插件，它使得Java代码修改后不用重启系统，立即生效。
IDEA上原生是不支持热部署的，一般更新了 Java 文件后要手动重启 Tomcat 服务器，才能生效，浪费时间浪费生命。
目前对于idea热部署最好的解决方案就是安装JRebel插件

## 已知的4种本地调试热部署方案比较

---

**第1种：**修改服务器配置，使得IDEA窗口失去焦点时，更新类和资源

菜单Run \-> EditConfiguration , 然后配置指定服务器下，右侧server标签下on frame deactivation = Update classes and resource。

优点：简单

缺点：基于JVM提供的热加载仅支持方法块内代码修改，只有debug模式下，并且是在idea失去焦点时才会出发热加载，相对加载速度缓慢

**第2种：**使用springloaded jar包

a. 下载jar包，github：[https://github.com/spring\-projects/spring\-loaded](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fspring-projects%2Fspring-loaded)

b. 启动应用时添加VM启动参数：\-javaagent:/home/lkqm/.m2/repository/org/springframework/springloaded/1.2.7.RELEASE/springloaded\-1.2.7.RELEASE.jar \-noverify

优点：对Spring系列框架支持好（不含Spring boot）, 支持 成员级别的修改（增删改方法、字段、注解），支持对枚举值集。

缺点：与优点相对

**第3种：**使用spring\-boot\-devtools提供的开发者工具

spring\-boot项目中引入如下依赖

org.springframework.bootspring\-boot\-devtools

优点：简单，支持Spring\-boot项目，支持成员级别的修改热部署。

缺点：只支持spring\-boot项目。

**第4种：**使用Jrebel插件实现热部署

在线安装：菜单File \-> Setting \-> Plugin, 点击右侧底部 Browse repositories, 弹出框顶部输入:JReble for Intellij， 选中安装即可。

优点：强大，对各类框架支持，并且提供IDE插件的方式。

**总结：**最后3种方法是基于类加载机制来实现热加载的，因此你修改完成代码后必须重新编译当前代码，才能触发热部署，Eclipse默认就支持了自动编译，而在Intellij IDEA中默认是关闭了自动编译的，可以按照如下2步设置开启：

IDEA开启项目自动编译，进入设置，Build,Execut, Deployment \-> Compiler 勾选中左侧的Build Project automatically

如第上述步骤自动编译无效请尝试下面，

IDEA开启项目运行时自动make, ctrl + shift + a搜索命令：registry \-> 勾选compiler.automake.allow.when.app.running

## JRebel远程热部署

通过instrument来修改ClassLoader和Reflection相关的类，加入自己的入口（比如监测类文件或者资源改变），因为我们无法修改已经装载的类，只能通过产生新的类来实现，因此在装载类A的时候，实际上我们是产生了一个A1给应用使用。

![](https://upload-images.jianshu.io/upload_images/2331003-cfaf89a8b236dbc9.png?imageMogr2/auto-orient/strip|imageView2/2/w/550/format/webp)

# 二、Xrebel介绍

XRebel 是不间断运行在 web 应用的交互式分析器。可以看到网页上的每一个操作在前端以及服务端、数据库、网络传输都花费多少时间，当发现问题会在浏览器中显示警告信息。是调优程序、追踪性能问题的一大利器。

## 监视应用程序性能

**XRebel** 监视应用程序性能。 此视图显示所有正在处理的请求的完整细分。 根据请求的持续时间，以降序显示请求。 可以扩展显示的请求，以查看在每个特定分支或方法中花费的时间。 IO查询也显示在此视图中。

按下**XRebel** 工具栏 上的 “ **应用程序性能”** 按钮（最 **上方）** 以打开此视图。

按 请求详细信息前面 的 `+` 和 `-` 按钮以展开或折叠其他信息。 使用 标题中 的“ **排序请求”** 功能进一步对所有结果进行排序。

[![../_images/app.png](https://manuals.jrebel.com/xrebel/_images/app.png)](https://manuals.jrebel.com/xrebel/_images/app.png)

这是此视图的视觉细分：

[![../_images/xrebel-app-view-14p.png](https://manuals.jrebel.com/xrebel/_images/xrebel-app-view-14p.png)](https://manuals.jrebel.com/xrebel/_images/xrebel-app-view-14p.png)

*   **在分支机构中花费的时间** –当前分支机构中花费的总时间百分比。

*   **请求名称** –在此期间正在执行的请求的全名。

*   **程序包名称** –当前请求所属的程序包名称的前两个元素。 将鼠标悬停在此名称上可以查看程序包的全名。 在以下所有行中，程序包名称均有效，直到显示新的程序包名称为止。

*   **过滤的方法** –按下可见的方法之间的区域（用 **脱字符号标记** ），以查看对时间的影响不大（小于1％）的过滤的方法。 显示这些过滤的方法以揭示两种基本方法之间的联系。

*   **方法花费的时间** –突出显示的百分比和绝对时间显示了在线显示的特定方法花费的时间。

# 参考资料：
1. jRebel：https://blog.csdn.net/liuzhigang828/article/details/72875190
2. xRebel：https://blog.csdn.net/anttu/article/details/77688057
3. 免费破解jRebel:https://blog.csdn.net/zhoukikoo/article/details/80644093
4. xRebel官网文档：https://manuals.jrebel.com/xrebel/use/index.html#application-performance
5. jRebel官网文档：https://manuals.jrebel.com/jrebel/index.html
---
layout: post
title: Intellij IDEA集成JProfiler性能分析
categories: Java
description: Intellij IDEA集成JProfiler性能分析
index_img: 
date: 2020-11-11 09:09:09
tags: [Intellij,IDEA,JProfiler,性能分析]
---

Jprofiler是用于分析J2EE软件性能瓶颈并准确定位到java类或者方法并有效解决性能问题的主流工具，它通常需要与性能测试工具如LoadRunner配合使用，因为往往只用当系统处于压力状态下才能反映出性能问题。

### 一. 插件&软件安装

1. 插件在IDEA上直接下载Preferences–>plugins–>Browse repositories。
![](/images/posts/other/1604971412833.jpg)

2. JProfiler软件安装，[官网下载](https://www.ej-technologies.com/download/jprofiler/files)

![](/images/posts/other/1604971709654.jpg)

### 二. 配置IDEA运行环境
1. JProflier激活
    * JProfiler11 序列号
    > L-J11-Everyone#speedzodiac-327a9wrs5dxvz#463a59
    > 
    >A-J11-Everyone#admin-3v7hg353d6idd5#9b4
    ![](/images/posts/other/2018042515023882.png)
2. Preferences–>Tools–>JProflier–>JProflier executable选择JProfile安装可执行文件。
![](/images/posts/other/QQ20201110-093306.png)

### 三. IDEA中启动JProflier

1. 选择要分析的项目，点击JProfiler图标启动。
 ![](/images/posts/other/1604972954014.png)
2. 启动完成会自动弹出JProfiler窗口，在里面就可以监控自己的代码性能了。
![](/images/posts/other/20201110095319.png)
![](/images/posts/other/QQ20201110-095418.png)

### 四. JProflier功能介绍
1. 内存视图memory views

JProfiler的内存视图部分可以提供动态的内存使用状况更新视图和显示关于内存分配状况信息的视图。所有的视图都有几个聚集层并且能够显示现有存在的对象和作为垃圾回收的对象。

*   所有对象 All Objects 显示类或在状况统计和尺码信息堆上所有对象的包。你可以标记当前值并显示差异值。

*   记录对象 Record Objects 显示类或所有已记录对象的包。你可以标记出当前值并且显示差异值。

*   分配访问树 Allocation Call Tree 显示一棵请求树或者方法、类、包或对已选择类有带注释的分配信息的J2EE组件。

*   分配热点 Allocation Hot Spots
    显示一个列表，包括方法、类、包或分配已选类的J2EE组件。你可以标注当前值并且显示差异值。对于每个热点都可以显示它的跟踪记录树。

*   类追踪器 Class Tracker 类跟踪视图可以包含任意数量的图表，显示选定的类和包的实例与时间。

2. 堆遍历 heap walker

在JProfiler的堆遍历器(Heap Walker)中，你可以对堆的状况进行快照并且可以通过选择步骤下寻找感兴趣的对象。堆遍历器有五个视图.

*   类 Classes : 显示所有类和它们的实例，可以右击具体的类"Used Selected Instance"实现进一步跟踪。

*   分配 Allocations 为所有记录对象显示分配树和分配热点。

*   索引 References 为单个对象和“显示到垃圾回收根目录的路径”提供索引图的显示功能。还能提供合并输入视图和输出视图的功能。

*   时间 Time 显示一个对已记录对象的解决时间的柱状图。

*   检查 Inspections 显示了一个数量的操作，将分析当前对象集在某种条件下的子集，实质是一个筛选的过程。

*   图表 Graph

你需要在references视图和biggest视图手动添加对象到图表，它可以显示对象的传入和传出引用，能方便的找到垃圾收集器根源。

Ps:在工具栏点击"Go To Start"可以使堆内存重新计数，也就是回到初始状态。

3. cpu视图 cpu views

JProfiler 提供不同的方法来记录访问树以优化性能和细节。线程或者线程组以及线程状况可以被所有的视图选择。所有的视图都可以聚集到方法、类、包或J2EE组件等不同层上。

*   访问树 Call Tree
    显示一个积累的自顶向下的树，树中包含所有在JVM中已记录的访问队列。JDBC,JMS和JNDI服务请求都被注释在请求树中。请求树可以根据Servlet和JSP对URL的不同需要进行拆分。

*   热点 Hot Spots
    显示消耗时间最多的方法的列表。对每个热点都能够显示回溯树。该热点可以按照方法请求，JDBC，JMS和JNDI服务请求以及按照URL请求来进行计算。

*   访问图 Call Graph 显示一个从已选方法、类、包或J2EE组件开始的访问队列的图。

*   方法统计 Method Statistis 显示一段时间内记录的方法的调用时间细节。

4. 线程视图thread views

JProfiler通过对线程历史的监控判断其运行状态，并监控是否有线程阻塞产生，还能将一个线程所管理的方法以树状形式呈现。对线程剖析。

*   线程历史 Thread History
    显示一个与线程活动和线程状态在一起的活动时间表。

*   线程监控 Thread Monitor
    显示一个列表，包括所有的活动线程以及它们目前的活动状况。

*   线程转储 Thread Dumps
    显示所有线程的堆栈跟踪。

5. 监控器视图monitor views

*   当前锁定图表 Current Locking Graph :显示JVM中的当前锁定情况。
*   当前监视器 Current Monitors :显示当前正在等待或阻塞中的线程操作。
*   锁定历史图表 Locking History Graph :显示记录在JVM中的锁定历史。
*   监控器历史 Monitor History :显示等待或者阻塞的历史。
*   监控器使用统计 Monitor Usage Statistics :计算统计监控器监控的数据。

6. vm遥感勘测技术视图VM telemetry views

*   内存 Memory :显示堆栈的使用状况和堆栈尺寸大小活动时间表。
*   记录的对象 Recorded Objects :显示一张关于活动对象与数组的图表的活动时间表。
*   记录的生产量 Recorded Throughput ：

显示一段时间累计的JVM生产和释放的活动时间表。

*   垃圾回收活动 GC Activity：显示一张关于垃圾回收活动的活动时间表。
*   类 Classes ：显示一个与已装载类的图表的活动时间表。
*   线程 Threads ：显示一个与动态线程图表的活动时间表。
*   CPU负载 CPU Load ：显示一段时间中CPU的负载图表。


### 五. 配置JRebel和JProfiler同时运行
有些人在IDEA中配置了JRebel热部署，要想JRebel和JProfiler同时运行，就要改手动管理JProfiler session了。

**第一步，配置启动参数**
- 方式1：
在要分析的tomcat启动脚本catalina.bat中增加一行。
> set CATALINA_OPTS=-agentpath:D:\service\jprofiler9\bin\windows-x64\jprofilerti.dll=port=8849,nowait,id=80,config=C:\Users\Administrator\.jprofiler9\config.xml %CATALINA_OPTS%
- 方式2：
在IDEA Run/Debug Configurations窗口JRebel Debug中配置Environment Variables属性。
> CATALINA_OPTS=-agentpath:D:\service\jprofiler9\bin\windows-x64\jprofilerti.dll\=port\=8849,nowait,id\=81,config\=C:\Users\Administrator\.jprofiler9\config.xml

![](/images/posts/other/20201110095319.png)

**第二步，启动JRebel**
![](/images/posts/other/QQ20201110-095418.png)

**第三步，看到控制台如下信息，说明JRebel和JProfiler都启动了**
![](/images/posts/other/QQ20201110-095418.png)
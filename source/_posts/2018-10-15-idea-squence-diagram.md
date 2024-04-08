---
layout: post
title: IntelliJ-IDEA快速生成序列图
categories: Tools
description: IntelliJ-IDEA快速生成序列图
index_img: /img/post_def.png
date: 2018-10-15 09:09:09
tags: [tools,idea,skill]
---

在平时的学习/工作中，我们会经常面临如下场景：

1.  阅读别人的代码
2.  阅读框架源码
3.  阅读自己很久之前写的代码。

千万不要觉得工作就是单纯写代码，实际工作中，你会发现你的大部分时间实际都花在了阅读和理解已有代码上。

为了能够更快更清晰地搞清对象之间的调用关系，我经常需要用到序列图。手动画序列图还是很麻烦费时间的，不过 IDEA 提供了一个叫做**SequenceDiagram** 的插件帮助我们解决这个问题。通过 SequenceDiagram 这个插件，我们一键可以生成时序图。

## 哪些场景下需要查看类的时序图?

我们在很多场景下都需要时序图，比如说：

1.  **阅读源码** ：阅读源码的时候，你可能需要查看调用目标方法涉及的相关类的调用关系。特别是在代码的调用层级比较多的时候，对于我们理解源码非常有用。（*题外话：实际工作中，大部分时间实际我们都花在了阅读理解已有代码上。*）
2.  **技术文档编写** ：我们在写项目介绍文档的时候，为了让别人更容易理解你的代码，你需要根据核心方法为相关的类生成时序图来展示他们之间的调用关系。
3.  **梳理业务流程** ：当我们的系统业务流程比较复杂的时候，我们可以通过序列图将系统中涉及的重要的角色和对象的之间关系可视化出来。
4.  ......

## 如何使用 IDEA 根据类中方法生成时序图？

**通过 SequenceDiagram 这个插件，我们一键可以生成时序图。**

并且，你还可以：

1.  点击时序图中的类/方法即可跳转到对应的地方。
2.  从时序图中删除对应的类或者方法。
3.  将生成的时序图导出为 PNG 图片格式。

### 安装

我们直接在 IDEA 的插件市场即可找到这个插件。

> 如果你因为网络问题没办法使用 IDEA 自带的插件市场的话，也可以通过[IDEA 插件市场的官网](https://plugins.jetbrains.com/idea)手动下载安装。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c03be7afd3c24eae9644601df5ea954d~tplv-k3u1fbpfcp-zoom-1.image)

### 简单使用

1.  选中方法名（注意不要选类名），然后点击鼠标右键，选择 **Sequence Diagram** 选项即可！

![](https://img.itgo.ml/img/20201026094934.png)

2.  配置生成的序列图的一些基本的参数比如调用深度之后，我们点击 ok 即可！

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/63bc130cf38d49d6b637128b87a6648b~tplv-k3u1fbpfcp-zoom-1.image)

你还可以通过生成的时序图来定位到相关的代码，这对于我们阅读源码的时候尤其有帮助！

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c3a56ba79e74146b15a102ba9796cbe~tplv-k3u1fbpfcp-zoom-1.image)

时序图生成完成之后，你还可以选择将其导出为图片。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a40c87b53b7244dd8aa1defef1ca38ee~tplv-k3u1fbpfcp-zoom-1.image)

-------------------------


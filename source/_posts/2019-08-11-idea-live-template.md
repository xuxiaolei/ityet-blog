---
layout: post
title: IntelliJ IDEA代码模版使用介绍
categories: Java
description: IntelliJ IDEA代码模版使用介绍
index_img: https://xn--6or.us.kg/api/?raw
date: 2019-08-11 09:09:09
tags: [Java,IDEA]
---
# 背景
Intellij IDEA中的实时模板非常灵活且强大。你可以用它来极大提升代码的生产力

Java8除了lambda，最实用的特性是新的数据流API。集合操作在任何我见过的代码库中都随处可见。而且对于那些集合操作，数据流是提升代码可读性的好方法。

但是一件关于数据流的事情十分令我困扰：数据流只提供了几个终止操作，例如reduce和findFirst属于直接操作，其它的只能通过collect来访问。工具类Collctors提供了一些便利的收集器，例如toList、toSet、joining和groupingBy。

例如，下面的代码对一个字符串集合进行过滤，并创建新的列表：

`stringCollection
  .stream()
  .filter(e -> e.startsWith("a"))
  .collect(Collectors.toList());
`

在迁移了300k行代码到数据流之后，我可以说，toList、toSet、和groupingBy是你的项目中最常用的终止操作。所以我不能理解为什么不把这些方法直接集成到Stream接口上面，这样你就可以直接编写：

`stringCollection
  .stream()
  .filter(e -> e.startsWith("a"))
  .toList();
`

这在开始看起来是个小缺陷，但是如果你需要一遍又一遍地编写这些代码，它会非常烦人。

有toArray()方法但是没有toList()，所以我真心希望一些便利的收集器可以在Java9中这样添加到Stream接口中。是吧，Brian？ಠ\_ಠ

> 注：Stream.js是浏览器上的Java 8 数据流API的JavaScript接口，并解决了上述问题。所有重要的终止操作都可以直接在流上访问，十分方便。详情请见API文档。

无论如何，IntelliJ IDEA声称它是最智能的Java IDE。所以让我们看看如何使用IDEA来解决这一问题。

# [使用 IntelliJ IDEA 来帮忙](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

# 一、Live Template介绍
IntelliJ IDEA自带了一个便利的特性，叫做实时模板（Live Template）。如果你还不知道它是什么：**实时模板是一些常用代码段的快捷方式。** 例如，你键入sout并按下TAB键，IDEA就会插入代码段System.out.println()。

如何用实时模板来解决上述问题？实际上我们只需要为所有普遍使用的默认数据流收集器创建我们自己的实时模板。例如，我们可以创建.toList缩写的实时模板，来自动插入适当的收集器.collect(Collectors.toList())。

下面是它在实际工作中的样子：

![图片](https://mmbiz.qpic.cn/mmbiz_gif/6mychickmupXo3YZtg2jsqBkcy1lcXjJ0QuzDVpiaA4007WycwoA9K4MkRbXQXvwhhCAwbMXTuHqWBYjutKlf0cQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

# [构建你自己的实时模板](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

让我们看看如何自己构建它。首先访问设置（Settings）并在左侧的菜单中选择实时模板。你也可以使用对话框左上角的便利的输入过滤。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/6mychickmupXo3YZtg2jsqBkcy1lcXjJ0K37JDhzuRAMTgniaZY21wuT5zNFZThYyR7MmEp57qib3QeWPILzKVoiaw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

下面我们可以通过右侧的+图标创建一个新的组，叫做Stream。接下来我们向组中添加所有数据流相关的实时模板。我经常使用默认的收集器toList、toSet、groupingBy 和 join，所以我为每个这些方法都创建了新的实时模板。

这一步非常重要。在添加新的实时模板之后，你需要在对话框底部指定合适的上下文。你需要选择Java → Other，然后定义缩写、描述和实际的模板代码。

`//Abbreviation: .toList
.collect(Collectors.toList())

// Abbreviation: .toSet
.collect(Collectors.toSet())

// Abbreviation: .join
.collect(Collectors.joining("$END$"))

// Abbreviation: .groupBy
.collect(Collectors.groupingBy(e -> $END$))
`

特殊的变量`$END$`指定在使用模板之后的光标位置，所以你可以直接在这个位置上打字，例如，定义连接分隔符。

> 提示：你应该开启"Add unambiguous imports on the fly"（自动添加明确的导入）选项，便于让IDEA自动添加java.util.stream.Collectors的导入语句。选项在Editor → General → Auto Import中。

让我们在实际工作中看看这两个模板：

## [连接](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

![图片](https://mmbiz.qpic.cn/mmbiz_gif/6mychickmupXo3YZtg2jsqBkcy1lcXjJ0Tj5I4Yz9rfiarnrCW9TPbOVjq84ibFuhNGYict7vJd5a77qJIN87Vj7RA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

## [分组](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

![图片](https://mmbiz.qpic.cn/mmbiz_gif/6mychickmupXo3YZtg2jsqBkcy1lcXjJ05JM5ua6b3LnlQegHwlL41icyx4gbgiaWFs53Kic54eJzurtRhfRbruKwA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)


# 二、其他代码模版介绍

## 2.1、try/if/while/for/synchronized代码段

Intellij IDEA开发工具下，选中代码段，使用快捷键option+command+T 根据弹窗选中对应的语法，就看快速将代码段包含到对应语法内

## 2.2、fori/geti/main/psvm...快速生成语法结构模版

![image.png](https://cdn.nlark.com/yuque/0/2020/png/2545557/1602590096018-5311e794-8e88-48b3-bbeb-0fa6076f7520.png#align=left&display=inline&height=361&margin=%5Bobject%20Object%5D&name=image.png&originHeight=722&originWidth=982&size=401048&status=done&style=none&width=491)

1.  进入Intellij IDEA的设置页窗口
2.  选择Editor\->live Templates
3.  查看和编辑快捷模版

![image.png](https://cdn.nlark.com/yuque/0/2020/png/2545557/1602590209597-454b1bdf-545a-4a9b-9da0-2606c08a8d0b.png#align=left&display=inline&height=129&margin=%5Bobject%20Object%5D&name=image.png&originHeight=257&originWidth=673&size=49944&status=done&style=none&width=336.5)
在开发过程中Intellij IDEA输入指定缩写，就能快速生成对应语法结构。

## 2.3 自动代码抽取封装

![image.png](https://cdn.nlark.com/yuque/0/2020/png/2545557/1602592322151-2e112f6a-a687-405e-8b0f-0812e39f88d7.png#align=left&display=inline&height=354&margin=%5Bobject%20Object%5D&name=image.png&originHeight=707&originWidth=900&size=223054&status=done&style=none&width=450)
选中想要抽取的代码段，快捷键option+command+M，弹出自动抽取的方法，根据情况做对应修改调整，点击refactor自动完成抽取

---
layout: post
title: C#中相同string的GetHashCode() 值不一样详解
categories: C#
description: C#中相同string的GetHashCode() 值不一样详解
keywords: C#, GetHashCode
---

近日在开发使用Redis缓存过程中，为了避免缓存的KEY过长的情况，偷懒没用MD5，SHA2等hash算法,
而是使用了.NET中自带的GetHashCode()。

然后在使用的过程中问题就出现了，使用Redis缓存，必然要ADD、GET和DELECT,而由于项目原因，我的ADD和DELECT是在不同的项目中调用。

由于我的开发是在同一个项目中执行的，所以一直很顺利。

直到发布之后，测试过程中，出现了缓存一直清除不了的情况。一开始，以为没有更新代码所致，更新之后还是不行，检查了一遍代码，ADD和DELECT入参都是一致的，于是又怀疑是配置问题，然后对比了开发和预发布库的配置，发现还是没问题。

辗转查找原因，一直未解决。

最后查看了redis缓存记录的才发现，缓存里面的key和我开发时候产生的key是不一样的，于是百度确认了一下，最终确定就是这个GetHashCode()方法生成的KEY不一致的原因。    

** 那么为什么同一个string，两次运行GetHashCode得到的返回值却不一样呢？  

再次研究了这个“诡异”的函数。原来GetHashCode用于基于Hash的数据结构的key。Object.GetHashCode的实现只能保证将为相同的实例返回相同的哈希代码；它无法保证不同的实例将具有不同的哈希代码，或者两个引用相同值的对象将具有相同的哈希代码。

什么意思呢？就是在同一个进程的内存空间中，每个string的hash值都被微软保证不会重复（除非两个string的内容一样），虽然字符串的组合是无限的，但是对于一个进程的内存空间，顶多就几个G的，几个G的内存所能容纳的string的组合就变成了“有限的”了，int型的几十亿足够来保证这些“有限的”string组合有不同的hash值！  

但是微软不保证同一个string在调用GetHashCode之后得到的返回值是相同的！
因为int就几十亿，如果用来保证每次调用得到的返回值相同，那么势必出现hash值的碰撞！  这也是为什么MD5，SHA2等hash算法得到的返回值都是128位或者256位的原因，因为只有足够长，才能保证少发生碰撞或者发生碰撞之后可以二次hash！

此外.NET框架的不同版本对于相同的实例也可能会生成不同的哈希代码。 
     
## 参考

* [微软GetHashCode()的备注][1]


[1]: http://msdn.microsoft.com/zh-cn/library/system.string.gethashcode.aspx


---
layout: post
title: 解决Git Revert操作后再次Merge代码被冲掉的问题
categories: Git
description: Git Revert操作
date:  2021-10-14 09:09:09
tags: [Git,Java]
---
由于一次错误的代码合并操作dev\_cxn--->release--->master，导致还没有经过测试的代码被带到线上，并且在经历了几次其它分支的Merge后才发现线上代码有问题，当时想到两种解决办法：

1.reset到错误的合并之前，把后面的Merge操作再执行一遍。

2.直接使用revert，把错误的合并反向删除掉。

考虑到方法1操作动静太大，便使用了方法2，十分方便。

但是昨天发现了新的问题，即把master合并到dev\_cxn时，发现dev\_cxn上最后一次提交的代码丢失。经过检查，发现这次合并是fast-forward，而master上的revert操作**不会改变合并历史**，只是反向删除，所以**版本节点是向前更新**的，如下：

分支master  ---o---o---o---M---x---x---W
/
分支dev\_cxn      ---A---B

M是那次错误的合并，W是revert操作，此时若把master合并到dev\_cxn，将会直接快进到W节点，导致代码丢失。

于是想到把dev\_cxn合到master，结果发现Already-up-to-date，失败。。

再想到如果不用快进合并的方式，也许能够解决问题，于是执行git merge master --no-ff

\--no-ff

Create a merge commit even when the merge resolves as a fast-forward. This is the default behaviour when merging an annotated (and possibly signed) tag.

结果发现，即使不用快进的方式，revert那次操作还是会实实在在的执行进dev\_cxn分支，失败。。。

由于只是要让dev\_cxn的代码不被删除，便查了下Git是否能够执行类似“增量合并”的merge操作，无果。。。

最终在StackOverflow上找到一个类似问题：

![](https://img-blog.csdn.net/20150911145938325?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

原来只需要对那次revert操作再执行一下revert就行了，so easy。。。

Git不愧是地球第一版本管理神器
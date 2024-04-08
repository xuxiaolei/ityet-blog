---
layout: post
title: IIS支持htaccess伪静态规则的配置教程
categories: Blog
description: IIS支持htaccess伪静态规则的配置教程
date: 2018-11-21 09:09:09
tags: [htaccess, IIS,伪静态]
---

第一步：下载IIS伪静态组件——ISAPI_Rewrite3  [点击下载](httpwww.jb51.netsofts41171.html)

第二步：先在服务器上点击安装ISAPI_Rewrite3_0082.msi  [点击下载](httpwww.helicontech.comisapi_rewritedownload.html)，选择I accept选项后一路下一步默认安装 
![](httpsraw.githubusercontent.comxuxiaoleiimagesmaster20190717172001.png)

第三步：打开安装的目录，默认安装的都是在CProgram FilesHeliconISAPI_Rewrite3这个目录下，先删除掉ISAPI_Rewrite.dll和ISAPI_RewriteSnapin.dll这两个文件，如果删除不掉，就重启iis，在重启过程中删除，重启iis的图片看下图

![](httpsraw.githubusercontent.comxuxiaoleiimagesmaster20190717173418.png)

第四步：粘贴软件包里面的ISAPI_Rewrite.dll和ISAPI_RewriteSnapin.dll这两个文件到你的安装目录中；

第五步：在软件安装目录下找到httpd.conf这个文件，用文本文档打开，把以下内容复制到里面保存：

RegistrationName= coldstar

RegistrationCode= 2EAD-35GH-66NN-ZYBA

第六步：在软件安装目录，右键点击ISAPI_Rewrite.dll的属性按钮，在弹出的对话框中选择“安全”标签，点击添加，如下图：

![](https://img.itgo.ml/img/20190717173631.png)

在弹出的对话框中选择高级，如下图：

![](https://img.itgo.ml/img/20190717173735.png)

在接下来的对话框中再选择“立即查找”，如下图

![](https://img.itgo.ml/img/20190717173820.png)

在下面的用户列表中找到everyone这个用户，点击确定，如下图：

![](https://img.itgo.ml/img/20190717173852.png)

再点击确定，如下图：

![](https://img.itgo.ml/img/20190717173938.png)

最后再勾选上全部的权限，点击确定就可以了，如下图：

![](https://img.itgo.ml/img/20190717174025.png)

第七步：大功告成，这样以后你的程序如果支持伪静态的话就把.htaccess文件放到你的网站根目录下就可以了，默认是全局支持的！

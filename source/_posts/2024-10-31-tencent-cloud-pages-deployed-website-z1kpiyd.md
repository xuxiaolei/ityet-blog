---
layout: post
title: 腾讯云Pages部署网站
date: '2024-10-31 10:38:31'
permalink: /post/tencent-cloud-pages-deployed-website-z1kpiyd.html
published: true
---

# 腾讯云Pages部署网站

​![image](https://ityet.com/img/image-20241031104343-dx1eqqr.png)​

## 一、简单介绍

EdgeOne Pages 是基于 腾讯云 EdgeOne 基础设施打造的前端开发和部署平台，专为现代 Web 开发设计，帮助开发者快速构建、部署静态站点和无服务器应用。通过集成边缘函数能力，实现高效的内容交付和动态功能扩展，支持全球用户的快速访问。

适用场景：

> **静态与动态网站托管**：适合使用静态生成器，如 Next.js、Hexo 等构建的网站，以及使用 React、Vue 等现代前端框架构建的单页应用。
>
> **构建部署自动化**：通过 GitHub 等代码管理平台集成，在每次代码提交时自动构建和部署网站，简化开发部署流程，提升研发效率。
>
> **即将推出边缘开发**：边缘函数能力即将上线，其提供了边缘节点的 Serverless 代码执行环境，方便开发者高效地开发全栈类应用。

‍

## 二、部署步骤

### 2.1 连接Git

​![image](https://ityet.com/img/image-20241031105317-kzrs2sa.png)​

Github授权，选择关联个人仓库或者企业仓库

### 2.2 选择Github源码开始部署

​![image](https://ityet.com/img/image-20241031105242-32qioel.png)​

按照自己项目情况配置，根目录，输出目录，构建命令，安装命令

### 2.3 自定义域名配置

​![image](https://ityet.com/img/image-20241031105903-33x0ali.png)​

​![image](https://ityet.com/img/image-20241031105935-jprjs4z.png)​

​![image](https://ityet.com/img/image-20241031110017-hpvua4n.png)​

打开项目设置页面，选择添加自定义域名，填写域名后，会显示cname记录值，配置到阿里云云解析，配置完成后点击验证，验证成功，会显示已设置，证书会自动生成，需要等待几分钟。

### 2.4 代码修改&分支修改

​![image](https://ityet.com/img/image-20241031110238-5mq4b7h.png)​

​![image](https://ityet.com/img/image-20241031105748-y24u7og.png)​

打开项目设置，选择Git管理，指定生产分支。

修改代码Github提交后，自动产生部署记录。

### 2.5 部署结果

​![image](https://ityet.com/img/image-20241031105724-wiqiqkb.png)​

部署成功后，可以看到网站预览图，可以看到绑定的自定义域名，还有部署记录。

## 三、常见问题

#### **1.如何开始使用 EdgeOne Pages？**

EdgeOne Pages 的入门非常简单！只需注册腾讯云账户，连接您的 Git 仓库，选择我们的模板或您自己的 Git 项目，然后点击部署。参照页面上的指引即可快速上线项目。

#### **2.我可以部署哪些类型的 Web 应用程序？**

您可以部署多种类型的 Web 应用程序，例如静态生成器 Next.js、Hexo 等构建的网站，React、Vue 等现代前端框架构建的单页应用，以及利用即将上线的边缘函数能力构建的全栈应用。

#### **3.如何设置自定义域名？**

添加自定义域名非常简单。只需在控制台中添加您的域名，并在域名的供应商系统更新 **DNS** 设置。设置成功后平台会自动为您提供 [SSL 证书](https://cloud.tencent.com/product/symantecssl?from_column=20065&from=20065)。

#### **4.是否提供免费版本？**

我们长期提供免费版本，公测期间几乎无限制，允许您使用产品的基本功能，同时我们将持续迭代更多高级能力，并确保服务的稳定性。当未来我们进行商业化时，免费版本可能有一定限制，例如构建次数等，届时将及时通知到您。

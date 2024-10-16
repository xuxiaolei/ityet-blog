---
layout: post
title: Gson反序列化泛型
categories: Java
description: Gson反序列化泛型
index_img: 
date: 2019-07-30 09:09:09
tags: [Gson,Json]
---

公司内部微服务架构基础设，通常技术选型以Spring Cloud技术为主，也被大家俗称作“全家桶”。

> 因其具备微服务架构体系中所需的各个服务组件，比如服务注册发现(如Spring Cloud Eureka、Zookeeper、Consul)、API网关路由服务(Spring Cloud Zuul)，客户端负载均衡(Spring Cloud Ribbon，Zuul默认集成了Ribbon)、服务容错保护(Spring Cloud Hystrix)，消息总线 (Spring Cloud Bus)、分布式配置中心(Spring Cloud Config)、消息驱动的微服务(Spring Cloud Stream)、分布式链路跟踪服务(Spring Cloud Sleuth)。

下面围绕其中一个组件 分布式配置中心 展开了解学习。

## Spring Cloud Config配置中心介绍&架构

在微服务架构体系中配置中心是比较重要的组件之一，Spring Cloud官方自身提供了Spring Cloud Config分布式配置中心，由它来提供集中化的外部配置支持，它分为客户端和服务端两个部分。其中服务端称作配置中心，是一个独立的微服务应用，用来连接仓库(如Git、Svn)并未客户端提供获取配置的接口；而客户端是各微服务应用，通过指定配置中心地址从远端获取配置内容，启动时加载配置信息到应用上下文中。因Spring Cloud Config实现的配置中心默认采用了Git来存储配置信息，所以版本控制管理也是基于Git仓库本身的特性来支持的 。对该组件调研后，主要采用基于消息总线的架构方式，架构图如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz/N34tfh8WYkhUibxxQ705F0befWxoux8TxlBhumByGqcv0rHuypmZ8y0oGSdXhVvKU1GewWTBE7oUaL0OwsfFrEA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

基于消息总线的配置中心架构中需要依赖外部的MQ组件，如Rabbit、Kafka 实现远程环境事件变更通知，客户端实时配置变更可以基于Git Hook功能实现。同时架构图中看到最右侧，有一个**Self scheduleing refresher**这个是我在实践中自己新增的一个扩展功能，目的是当依赖的消息组件出现问题时，此时如果Git仓库配置发生了变更，会导致部分或所有客户端可能无法获取到最新配置，这样就造成了客户端应用配置数据无法达到最终一致性，进而引起线上问题。

> **Self scheduleing refresher**是一个定时任务，默认5分钟执行一次，执行时会判断本地的Git仓库版本与远程Git仓库版本如果不一致，则会从配置中心获取最新配置进行加载，保障了配置最终一致性。

经过实际使用你会发现Spring Cloud Config这个配置中心并不是非常好用，如果是小规模的项目可以使用问题不大，但它并不适用于中大型的企业级的配置管理。因此，我对业界开源的配置中心做个对比后最终选择了携程开源的Apollo配置中心解决了微服务架构配置管理和其他项目中配置管理痛点。

下面就针对Spring Cloud Config和Apollo配置中心做个更加直观的比对：**Apollo VS Spring Cloud Config**

![图片](https://mmbiz.qpic.cn/mmbiz/N34tfh8WYkhUibxxQ705F0befWxoux8TxVJYyqpA5Nf9guGTgCl6WlPDVWLZpQvlIasGV6rXDfoiaRlD8DUq8Bfg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

通过以上对比图发现Spring Cloud Config缺陷还是挺大的，比如最后一条高可用，Apollo配置中心客户端应用加载配置后本地会生成缓存文件，即使配置中心所有的服务都挂掉，只是配置无法更新，但是不影响你的服务启动。而这Spring Cloud Config是无法做到的，有人会说我们可以在应用classpath下添加应用配置文件作为「兜底使用」，这样做首先配置不会自动同步，而且也不是Spring Cloud Config自身的功能。

另外还有一个原因是因为现阶段项目中也使用了一些自研的配置中心，但都差强人意，有的配置中心仅支持xml格式的，无法支持KV形式；还有的配置中心是基于JMX开发的，但只支持属性配置推送。所以经过对Apollo配置中心的调研和使用发现这款产品不仅适用于微服务配置管理场景，同时也支持多种配置格式，如xml、json、yml，还支持多语言客户端的接入，在配置服务治理方面也是很完善的，在携程内部已经支撑10万+的实例运行，成熟又稳定！

## **开源配置中心对比**

下面这个图详细的开源配置中心对比图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/N34tfh8WYkhUibxxQ705F0befWxoux8Tx5QhlALtHCEsUeKoCc04ZED3fq4YRzRVJ9CRWLrgjQeSKMrhVk89ia7Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在上述几个开源配置中心里，Apollo社区是非常活跃的，不断更新迭代，github上的Star数量已达8K+，Fork数量已达2.8K+。在Apollo出现之前百度开源的disconf配置中心使用的更多些，disconf最新代码更新时间还是2年前的，且与Apollo的对比社区活跃度有所下降。

## Apollo配置中心介绍&架构

Apollo（阿波罗）是携程框架部门研发的分布式配置中心，能够集中化管理应用不同环境、不同集群的配置， 配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。服务端基于Spring Boot和Spring Cloud开发，不依赖外部容器，便于部署。Java客户端不依赖任何框架，能够运行于所有Java运行时环境，同时对Spring/Spring Boot环境也有额外支持。原生支持Java和.Net客户端，同时也支持其他语言客户端，目前已支持Go、PHP、Python、NodeJS、C++。

### 主要功能特性：

> 统一管理不同环境、不同集群的配置 配置修改实时生效（热发布） 版本发布管理 灰度发布 权限管理、发布审核、操作审计 客户端配置信息监控 提供Java和.Net原生客户端 提供开放平台API 部署简单，依赖少

### Apollo总体架构设计：

![图片](https://mmbiz.qpic.cn/mmbiz_png/N34tfh8WYkhUibxxQ705F0befWxoux8TxXe97HBXKXNOia01kkWLicKSwgMiaNibst0VOz7Tib06aXxibR0s3XTaOJtJw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

各组件作用说明：

![图片](https://mmbiz.qpic.cn/mmbiz/N34tfh8WYkhUibxxQ705F0befWxoux8TxzoJwuy42qibicBicibZia4SKoKhXY90zWS3Qc1uye3RC8Ex8tJeiblfuBF2w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### Apollo HA高可用设计：

![图片](https://mmbiz.qpic.cn/mmbiz/N34tfh8WYkhUibxxQ705F0befWxoux8TxnQq5ibT5O1wDlNI4bxdRxwCDJx8uwYoGFjicG04Qt46fB18ibXZpKpyjA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### Apollo客户端架构：

![图片](https://mmbiz.qpic.cn/mmbiz/N34tfh8WYkhUibxxQ705F0befWxoux8TxibibTz98OY9b4VgAx9TwLiaib62qBaMiaWRU4SU7mIGkF5EUibt9lEkR36ug/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

客户端架构原理：

1.  推拉结合方式 客户端与配置中心保持一个长连接，配置实时推送 定时拉配置(默认5分钟)

2.  本地缓存 配置缓存在内存 本地缓存一份配置文件

3.  应用程序 通过Apollo客户端获取最新配置 订阅配置更新通知

### Apollo核心概念：

application (应用)

> 每个应用都需要有唯一的身份标识 \-\- appId

environment (环境)

> Apollo客户端通过不同环境获取对应配置

cluster (集群)

> 一个应用下不同实例的分组，不同的cluster，可以有不同的配置。比如北京机房和天津机房可以有不一样的kafka或zk地址配置。

namespace (命名空间)

> 一个应用下不同配置的分组，不同的namespace的类似于不同的文件。如：数据库配置，RPC配置等。支持继承公共组件的配置。**配置分类**私有类型（private）：只能被所属应用获取 公共类型（public）：必须全局唯一。使用场景：部门/小组级别共享配置，中间件客户端配置。关联类型（继承类型）：私有继承公有配置并覆盖；定制公共组件配置场景。 **配置项(Item)**默认和公共配置使用properties格式；私有配置支持properties/json/xml/yaml/yml格式。定位方式：app+cluster+namespace+item\_key

权限管理

> 系统管理员拥有所有的权限 创建者可以代为创建项目，责任人默认是项目管理员，一般创建者=责任人 项目管理员可创建集群，Namespace，管理项目和Namespace权限 编辑权限只能编辑不能发布 发布权限只能发布不能编辑 普通用户可以搜索查看所有项目配置，但没有相关操作权限

### Apollo配置中使用及扩展

使用Apollo配置中心后，做了进一步的功能开发扩展，接入公司的SSO和邮件通知接入。基于Spring Cloud Config微服务架构体系中，如果之前使用了Spring Cloud Config配置中心，也可以通过下列方式平滑的迁移到Apollo配置中心。

Spring Cloud微服务项目在pom.xml中引入如下依赖：

`<dependency>
<groupId>com.letv.micro.apollo</groupId>
<artifactId>micro-apollo-spring-boot-starter</artifactId>
<version>1.0-SNAPSHOT</version>
</dependency>
`

该源码参考Github：https://github.com/david1228/micro\-apollo\-spring\-boot\-starter，需要自行编译打成jar包使用。这个jar包对Spring Cloud配置刷新机制集成Apollo客户端做了进一步封装，实现的主要功能如下：

> 1、在Apollo配置中心发布配置后，微服务应用客户端监听配置变更，包括默认的配置和公共的配置，通过ContextRefresher中的refresh()方法完成应用环境上下文的配置刷新。2、支持对日志级别和日志路径文件的动态配置变更。\[Apollo Client无法很好的支持日志级别和日志路径文件的变更，因日志的LoggingApplicationListener加载优先级高，Apollo配置加载滞后。

上述jar包已上传公司的Maven私服。具体配置使用示例可以参考「4.Apollo配置中心使用示例」

引入micro\-apollo\-spring\-boot\-starter之后，可以将spring\-cloud\-stater\-config依赖从pom.xml中去掉了。**Apollo配置中心公共配置命名规范:**公共配置建议统一放到新创建的项目中，该项目中存放Spring Cloud相关的公共组件的配置，比如Eureka、Zipkin、Stream等配置，比如Eureka地址可能是多个微服务应用共用的，便于在该项目中统一对配置进行管理。创建项目时，选择的部门如为「微服务公共平台(dpms)」 各微服务应用项目创建后可以添加Namespace，选择关联公共配置。公共配置命名规则：{部门前缀}.application 或者 {部门前缀}.application\-{具体的细分配置} 当Apollo配置发布后，若需让Spring Cloud配置实现动态加载，公共配置命名必须以application关键字开头，在上述依赖的jar包中会对这类命名的Namespace做配置变更监听。例如：

> dpms.application\-eureka 存放eureka相关配置 或 dpms.application\-zipkin 存放zipkin相关配置 或 dpms.application 存放Spring Cloud所有的公共相关配置 其他微服务应用关联公共配置后，默认使用的公共配置项。你也可以对公共配置所有参数做覆盖，覆盖后优先获取本项目中的配置，这个特性在Apolo的公共配置界面能够直观的展示出来。

以上就是对为什么要选择Apollo配置中心的一些介绍，相信你的项目中可能也遇到了类似的配置管理问题或痛点，强烈建议使用Apollo配置中心作为你的配置管理基础服务使用。

关于Apollo更详尽的文档请参考Github：https://github.com/ctripcorp/apollo

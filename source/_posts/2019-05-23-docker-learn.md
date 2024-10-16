---
layout: post
title: Docker容器化技术入门
categories: Docker
description: Docker容器化技术入门
index_img: /img/post_def.png
date: 2019-05-23 09:09:09
tags: [Docker]
---

![Docker](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f0545712925d441fafbd015e9ca50849~tplv-k3u1fbpfcp-watermark.image)


## 一、解决的问题

1.  开发人员 与 测试人员 环境不一致，导致运行结果不同
2.  应用和环境的 布署、迁移 工作繁重
3.  虚拟机耗费资源多、性能差

## 二、发展历程

1.  虚拟化 Virtualization \- 将硬件资源(CPU、内存、硬盘、网络)虚拟化，再通过虚拟化出来的空间安装操作系统
2.  容器化 Container \- 将应用和环境打包成一个统一标准的容器，使其可在任意操作系统上的 Container Runtime 运行 \- 类似 JVM

## 三、核心组件

1.  Docker 引擎(守护进程) \- 一个长期运行的进程

    *   与 Docker 客户端交互

    *   构建、运行、分发 Docker 容器

2.  Docker 镜像 \- 只读模板，由一系列指令构建而成

    *   Docker 生命周期中，构建和打包阶段

    *   当容器从镜像启动时，会在镜像最上层创建一个可写层

3.  Docker 容器

    *   运行、隔离应用

    *   Docker 生命周期中，启动和运行阶段

4.  Docker 仓库

    *   集中存放某一类镜像文档，不同镜像透过 Tag 来区分，类似 Git 中的项目
    *   仓库注册服务器(Registry)，负责存放仓库，类似 GitHub
    *   根据镜像的公开与否，可分为 公开仓库、私有仓库

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fcafee02626e4a7896539d925b7f0965~tplv-k3u1fbpfcp-zoom-1.image)

## 四、指令

### 1\. 镜像

*   获取 \- docker pull NAME\[:TAG\]

    *   NAME 的 Registry 默认为 registry.hub.docker.com
    *   TAG 默认为 latest
*   添加标签 \- docker tag \[原镜像名:TAG\] \[目标镜像名:TAG\] \- 会有两个镜像消息，但都属于同一个镜像

*   查看详细信息 \- docker inspect NAME\[:TAG\]

*   查看分层信息 \- docker history 镜像ID

*   查看所有镜像信息 \- docker images \- 使用 `-f` 过滤信息

*   搜索 \- docker search 名称

*   删除

    *   docker rmi NAME\[:TAG\]

    *   docker rmi IMAGE\_ID

        > 如有容器正在运行该镜像，将无法删除，可以使用 \-f 强制删除，但不推荐

*   上传 \- docker push NAME\[:TAG\] \- 默认上传到 Docker Hub 官方仓库(需登录)

*   透过 DockerFile 创建 \- docker build \[\-f DockerFile路径\] \[\-t 镜像名称:标签\] .

### 2\. 容器

*   创建 \- docker create NAME\[:TAG\]

    *   \-i \- 交互模式

    *   \-t \- 伪终端

    *   \-d \- 后台运行

    *   \-e key=value \- 指定环境变量

    *   \-m 内存大小 \- 最大内存使用量 \- b、k、m、g

    *   \-v Linux目录:Docker目录\[:ro\] \- Linux目录与容器内数据卷映射

        > Linux 目录必须存在

    *   \-p 端口:端口 \- 将 Linux 端口与 Docker 端口映射

    *   \-\-name 容器名 \- 指定容器名

    *   \-\-rm \- 容器退出后自动删除容器

    *   \-\-network

        *   bridge \- 【默认】网桥，根据 MAC 地址转发侦(路由器是根据IP转发侦)
        *   host \- 使用宿主机 IP \- 不能再使用 `-p`
        *   overlay \- 将多个 Docker 引擎连接在一起，使集群间可以互相通信
        *   macvlan \- 将 MAC 地址分配给容器，在网络上显示为物理设备
        *   none \- 禁用网络
    *   \-\-tmpfs \- 数据卷只挂载在内存，不会持久化

*   启动 \- docker start 容器id

*   查看状态

    *   运行的容器 \- docker ps
    *   所有的容器 \- docker ps \-a
*   查看详情

    *   docker inspect 容器id
    *   docker stats 容器id
*   新建并启动 docker run NAME\[:TAG\]

    > 相当于 create + start
    >
    > 可使用 create 的选项参数

*   停止

    *   docker stop 容器id \-t 时间 \- 【默认】 10 秒
    *   docker kill 容器id
*   重启 \- docker restart 容器id

*   进入 \- docker exec \-it 容器id /bin/bash

*   删除 \- docker rm 容器id

    > 如有该容器正在运行，将无法删除，可以使用 \-f 强制删除

*   暂停 \- docker pause 容器id

*   暂停恢复 \- docker unpause 容器id

### 3\. 仓库

*   登录 \- docker login \-u username \-p password
*   注销 \- docker logout
*   认证文档 \- Linux 下复制文档内容即可免密登录 /root/.docker/config.json

### 4\. DockerFile

> 由一行行命令语句构成
>
> 【目的】构建镜像

*   FROM NAME\[:TAG\] \- 基于镜像
*   MAINTAINER 姓名 信箱 \- 维护者姓名、信箱
*   COPY Linux路径 Docker路径 \- 将本机资源复制进 Docker 容器中 \- 支持通配符、正则
*   RUN 命令 \- 运行命令
*   ADD 路径1 Docker路径 \- 获取资源(可远程)到 Docker 容器中
    *   【推荐】远程资源 用 RUN wget
    *   【推荐】本机资源 用 COPY
*   CMD 命令 \- Docker 启动后运行的命令 \- 只能有一条
*   ENTRYPOINT 命令 \- 与 CMD 指令相似
*   ENV key=value \- 设置环境变量
*   VOLUME Docker路径 \- 声明 Docker 容器中的路径为匿名卷
*   WORKDIR Docker路径 \- 切换目录
*   EXPOSE 端口1 端口2 \- 设置监听端口

### 5\. 数据卷

*   创建 \- docker volume create 卷名
*   查看列表 \- docker volume ls
*   查看消息 \- docker volume inspect 卷名
*   删除 \- docker volume rm 卷名

### 6\. Compose

> 【目的】 同时运行多个不同镜像的 Docker 容器

**命令**

*   运行( build + start ) \- docker\-compose up \-d
*   查看 \- docker\-compose ps
*   查看日志 \- docker\-compose logs
*   服务端口绑定 \- docker\-compose port 服务名 端口
*   重新构建服务 \- docker\-compose build
*   启动 \- docker\-compose start \[服务名\]
*   停止 \- docker\-compose stop \[服务名\]
*   删除 \- docker\-compose rm \[服务名\]
*   停止并删除 \- docker\-compose down \[服务名\] \[\-v\]
*   运行命令 \- docker\-compose run 服务名 命令

**配置**

> 默认文件名称为 docker\-compose.yml

*   version \- 与 Swarm 结合需要 3.0 以上
*   services \- 定义服务
*   image \- 使用镜像
*   build \- 使用 DockerFile 构建镜像
*   ports \- 设置端口映射
*   volumes \- 映射数据卷容器
*   command \- 覆盖容器启动后默认运行的命令
*   depends\_on \- 设置服务启动顺序
*   environment \- 设置环境变量
*   network\_mode 设置网络模式

```yaml
version: '2.0' # 版本2.0
services: # 定义服务
  mysql: # 服务名称
    image: mysql:5.7 # 使用镜像
    ports: #端口映射
      - 3306:3306
    volumes: # 卷映射
      - /mydata/mysql/log:/var/log/mysql
      - /mydata/mysql/data:/var/lib/mysql
      - /mydata/mysql/conf:/etc/mysql
    environment:
      MYSQL_ROOT_PASSWORD: myPassword
  redis:
     image: redis:6.0
     ports:
       - 6379:6379
  halo:
    image: tony60107/halo:v1
    ports:
      - 8090:8090
复制代码
```

### 7\. Swarm

> 【目的】修改 Docker 容器集群配置，无须手动重启

*   查看集群状态 \- docker info
*   查看节点信息 \- docker node ls
*   查看服务信息
    *   所有的服务 \- docker service ls
    *   运行的服务 \- docker service ps \[服务名称\]
*   在 Manager 查看令牌消息 \- docker swarm join\-token manager
*   添加工作节点到集群 \- docker swarm join 令牌
*   添加管理节点到集群 \- docker swarm join\-token manager 令牌
*   将当前节点驱离集群 \- docker swarm leave \-\-force
*   将服务发布到集群 \- docker service create 服务名
    *   \-p 端口:端口 \- 端口映射
    *   \-\-replicas 实例数 \- 总运行实例个数
    *   \-\-name 名称 \- 自订服务名称
*   停止并删除服务 \- docker service rm \[服务名\]
*   扩展服务 \- docker service scale 服务名=实例数
*   更新服务 \- docker service update 属性名 属性
---
layout: post
title: Maven中配置多个mirror切换
categories: Java
description: Maven中配置多个mirror切换
index_img: 
date: 2017-05-11 09:09:09
tags: [Maven]
---

# 前言

Maven是开发过程中必备的工具。在日常工作中，公司用部门搭建的maven私服做镜像，回到家用aliyun的镜像，每次都要改配置文件，很麻烦，希望能够不改动配置文件的情况下，动态切换mirror配置。

# Maven的mirror和repository
Maven里的mirror和repository是两个比较容易混淆的概念，它们的作用都是配置远程maven仓库的地址。顾名思义，repository就是直接配置站点地址，mirror则是作为站点的镜像，代理某个或某几个站点的请求，实现对repository的完全代替。

## Repository
有两种形式可以配置多个repository, 配置多个profile或者在同一个profile中配置多个repository.配置多profile时，还需要配置activeProfiles使配置生效。

配置示例：

```xml
        <profiles>
          <profile>
            <id>central</id>
            <repositories>
                <repository>
                    <id>central</id>
                    <url>http://search.maven.org/</url>
                    <!-- <releases> <enabled>true</enabled> </releases> <snapshots> <enabled>true</enabled> 
                        </snapshots> -->
                    <releases>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                    </releases>
                    <snapshots>
                        <enabled>false</enabled>
                        <updatePolicy>always</updatePolicy>
                    </snapshots>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <id>central</id>
                    <url>http://search.maven.org/</url>
                    <releases>
                        <enabled>false</enabled>
                        <updatePolicy>always</updatePolicy>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
          <profile>
            <id>aliyun</id>
            <repositories>
                <repository>
                    <id>aliyun</id>
                    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
                    <!-- <releases> <enabled>true</enabled> </releases> <snapshots> <enabled>true</enabled> 
                        </snapshots> -->
                    <releases>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                    </snapshots>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <id>aliyun</id>
                    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
                    <releases>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
        </profiles>
        <activeProfiles>
            <activeProfile>aliyun</activeProfile>
            <activeProfile>central</activeProfile>
        </activeProfiles>
```

单profile,多repository的配置也类似。

这样就实现了多站点的配置。下载依赖时，maven会按照配置从上到下的顺序，依次尝试从各个地址下载，成功下载为止。

## mirror

mirror的配置示例：

```xml
    <mirror>
            <id>nexus-aliyun</id>
            <mirrorOf>*</mirrorOf>
            <name>Nexus aliyun</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
```

使用`mirrorOf`指定这个镜像是针对哪个`repository`的，配置成`*`就表示要代理所有repository的请求。

需要注意的是，与repository不同，配置到同一个repository的多个mirror时，相互之间是备份的关系，只有在仓库连不上的时候才会切换另一个，而如果在能连上的情况下找不到包，是不会尝试下一个地址的。

# mirror动态切换实现
```xml
<mirrors>
  <mirror>
    <id>aliyun</id>
    <url>https://maven.aliyun.com/repository/public</url>
	<mirrorOf>${aliyun}</mirrorOf>
  </mirror>
  <mirror>
    <id>netease</id>
    <url>http://mirrors.163.com/maven/repository/maven-public/</url>
    <mirrorOf>${netease}</mirrorOf>
  </mirror>
   <mirror>
    <id>default</id>
    <url>http://192.168.0.123/nexus/repository/maven-public/</url>
    <mirrorOf>central</mirrorOf>
  </mirror>
</mirrors>
```

我们知道，默认情况下配置多个mirror的情况下，只有第一个生效。那么我们可以将最后一个作为默认值，前面配置的使用环境变量动态切换。

默认情况下，执行： `mvn help:effective\-settings` 可以看到使用的是私服。

如果希望使用阿里云镜像，如下执行：

```
mvn help-effective-settings -Daliyun=central

```

同样的道理，使用网易镜像，则执行：

```
mvn help:effective-settings -Dnetease=central
```
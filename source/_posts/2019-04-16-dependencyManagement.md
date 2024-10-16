---
layout: post
title: Maven中dependencyManagement标签学习
categories: Java
description: Maven中dependencyManagement标签学习
index_img: https://xn--6or.us.kg/api/?raw
date: 2019-04-16 09:09:09
tags: [Java,pom,dependencyManagement]
---

当我们的项目模块很多的时候，在我们项目顶层的POM文件中，我们会看到dependencyManagement元素。通过它元素来管理jar包的版本，让子项目中引用一个依赖而不用显示的列出版本号。Maven会沿着父子层次向上走，直到找到一个拥有dependencyManagement元素的项目，然后它就会使用在这个dependencyManagement元素中指定的版本号。

---

#### 业务场景

*   一个项目有很多模块，每个模块都会用到一些**公共的依赖**
*   这些公共的依赖若交由**各个模块独自管理**，若每个模块同一个依赖的版本号不一致，会给项目的整
    *   打包和开发测试环境下**对同一 jar 包不同版本号的处理可能不一致**，造成运行时和测试时**结果不一致**
    *   项目升级时，会造成**修改版本号时遍地开花**的问题
*   该标签通常适用于多模块环境下**定义一个top module**来专门管理公共依赖的情况下

#### 项目中依赖包版本号判断途径

*   若 dependencies 里的 dependency 自己没有声明 version 元素，那么maven 就会 到 depenManagement 里去找有没有该 artifactId 和 groupId 进行过版本声明，若存在，则继承它，若没有则报错，你必须为dependency声明一个version\*\*
*   若 dependencies 中的 dependency 声明了version，则 dependencyManagement 中的声明无效

**单一模块情况下 pom.xml**

~~~
//只是对版本号进行管理，不会实际引入jar  
<dependencyManagement>  
      <dependencies>  
            <dependency>  
                <groupId>org.springframework</groupId> //jar 包身份限定  
                <artifactId>spring-core</artifactId>  
                <version>3.2.7</version>  //版本号的声明
            </dependency>  
    </dependencies>  
</dependencyManagement>  
  
//会实际下载jar包  
<dependencies>  
       <dependency>  
                <groupId>org.springframework</groupId>  
                <artifactId>spring-core</artifactId> //不声明version 标签，则会继承
       </dependency>  
</dependencies>

~~~

**多模块情况：**  
parent-module 顶层模块，son1-module 和 son2-module 并列继承 parent-module

~~~
parent-module 中 pom.xml

<properties>
    // 集中在properties 标签中定义所有 依赖的版本号
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <org.eclipse.persistence.jpa.version>1.2.6</org.eclipse.persistence.jpa.version>
    <developer.organization>xxx</developer.organization>
    <javaee-api.version>1.8</javaee-api.version>
</properties>

<dependencyManagement>  
    //定义公共依赖的版本号
    <dependencies> 
        <dependency>  
            <groupId>org.eclipse.persistence</groupId>  
            <artifactId>org.eclipse.persistence.jpa</artifactId>  
            <version>${org.eclipse.persistence.jpa.version}</version>  
            <scope>provided</scope>  
        </dependency>  
          
        <dependency>  
            <groupId>javax</groupId>  
            <artifactId>javaee-api</artifactId>  
            <version>${javaee-api.version}</version>  
        </dependency>  
    </dependencies>  
</dependencyManagement> 

~~~

~~~
son-module1 中 的 pom.xml
<!--继承父类-->  
<parent>  
    <artifactId>parent-module</artifactId> //声明父类的身份信息
    <groupId>com.ityet</groupId>  
    <version>0.0.1-SNAPSHOT</version>  
    <relativePath>../parent-module/pom.xml</relativePath> //声明父类的pom 文件路径
</parent>  

<modelVersion>4.0.0</modelVersion>  
<artifactId>son-module</artifactId>  
<packaging>ejb</packaging>  
  
<!--依赖关系-->  
<dependencies>  
    <dependency>  
        <groupId>javax</groupId>  
        <artifactId>javaee-api</artifactId> //继承父类
    </dependency>  
      
    <dependency>  
        <groupId>com.fasterxml.jackson.core</groupId>  
        <artifactId>jackson-annotations</artifactId>  //继承父类
    </dependency>  
      
    <dependency>  
        <groupId>org.eclipse.persistence</groupId>  
        <artifactId>org.eclipse.persistence.jpa</artifactId>  //继承父类
        <scope>provided</scope>  
    </dependency>  
</dependencies> 

~~~

#### 与 dependencies 标签下 dependency 的区别

*   所有声明在d ependencies 里的依赖都会自动引入，并默认被所有的**子项目继承**
*   dependencies 即使在子项目中不写该依赖项，那么子项目仍然会**从父项目中继承**该依赖项（全部继承）
*   dependencyManagement **只是声明依赖的版本号，该依赖不会引入**，因此子项目**需要显示声明所需要引入的依赖**，若不声明则不引入
*   子项目声明了依赖且**未声明版本号和scope**，则会继承父项目的**版本号和scope**，否则**覆盖**

##### 参考：[http://blog.csdn.net/liutengteng130/article/details/46991829](https://link.jianshu.com?t=http%3A%2F%2Fblog.csdn.net%2Fliutengteng130%2Farticle%2Fdetails%2F46991829)

##### 参考：[https://www.cnblogs.com/mr-wuxiansheng/p/6189438.html](https://link.jianshu.com?t=https%3A%2F%2Fwww.cnblogs.com%2Fmr-wuxiansheng%2Fp%2F6189438.html)
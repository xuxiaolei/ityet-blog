---
layout: post
title: Maven依赖机制与调节方案
categories: Maven
description: Maven进阶指南：依赖机制与调节方案
index_img: /img/post_def.png
date:  2021-06-12 09:09:09
tags: [Maven,Java]
---
Maven由Java语言编写，基于**微内核架构**和**可扩展插件机制**，是一款优秀且成熟的项目管理工具。经过十几年完善和发展，Maven在Java服务端项目管理上已经成为事实上的标准工具。

在Maven出现之前，Java语言的项目管理工具一直由Ant统治着；在此之后，又有Gradle逐渐在Android项目中作为配套打包工具流行开来。在目前看来，Maven依旧是Java服务器端项目管理工具中的王者。

因此，每一位高级工程师或软件架构师，都应该至少具备以下两项Maven技能：

*   熟练使用Maven构建项目
*   排查并调解项目依赖冲突

## **#1 依赖机制**

**1.1 依赖传递**

依赖传递的发生有两种情况：一种是存在模块之间的**继承**关系，在继承父模块后同时引入了父模块中的依赖，可通过**可选依赖机制**放弃依赖传递到子模块；另一种是引包时附带引入该包所依赖的包，该方式是***引起依赖冲突的主因***。

除了包传递之外，依赖传递还可以传递其它pom元素。以下是一个较为常见的pom文件，该文件中能够传递的元素有<inceptionYear/>、<developers/>、<properties>等。读者可自行查阅相关资料来获取所有的可传递依赖元素。

```xml
<groupId>com.github.miofy.examples</groupId>
<artifactId>examples-maven-project</artifactId>
<packaging>pom</packaging>
<version>0.1-SNAPSHOT</version>
<name>examples-maven-project</name>
<description>Parent Project</description>
<inceptionYear>2019</inceptionYear>
<developers>
  <developer>
    <name>miofy</name>
    <email>limiaofei@51dojo.com</email>
  </developer>
</developers>
<modules>
  <module>examples-maven-module-a</module>
  <module>examples-maven-module-b</module>
</modules>
<properties>
  <jersey.version>2.28</jersey.version>
  <junit.version>4.12</junit.version>
</properties>
<dependencies>
  <dependency>
    <groupId>org.glassfish.jersey.core</groupId>
    <artifactId>jersey-server</artifactId>
    <!-- 依赖范围：全阶段(编译、测试、运行)均有效，compile是默认选项-->
    <scope>compile</scope>
  </dependency>
  <dependency>
     <groupId>org.glassfish.jersey.containers</groupId>
     <artifactId>jersey-container-grizzly2-http</artifactId>
     <!-- 依赖排除 -->
     <exclusions>
       <exclusion>
         <groupId>org.glassfish.hk2.external</groupId>
         <artifactId>jakarta.inject</artifactId>
       </exclusion>
     </exclusions>
  </dependency>
  <dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-dbcp2</artifactId>
    <version>2.6.0</version>
    <!-- 依赖范围：仅在运行阶段有效 -->
    <scope>runtime</scope>
  </dependency>
  <dependency>
     <groupId>servlet</groupId>
     <artifactId>servlet-api</artifactId>
     <version>3.1</version>
     <!-- 依赖范围：编译和测试阶段均有效，运行时由其它的web容器或Java框架提供该包 -->
     <scope>provided</scope>
  </dependency>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>${junit.version}</version>
    <!-- 依赖范围: 仅在测试阶段有效 -->
    <scope>test</scope>
    <!-- 关闭依赖传递，默认是开启依赖传递的，即子模块可继承父模块依赖 -->
    <optional>false</optional>
  </dependency>
</dependencies>
<!-- 依赖管理: 主要用于统一模块的版本号，避免出现多版本共存 -->
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.glassfish.jersey</groupId>
      <artifactId>jersey-bom</artifactId>
      <version>${jersey.version}</version>
      <type>pom</type>
      <!-- 依赖导入: 引入其它项目的管理依赖 -->
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

**1.2 依赖范围**

项目的编译、测试和运行都会对应各自的一套类路径(classpath)，不同的类路径引入的包依赖也不同。Maven就是通过依赖范围来控制引包情况的。

```xml
<dependency>
  ...
  <scope>compile/provided/runtime/test/import/system</scope>
</dependency>
```

**1.3 依赖优化**

实际上Maven是比较“智能”的，它能够自动解析直接依赖和传递性依赖，根据预定义规则判断依赖范围的合理性，也可以对部分依赖进行适当调整来保证构件版本唯一。

即使这样，还会有些情况使Maven误判，因此手工进行依赖优化还是相当有必要的。读者可以使用maven-dependency-plugin提供的三个目标来实现依赖分析：

```bash
$ mvn dependency:list
$ mvn dependency:tree
$ mvn dependency:analyze
```

若读者还需更精细的分析结果，可以在命令后使用诸如以下参数：

```text
-Dverbose
-Dincludes=<groupId>:<artifactId>
```

**1.4 依赖调解**

依赖调解遵循以下两大原则：路径最短优先、声明顺序优先

*   第一原则：**路径最近者优先**

把当前模块当作顶层模块，直接依赖的包则作为次层模块，间接依赖的包则作为次层模块的次层模块，依次递推...，最后构成一棵引用依赖树。

假设当前模块是A，两种依赖路径如下所示：

```text
A --> B --> X(1.1)         // dist(A->X) = 2
A --> C --> D --> X(1.0)   // dist(A->X) = 3
```

此时，Maven可以按照第一原则自动调解依赖，结果是使用X(1.1)作为依赖。

*   第二原则：**第一声明者优先**

若冲突依赖的路径长度相同，那么第一原则就无法起作用了。

假设当前模块是A，两种依赖路径如下所示：

```text
A --> B --> X(1.1)   // dist(A->X) = 2
A --> C --> X(1.0)   // dist(A->X) = 2
```

当路径长度相同，则需要根据A直接依赖包在pom文件中的先后顺序来判定使用那条依赖路径，如果次级模块相同则向下级模块推，直至可以判断先后位置为止。

```xml
<!-- A pom.xml -->
<dependencies>
    ...
    dependency B
    ...
    dependency C
</dependencies>
```

假设依赖B位置在依赖C之前，则最终会选择X(1.1)依赖。

*   其它情况：**覆盖策略**

若相同类型但版本不同的依赖存在于同一个pom文件，依赖调解两大原则都不起作用，需要采用**覆盖策略**来调解依赖冲突，最终会引入最后一个声明的依赖。

```xml
<!-- 该pom文件最终引入commons-cli:commons-cli:1.3.jar依赖包。 -->

<dependencies>
  <dependency>
    <groupId>commons-cli</groupId>
    <artifactId>commons-cli</artifactId>
    <version>1.2</version>
  </dependency>
  <dependency>
    <groupId>commons-cli</groupId>
    <artifactId>commons-cli</artifactId>
    <version>1.4</version>
  </dependency>
  <dependency>
    <groupId>commons-cli</groupId>
    <artifactId>commons-cli</artifactId>
    <version>1.3</version>
  </dependency>
</dependencies>
```

## **#2 依赖解调三板斧**

**2.1 问题定位**

若启动应用时出现以下异常错误信息，很可能是发生了依赖冲突。

```text
NoClassDefFoundError
NoSuchMethodError
ClassNotFoundException
```

**2.2 依赖排查**

根据异常提示信息，找到相应类，定位该类所在包。使用以下命名显示该包涉及到的依赖树。

```text
$ mvn clean dependency:tree -Dverbose -Dincludes=<groupId>:<artifactId>
```

依赖树是以当前模块作为顶层节点，引入的其它模块作为子节点，一般的项目都会存在多层级依赖情况。查看依赖树时需要重点关注**包冲突**和**包重复**两个部分。

```text
Part 1： omitted for conflict with XXX
Part 2： omitted for duplicate
```

如果是IDEA旗舰版用户，还可以使用Diagram来分析依赖，重点关注依赖图中**红线**连接部分，那里可能是发生依赖冲突的地方。这种方式虽然直观，但是依赖包过多时排查难度陡增。

![](https://pic2.zhimg.com/v2-7258669f759f990b7a9abf32a1af4181_b.jpg)

![](https://pic2.zhimg.com/80/v2-7258669f759f990b7a9abf32a1af4181_1440w.jpg)

IDEA Pom Diagram

还有一种方式是安装IDEA插件市场中提供的依赖分析插件。操作界面简单，排查冲突很方便。

![](https://pic2.zhimg.com/v2-f2b5b4f08b001cf2c72b626e049e4b55_b.jpg)

![](https://pic2.zhimg.com/80/v2-f2b5b4f08b001cf2c72b626e049e4b55_1440w.jpg)

Maven Dependency Plugin for IDEA user

![](https://pic1.zhimg.com/v2-0bdc2bc54bad42f12c880a55166d9280_b.jpg)

![](https://pic1.zhimg.com/80/v2-0bdc2bc54bad42f12c880a55166d9280_1440w.jpg)

Dependecy Analyzer

**2.3 解决冲突**

冲突解决方式简单粗暴，直接在pom文件中排除冲突依赖即可。

```xml
<dependency>
  <groupId>org.glassfish.jersey.containers</groupId>
  <artifactId>jersey-container-grizzly2-http</artifactId>
  <!-- 剔除依赖 -->
  <exclusions>
    <exclusion>
      <groupId>org.glassfish.hk2.external</groupId>
      <artifactId>jakarta.inject</artifactId>
    </exclusion>
    ...
  </exclusions>
</dependency>
```
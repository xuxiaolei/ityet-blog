---
layout: post
title: 在pom文件中引入本地jar包
date: '2024-11-08 10:26:31'
permalink: /post/introduce-the-local-jar-package-in-the-pom-file-z1eaxgm.html
published: true
---

# 在pom文件中引入本地jar包

### 方法一

去maven仓库下载jar包，mvn仓库地址： [https://mvnrepository.com/](https://mvnrepository.com/)

​![image](https://ityet.com/img/image-20241108103501-afi8t0k.png)​

点击需要的版本

​![image](https://ityet.com/img/image-20241108103414-x5cbq79.png)​

以jar包的形式下载

​![image](https://ityet.com/img/image-20241108103403-5gf1fgl.png)​

在pom文件中添加jar依赖

```pom
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>4.0.3</version>
    <scope>system</scope>
    <systemPath>C:/test/maven/easyexcel-4.0.3.jar</systemPath>
</dependency>
```

### 方法二

若把jar包下载到本地，不知道jar包的groupId, artifactId等信息，则可采用此方法。  
新建lib包，之后导入

> eclipse直接build path
>
> idea：File→project structure→libraries→+jar包

​​![image](https://ityet.com/img/image-20241108103722-hnessji.png)​  
选择java,然后选择本地的jar即可。

### 方法三

jar包下载到本地后，不清楚对应的groupId, artifactId等信息，还是想通过pom文件的方式导入，则可采用此方法  
(1)：在项目下创建lib目录，将需要引入的jar包复制进去  
​​![image](https://ityet.com/img/image-20241108103805-c1l1re3.png)​  
(2) pom.xml文件中引入

```bash
 <dependency>
  <groupId>com.netflix.servo</groupId>
   <artifactId>servo-core</artifactId>
   <version>0.12.21</version>
   <scope>system</scope>
   <systemPath>${project.basedir}/src/main/resources/servo-core-0.12.21.jar</systemPath>
 </dependency>
```

注意：

> * groupId：自定义
> * artifactId：自定义
> * version：自定义
> * scope：必须是system
> * systemPath：jar包的路径（idea编写的时候会有提示的）

通过上述方式，在开发环境没有什么问题，不会存在什么包找不到等情况

但是，**maven** project部署一般打包发布，所以打包是需要额外配置的

(3) 配置打包参数

打包的时候需要做如下配置，需要通过resource标签引入，位置build→resources→resource

```bash
<build>
   <resources>
    <resource>
      <directory>lib</directory>
      <targetPath>/BOOT-INF/lib/</targetPath>
      <includes>
        <include>**/*.jar</include>
      </includes>
    </resource>
   </resources>
 </build>
```

* directory：指定lib文件夹的位置，由于是相对工程根目录，所以直接写上lib即可
* targetPath：打包到的文件夹位置，写上BOOT-INF/lib即可，或者是WEB-INF/lib。【斜杠（/）加不加都行，如果是mac的话写./】
* includes：一般都是以jar结尾，就写**/*.jar

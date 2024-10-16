---
layout: post
title: PropertySource配置方式深入了解
categories: Java
description: PropertySource配置方式深入了解
index_img: /img/post_def.png
date: 2020-10-30 09:09:09
tags: [PropertySource]
---
## @PropertySource功能
 
   1. 加载指定的属性文件（*.properties）到 Spring 的 Environment 中。可以配合 @Value 和
   @ConfigurationProperties 使用。
   
   2. @PropertySource 和 @Value
   组合使用，可以将自定义属性文件中的属性变量值注入到当前类的使用@Value注解的成员变量中。
   
   3. @PropertySource 和 @ConfigurationProperties
   组合使用，可以将属性文件与一个Java类绑定，将属性文件中的变量值注入到该Java类的成员变量中。


## 源码学习
---

让我们一起看下`@PropertySourc`e的源码如下：

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented@Repeatable(PropertySources.class)
public @interface PropertySource { 
    /** 
     * 资源的名称 
     */ 
    String name() default "";
    /** 
     * 资源文件路径，可以是数据多个文件地址 
     * 可以是classpath地址如： 
     * "classpath:/com/myco/app.properties" 
     * 也可以是对应的文件系统地址如： 
     * "file:/path/to/file" 
     */ 
    String[] value(); 
    /** 
     * 是否忽略文件资源是否存在，默认是false,也就是说配置不存在的文件地址spring启动将会报错 
     */ 
    boolean ignoreResourceNotFound() default false; 
    /** 
     * 这个没什么好说的了就是对应的字符编码了，默认是空值，如果配置文件中有中文应该设置为utf-8 
     */ 
    String encoding() default ""; 
    /** 
     * 关键的元素了 读取对应资源文件的工厂类了 默认的是PropertySourceFactory 
     */ 
    Class<? extends PropertySourceFactory> factory() default PropertySourceFactory.class;}
```

## 配置实践
---
1. 第一种配置方式：
现在我把资源文件的路径放在application.properties里

```
config.path=/home/myservice/config.properties
```

```
@PropertySource(value = {"file:${config.path}"}, encoding="utf-8")
public class MyConfig {
    @Value("${myconfig.index}")
    private String index;

    public String getIndex() {
       return index;
    }}
```
配置文件中 配置文件是绝对路径时 用:file

1. 第二种配置方式：classpath路径

```
@PropertySource("classpath:/document.properties")
public class MyConfig {
    @Value("${myconfig.index}")
    private String index;

    public String getIndex() {
       return index;
    }
}
```
classpath路径下就是可以用第二种方式springboot 

## 文件加载顺序 
---
一般默认加载application.propertise 或者 application.yml文件
*jar包目录下*
1.config
2.jar同级目录
*class目录下*
3.config目录
4.class
同级目录相同属性，按照优先级最高的匹配
另外：可以在启动命令中 加入 --spring.config.location 指定文件地址 多个文件用 ","隔开

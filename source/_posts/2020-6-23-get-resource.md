---
layout: post
title: 静态方法中的getResourceAsStream
categories: Java
description: 静态方法中的getResourceAsStream
index_img: /img/post_def.png
date: 2020-06-23 09:09:09
tags: [Java,Resource]
---
要以静态方法调用`getResourceAsStream` ，我们使用`ClassName.class`而不是`getClass()`

1.非静态方法

```java
getClass().getClassLoader().getResourceAsStream("config.properties"))
```

2.静态方式

```java
ClassName.class.class.getClassLoader().getResourceAsStream("config.properties"))
```

## 1.非静态方法

项目类路径中的`.properties`文件。

src/main/resources/config.properties

```bash
#config filejson.filepath = /Users/mkyong/Documents/workspace/SnakeCrawler/data/
```

FileHelper.java

```java
package com.mkyong;
import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;
public class FileHelper {
    public static void main(String[] args) {
        FileHelper obj = new FileHelper();
        System.out.println(obj.getFilePathToSave());
    }
    public String getFilePathToSave() {
        Properties prop = new Properties();
        String result = "";
        try (InputStream inputStream = getClass()
                .getClassLoader().getResourceAsStream("config.properties")) {
            prop.load(inputStream);
            result = prop.getProperty("json.filepath");
        } catch (IOException e) {
            e.printStackTrace();
        }
        return result; 
    }
}
```

输出量

```bash
/Users/mkyong/Documents/workspace/SnakeCrawler/data/
```

## 2.静态方法

如果将方法`getFilePathToSave()`转换为静态方法，则`getClass()`方法将失败，并提示*无法从对象类型对非静态方法getClass（）进行静态引用。*

**要解决此问题** ， `getClass()`更新为`ClassName.class`

FileHelper.java

```java
package com.mkyong;
import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;
public class FileHelper { 
    public static void main(String[] args) {
        System.out.println(getFilePathToSaveStatic());
    }
    public static String getFilePathToSaveStatic() {
        Properties prop = new Properties();
        String result = "";
        try (InputStream inputStream = FileHelper.class	
                .getClassLoader().getResourceAsStream("config.properties")) {
            prop.load(inputStream);
            result = prop.getProperty("json.filepath");
        } catch (IOException e) { 
            e.printStackTrace();
        } 
        return result;
    }
}
```

输出量

```bash
/Users/mkyong/Documents/workspace/SnakeCrawler/data/
```

## 参考文献

*   [Java属性文件示例](https://blog.csdn.net/java/java-properties-file-examples/)
*   [ClassLoader JavaDocs](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html)
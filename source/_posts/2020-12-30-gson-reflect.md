---
layout: post
title: Gson反序列化泛型
categories: Java
description: Gson反序列化泛型
keywords: Gson,Json
---

Java对象和Json之间的互转，一般用的比较多的两个类库是Jackson和Gson，下面记录一下Gson的学习使用。

# 基础概念

Serialization:序列化，使Java对象到Json字符串的过程。
Deserialization：反序列化，字符串转换成Java对象

**JSON数据中的`JsonElement`有下面这四种类型**

>JsonPrimitive —— 例如一个字符串或整型
>JsonObject—— 一个以 JsonElement 名字（类型为 String）作为索引的集合。也就是说可以把 JsonObject 看作值为 JsonElement 的键值对集合。
>JsonArray—— JsonElement 的集合。注意数组的元素可以是四种类型中的任意一种，或者混合类型都支持。
>JsonNull—— 值为null

# 依赖引用
使用Maven管理Gson，pom.xml导入gson的依赖
```xml
  <dependency>
     <groupId>com.google.code.gson</groupId>
     <artifactId>gson</artifactId>
     <version>2.3.1</version>
  </dependency>
```


# 基本语法
Gson的两个基础方法
```java
toJson();
fromJson();
```
# Gson中的一些注解

###### 1 @SerializedName注解

该注解能指定该字段在JSON中对应的字段名称

```java
public class Box {

  @SerializedName("w")
  private int width;

  @SerializedName("h")
  private int height;

  @SerializedName("d")
  private int depth;

  // Methods removed for brevity
}

```

也就是说`{"w":10,"h":20,"d":30}` 这个JSON 字符串能够被解析到上面的width，height和depth字段中。

##### 2 @Expose注解

该注解能够指定该字段是否能够序列化或者反序列化，默认的是都支持（true）。

```tsx
public class Account {

  @Expose(deserialize = false)
  private String accountNumber;

  @Expose
  private String iban;

  @Expose(serialize = false)
  private String owner;

  @Expose(serialize = false, deserialize = false)
  private String address;

  private String pin;
}

```

需要注意的通过 `builder.excludeFieldsWithoutExposeAnnotation()`方法是该注解生效。

```dart
  final GsonBuilder builder = new GsonBuilder();
    builder.excludeFieldsWithoutExposeAnnotation();
    final Gson gson = builder.create();

```

##### 3 @Since和@Until注解

Since代表“自从”，Until 代表”一直到”。它们都是针对该字段生效的版本。比如说 `@Since(1.2)`代表从版本1.2之后才生效，`@Until(0.9)`代表着在0.9版本之前都是生效的。

```kotlin
public class SoccerPlayer {

  private String name;

  @Since(1.2)
  private int shirtNumber;

  @Until(0.9)
  private String country;

  private String teamName;

  // Methods removed for brevity
}

```

也就是说我们利用方法`builder.setVersion(1.0)`定义版本1.0，如下：

```dart
 final GsonBuilder builder = new GsonBuilder();
    builder.setVersion(1.0);

    final Gson gson = builder.create();

    final SoccerPlayer account = new SoccerPlayer();
    account.setName("Albert Attard");
    account.setShirtNumber(10); // Since version 1.2
    account.setTeamName("Zejtun Corinthians");
    account.setCountry("Malta"); // Until version 0.9

    final String json = gson.toJson(account);
    System.out.printf("Serialised (version 1.0)%n  %s%n", json);

```

由于`shirtNumber`和`country`作用版本分别是1.2之后，和0.9之前，所以在这里都不会得到序列化，所以输出结果是：

```bash
Serialised (version 1.0)
  {"name":"Albert Attard","teamName":"Zejtun Corinthians"}

```

---

# Gson 序列化

英文Serialize和format都对应序列化，这是一个Java对象到JSON字符串的过程。
接着看一个例子,下面分别是java类和以及我们期望的JSON数据：

```tsx
public class Book {
  private String[] authors;
  private String isbn10;
  private String isbn13;
  private String title;
  //为了代码简洁，这里移除getter和setter方法等

}

```

```json
{
  "title": "Java Puzzlers: Traps, Pitfalls, and Corner Cases",
  "isbn-10": "032133678X",
  "isbn-13": "978-0321336781",
  "authors": [
    "Joshua Bloch",
    "Neal Gafter"
  ]
}

```

你肯定能发现JSON数据中出现了`isbn-10`和`isbn-13`, 我们怎么把字段数据`isbn10`和`isbn13`转化为JSON数据需要的`isbn-10`和`isbn-13`,Gson当然为我们提供了对应的解决方案

##### 1 序列化方案1

采用上面提到的`@SerializedName`注解。

```tsx
public class Book {
  private String[] authors;

  @SerializedName("isbn-10")
  private String isbn10;

  @SerializedName("isbn-13")
  private String isbn13;
  private String title;
  //为了代码简洁，这里移除getter和setter方法等

}

```

##### 2 序列化方案2

利用`JsonSerializer`类

```java
public class BookSerialiser implements JsonSerializer {

    @Override
    public JsonElement serialize(final Book book, final Type typeOfSrc, final JsonSerializationContext context) {

        final JsonObject jsonObject = new JsonObject();
        jsonObject.addProperty("title", book.getTitle());
        jsonObject.addProperty("isbn-10", book.getIsbn10());
        jsonObject.addProperty("isbn-13", book.getIsbn13());

        final JsonArray jsonAuthorsArray = new JsonArray();
        for (final String author : book.getAuthors()) {
            final JsonPrimitive jsonAuthor = new JsonPrimitive(author);
            jsonAuthorsArray.add(jsonAuthor);
        }
        jsonObject.add("authors", jsonAuthorsArray);

        return jsonObject;
    }
}

```

下面对序列化过程进行大致的分析：

*   JsonSerializer是一个接口，我们需要提供自己的实现，来满足自己的序列化要求。

```java
public interface JsonSerializer<T> {

  /**
   *Gson 会在解析指定类型T数据的时候触发当前回调方法进行序列化
   *
   * @param T 需要转化为Json数据的类型，对应上面的Book
   * @return 返回T指定的类对应JsonElement
   */
  public JsonElement serialize(T src, Type typeOfSrc, JsonSerializationContext context);
}

```

*   首先在上面的代码中，我们需要创建的是一个JsonElement对象，这里对应Book是一个对象，所以创建一个JsonObject类型。
    `final JsonObject jsonObject = new JsonObject();`
*   然后我们将相应字段里面的数据填充到jsonObject里面。

```css
jsonObject.addProperty...
jsonObject.add...

```

下面是jsonObject中的添加方法：

![](https://upload-images.jianshu.io/upload_images/1833901-12785eda5dc13763.png?imageMogr2/auto-orient/strip|imageView2/2/w/554/format/webp)

*   所以最后返回的还是一个JsonElement 类型，这里对应的是jsonObject。完成了javaBean\->JSON数据的转化。

*   同样需要配置,

```java
// Configure GSON
    final GsonBuilder gsonBuilder = new GsonBuilder();
    gsonBuilder.registerTypeAdapter(Book.class, new BookSerialiser());
    gsonBuilder.setPrettyPrinting();
    final Gson gson = gsonBuilder.create();

    final Book javaPuzzlers = new Book();
    javaPuzzlers.setTitle("Java Puzzlers: Traps, Pitfalls, and Corner Cases");
    javaPuzzlers.setIsbn10("032133678X");
    javaPuzzlers.setIsbn13("978-0321336781");
    javaPuzzlers.setAuthors(new String[] { "Joshua Bloch", "Neal Gafter" });

    // Format to JSON
    final String json = gson.toJson(javaPuzzlers);
    System.out.println(json);

```

，这里对应的是
`gsonBuilder.registerTypeAdapter(Book.class, new BookSerialiser())`方法进行JsonSerializer的配置。在上面例子中，通过调用`gsonBuilder.setPrettyPrinting();`方法还告诉了 Gson 对生成的 JSON 对象进行格式化

---

# Gson 反序列化

英文parse和deserialise对应反序列化，这是一个字符串转换成Java对象的过程。
我们同样采用上面一小节的代码片段，只不过现在我们需要做的是将：

```json
{
  "title": "Java Puzzlers: Traps, Pitfalls, and Corner Cases",
  "isbn-10": "032133678X",
  "isbn-13": "978-0321336781",
  "authors": [
    "Joshua Bloch",
    "Neal Gafter"
  ]
}

```

转化为对应的Book实体类，

##### 1 反序列化方案1

利用`@SerializedName` 注解
也就是说我们的实体类Book.java可以这么写：

```tsx
public class Book {
  private String[] authors;

  @SerializedName("isbn-10")
  private String isbn10;

  @SerializedName(value = "isbn-13", alternate = {"isbn13","isbn.13"})
  private String isbn13;
  private String title;
  //为了代码简洁，这里移除getter和setter方法等

}

```

![](https://upload-images.jianshu.io/upload_images/1833901-ab4b02fb8545cade.png?imageMogr2/auto-orient/strip|imageView2/2/w/790/format/webp)

> 可以看到这里我们在`@SerializedName` 注解使用了一个`value`, `alternate`字段,`value`也就是默认的字段，对序列化和反序列化都有效，`alternate`只有反序列化才有效果。也就是说一般服务器返回给我们JSON数据的时候可能同样的一个图片，表示"image","img","icon"等，我们利用`@SerializedName` 中的`alternate`字段就能解决这个问题，全部转化为我们实体类中的图片字段。

##### 2 反序列化方案2

我们在序列化的时候使用的是`JsonSerialize` ,这里对应使用`JsonDeserializer`
我们将解析到的json数据传递给Book的setter方法即可。

```dart
public class BookDeserializer implements JsonDeserializer<Book> {

  @Override
  public Book deserialize(final JsonElement json, final Type typeOfT, final JsonDeserializationContext context)
      throws JsonParseException {
    final JsonObject jsonObject = json.getAsJsonObject();

    final JsonElement jsonTitle = jsonObject.get("title");
    final String title = jsonTitle.getAsString();

    final String isbn10 = jsonObject.get("isbn-10").getAsString();
    final String isbn13 = jsonObject.get("isbn-13").getAsString();

    final JsonArray jsonAuthorsArray = jsonObject.get("authors").getAsJsonArray();
    final String[] authors = new String[jsonAuthorsArray.size()];
    for (int i = 0; i < authors.length; i++) {
      final JsonElement jsonAuthor = jsonAuthorsArray.get(i);
      authors[i] = jsonAuthor.getAsString();
    }

    final Book book = new Book();
    book.setTitle(title);
    book.setIsbn10(isbn10);
    book.setIsbn13(isbn13);
    book.setAuthors(authors);
    return book;
  }
}

```

和Gson序列化章节一样，我们这里接着分析我们是怎么将JSON数据解析（反序列化）为实体类的：

*   因为我们可以发现上面的JSON数据是一个`{}`大括号包围的，也就意味着这是一个Json对象。所以首先我们通过
    `final JsonObject jsonObject = json.getAsJsonObject();`将我们的JsonElement转化为JsonObject
*   通过`jsonObject.get("xxx").getAsString()`的形式获取相应String的值
*   通过`jsonObject.get("xx").getAsJsonArray();`获取相应的json数组，并遍历出其中的相应字段值
*   通过setter方法，将获取到的值设置给Book类。
*   最终返回的是 Book的对象实例。完成了JSON\->javaBean的转化
*   同样需要配置
*   关于从本地流中读取Json数据可以使用 `InputStreamReader`完成

```kotlin
 // Configure Gson
    GsonBuilder gsonBuilder = new GsonBuilder();
    gsonBuilder.registerTypeAdapter(Book.class, new BookDeserializer());
    Gson gson = gsonBuilder.create();

    // The JSON data
    try(Reader reader = new InputStreamReader(Main.class.getResourceAsStream("/part1/sample.json"), "UTF-8")){

      // Parse JSON to Java
      Book book = gson.fromJson(reader, Book.class);
      System.out.println(book);
    }
```

# 泛类型反序列化方案
```java
import com.google.common.reflect.TypeToken;
import com.google.gson.Gson;
import com.google.gson.JsonSyntaxException;
import java.lang.reflect.Type;


public class JsonHelper {

    public static void main(String[] args) {
        String respStr = "{\"status\":0,\"msg\":\"\",\"data\":{\"cardNo\":\"12345677\"}}";
        BaseResponse<CardInfo> cardInfo = parseResponse(respStr);
        System.out.println(cardInfo.data);
        respStr = "{\"status\":0,\"msg\":\"\",\"data\":{\"id\":\"123445678\",\"name\":\"张三\"}}";
        BaseResponse<UserInfo> userInfo = parseResponse(respStr);
        System.out.println(userInfo.data);
    }

    private static <T> BaseResponse<T> parseResponse(String responseData) throws JsonSyntaxException {

        Gson gson = new Gson();
        Type jsonType = new TypeToken<BaseResponse<T>>() {
        }.getType();
        BaseResponse<T> result = gson.fromJson(responseData, jsonType);

        return result;
    }

    class BaseResponse<T> {


        private int status;

        public int getStatus() {
            return status;
        }

        public void setStatus(int status) {
            this.status = status;
        }

        public String getMsg() {
            return msg;
        }

        public void setMsg(String msg) {
            this.msg = msg;
        }

        public T getData() {
            return data;
        }

        public void setData(T data) {
            this.data = data;
        }

        private String msg;


        private T data;


    }

    class UserInfo {
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "UserInfo{" +
                    "name='" + name + '\'' +
                    ", Id='" + Id + '\'' +
                    '}';
        }

        public String getId() {
            return Id;
        }

        public void setId(String id) {
            Id = id;
        }

        private String Id;
    }

    class CardInfo {
        public String getCardNo() {
            return cardNo;
        }

        public void setCardNo(String cardNo) {
            this.cardNo = cardNo;
        }

        @Override
        public String toString() {
            return "CardInfo{" +
                    "cardNo='" + cardNo + '\'' +
                    '}';
        }

        private String cardNo;
    }
}

```

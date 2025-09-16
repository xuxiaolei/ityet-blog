---
layout: post
title: 前端Long型ID精度丢失解决
description: ""
date: 2025-08-28T08:40:47.388Z
preview: ""
published: true
tags: []
categories: []
---

#### 引言

在前后端分离的时代，我们的生活充满了无数的机遇与挑战——包括那些突然冒出来的让人抓狂的 Bug。今天我们要聊的，就是一个让无数开发者哭笑不得的经典问题：**后端 `Long` 类型 ID 过长导致前端精度丢失**。说到这个问题，那可真是“万恶之源”啊，谁让 `JavaScript` 只能安全地处理 `Number.MAX_SAFE_INTEGER`（也就是 9007199254740991）以内的数值呢？

如果你曾经为了这个问题而冥思苦想，别担心，你不是一个人。今天，我们不仅要用幽默的方式来剖析这个“世纪难题”，还要带你从根源上解决它，让你的代码不再“失精”。

#### 问题背景：为什么 `Long` 这么长？

首先，我们得从 `Long` 类型说起。`Long`，就是 Java 中的 64 位整数类型，对于喜欢处理大数据、大数字的 Java 来说，这个类型简直就是福音。然而，前端世界却有点“孤陋寡闻”，它只懂得处理 53 位以内的整数。是的，你没听错，在这点上 `JavaScript` 就像是一个“慢半拍”的老学究，面对更大的数字时，它就开始摆弄小数点和指数，最后吐出一个让你感到绝望的数字。于是乎，精度丢失的问题就像幽灵一样，开始在你的项目中游荡。

#### 第一次相遇：精度丢失的那些事

故事得从某一天的 Bug 反馈开始：“开发哥哥，你看这个 ID 怎么变了样？这不是我数据库里的那个 ID 啊！”你皱着眉头一看，是的，`9223372036854775807` 变成了 `9223372036854776000`，哎呀，这多出来的数字简直像是魔术一样。明明后端给的是对的呀！你一边抓头一边心想：“这肯定是前端的锅！”

#### 前端的锅还是后端的锅？——追根溯源

其实吧，这个问题甩锅给前端也不是完全没道理。让我们来看看 `JavaScript` 在处理数字时的“短板”。`JavaScript` 的 `Number` 类型是基于 IEEE 754 标准的双精度浮点数格式，只能安全地表示 53 位二进制数字，也就是 `Number.MAX_SAFE_INTEGER` 的值——9007199254740991。

换句话说，超过这个范围的整数，`JavaScript` 就会开始“精度打折”，它的“脑容量”突然就不够用了。于是，你的 `Long` 类型 ID 就变成了它眼中的一堆没那么“性感”的数值。正因为如此，前端处理这些长得离谱的 ID 时，不得不“牺牲”一下，结果就是你那原本忠实的 ID 被截断，变成了一个“有趣”的新数字。

#### 从震惊到冷静：寻找解决方案

既然问题的根源已经找到，那就轮到我们这些开发者来大显身手了。接下来，我将带你深入了解几种解决方案，并告诉你每种方案的优缺点，毕竟条条大路通罗马。

##### 1. 直接转成字符串：简单粗暴却高效

面对这种问题，我们最先想到的肯定是最简单粗暴的方法：直接把 `Long` 类型的数据转换成字符串不就好了嘛！既然 `JavaScript` 是浮点数脑残粉，那我们干脆把问题丢回去，告诉它：“你只需要当这是个字符串，别担心它有多长！”

如何做到这一点呢？其实很简单，使用 `Jackson` 提供的 `ToStringSerializer`，我们可以轻松地把 `Long` 转成字符串。

代码语言：java

____

AI代码解释

复制

```java
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import com.fasterxml.jackson.databind.ser.std.ToStringSerializer;

public class UserDto {
    @JsonSerialize(using = ToStringSerializer.class)
    private Long id;
    // 其他字段
}
```

这样一来，当后端返回 `UserDto` 的时候，`Long` 类型的 `id` 会被序列化为字符串，前端接收到的也是字符串，精度问题迎刃而解。简直就像在 JavaScript 的脑袋上贴了一张便签：“这个是字符串哦，不用你操心！”

##### 2. 前端使用 `BigInt`：让大数也能精确运算

当然，简单粗暴的方法并不总是适合所有场景。想象一下，如果前端需要对这个 ID 进行某种数学运算，直接转成字符串可就不太好了。那么，前端该怎么处理这些“超长”的 ID 呢？

幸运的是，`JavaScript` 也不是一无是处。引入了 `BigInt` 之后，`JavaScript` 终于不再是那个只会摆弄小数点的呆子了。`BigInt` 是一种新的原始数据类型，专门用来处理任意精度的整数。你可以这样做：

代码语言：javascript

__代码运行次数：0

运行

____

AI代码解释

复制

```javascript
const id = BigInt("9223372036854775807");
console.log(id + 1n); // 输出：9223372036854775808n
```

这样，你就可以在前端精确地处理 `Long` 类型的数据，避免精度丢失的问题。当然，这里有一个小小的提醒：`BigInt` 并不是所有浏览器都支持的，所以你得确保你的应用环境能够兼容。

##### 3. 自定义序列化：复杂问题简单化

虽然上面的方法已经可以解决大多数问题，但有时候我们会遇到一些需要更细粒度控制的场景。这时候，`Jackson` 的自定义序列化器就派上用场了。

我们可以编写一个自定义的序列化器，根据需求灵活控制 `Long` 类型字段的序列化过程。比如，我们可以在序列化时决定是否将 `Long` 转换为字符串，或者针对特定条件进行不同处理：

代码语言：java

____

AI代码解释

复制

```java
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;

import java.io.IOException;

public class CustomLongSerializer extends JsonSerializer<Long> {

    @Override
    public void serialize(Long value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        if (value != null) {
            gen.writeString(value.toString());
        }
    }
}
```

然后在需要的地方应用这个自定义序列化器：

代码语言：java

____

AI代码解释

复制

```java
import com.fasterxml.jackson.databind.annotation.JsonSerialize;

public class UserDto {
    @JsonSerialize(using = CustomLongSerializer.class)
    private Long id;
    // 其他字段
}
```

这样，你就能掌控整个序列化过程，确保每一个 `Long` 都能按照你希望的方式被处理。当然，这种方法虽然灵活，但稍微复杂了一些，需要多写几行代码，也可能增加维护成本。

##### 4. 全局处理：省时省力的方案

如果你希望全局解决这个问题，省去在每个字段上配置注解的麻烦，可以考虑全局配置 `ObjectMapper`。通过在 `Spring Boot` 中配置全局的 `ObjectMapper`，你可以让所有的 `Long` 类型字段都自动转换为字符串。

代码语言：java

____

AI代码解释

复制

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.module.SimpleModule;

@Configuration
public class JacksonConfig {

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        SimpleModule module = new SimpleModule();
        module.addSerializer(Long.class, ToStringSerializer.instance);
        module.addSerializer(Long.TYPE, ToStringSerializer.instance);
        mapper.registerModule(module);
        return mapper;
    }
}
```

一劳永逸，不用担心忘记某个字段配置序列化器，整个项目都能享受到精度不丢失的快感。当然，使用全局配置的同时要注意，可能会影响到某些你不希望被转换的 `Long` 字段，因此需要谨慎考虑。

#### 实际案例：一波三折的 Bug 解决之路

为了让大家更好地理解这些解决方案的实际效果，我来分享一个真实项目中的故事。这个项目涉及用户 ID 的管理，由于用户量很大，后端采用了 `Long` 类型的唯一标识符。项目上线没几天，就出现了用户反馈：“我的 ID 怎么变了样？”。

###### 第一步：追查问题根源

我们首先检查了前端代码，发现 `JavaScript` 的 `Number` 类型确实无法准确表示这个长达 19 位的数字，于是导致了精度丢失。接着，我们查看了后端代码，发现虽然 `Long` 类型的数据在后端是正确的，但在通过 REST API 返回给前端时，数字的精度丢失了。

###### 第二步：选择合适的解决方案

为了快速解决问题，我们决定首先采用 `@JsonSerialize(using = ToStringSerializer.class)` 这个简单有效的办法。通过这个办法，我们成功地避免了前端接收精度丢失的数字。

然而，问题并没有完全解决。在后续的需求中，前端需要对 ID 进行某些运算，比如对用户的 ID 进行排序。这时候，字符串就显得有些力不从心了。

###### 第三步：使用 `BigInt` 解决前端运算问题

于是，我们决定在前端引入 `BigInt`。通过使用 `BigInt`，前端不仅能够精确地存储这些超长的 ID，还能进行必要的数学运算。经过测试，这种方法在各大主流浏览器上表现良好，唯一的缺点就是对一些旧版本浏览器的支持不太友好。

###### 第四步：最终的全局配置

为了避免今后类似问题再次发生，我们决定将 `ToStringSerializer` 配置成全局生效。这让所有的 `Long` 类型数据在序列化时都自动转换为字符串，既保证了前端的数据准确性，又减少了代码的重复配置。

#### 结语：精度丢失的终结者

从这个案例中可以看到，虽然 `Long` 类型的精度丢失问题看似简单，但在实际项目中可能带来诸多隐患。通过多种解决方案的对比和尝试，我们最终找到了适合自己项目的最佳方案。
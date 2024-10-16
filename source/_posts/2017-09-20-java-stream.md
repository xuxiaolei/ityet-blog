---
layout: post
title: Java Lambda表达式和Stream流操作
categories: Java
description: Java Lambda表达式和Stream流操作
index_img: 
date: 2017-09-20 09:09:09
tags: [Java]
---

# 

### Lambda表达式

Lambda是函数式接口的匿名实现类的实例对象，函数式接口使用`@FunctionalInterface`注解来限制接口只能有一个待实现的方法

#### 格式

*   \-> 箭头操作符
*   \->的左边是入参，入参的类型可以省略，多个入参需要添加`()`
*   \->的右边是Lambda体，就是实现类的方法体，如果只有一行可以省略掉`{}`

#### 简写格式

##### 无参 + 无返回值

```java
() -> System.out.println("xxx");
```

无参时入参需要写成`()`，当方法体只有一行时，可以省略掉`{}`

##### 有参 + 无返回值

```java
(String s1) -> System.out.println(s1);
(String s1, String s2) -> System.out.println(s1 + s2);

```

##### 入参数据类型省略

省略入参类型，会由编译器进行类型推断

```java
(s1) -> System.out.println(s1);
(s1, s2) -> System.out.println(s1 + s2);

```

##### 只有一个入参

```java
s1 -> System.out.println(s1);

```

##### 有返回值

方法体只有一行，可省略return

```java
i1 -> String.valueOf(i1);

```

直接方法引用，入参和return都可以省略

```java
String::valueOf

```

方法体有多行，`{}`和return都不可省略

```java
i1 -> {
  System.out.println(i1);
  return String.valueOf(i1);
}

```

### Stream流操作

这是一种高级的迭代器，用来操作集合，如查找和过滤等常见操作

Stream操作的过程分为：创建Stream对象 -> 中间操作 -> 终止操作

知识点：

1.  当执行了终止操作后，该流就不能再执行中间操作了
2.  在没有执行终止操作前，中间操作并不会执行，这个称为惰性
3.  中间操作分为：无状态操作和有状态操作两种
4.  无状态操作中每个元素都按照操作的顺序执行，并不是所有元素先完成一步操作后，再一起进入下一步操作
5.  有状态操作需要等待所有元素都执行完之前的中间操作后才会执行，比如元素被分组在不同的线程中进行无状态操作，当遇到排序这个操作时，就需要等待其他线程的操作都完成后才能执行排序操作
6.  串行流调用`parallel()`可变成并行流
7.  并行流调用`sequential()`可变成串行流
8.  多次连续调用`parallel()`和`sequential()`，只会保留最后一次调用，原因是调用这两个方法，只会改变当前流的`sourceStage.parallel`这个属性

#### 创建流

##### 串行流

```java
int[] nums = {1, 2, 3, 4, 5};
// 以下两个数组的类型任意
Arrays.stream(nums);
Stream.of(nums);
// 基本类型对应的Stream类.of()方法
IntStream.of(nums);
// 创建1-9的int流
IntStream.range(1, 10).forEach(System.out::println);
// 创建1-10的int流
IntStream.rangeClosed(1, 10).forEach(System.out::println);

```

##### 并行流

```java
int[] nums = {1, 2, 3, 4, 5};
// 可由串行流得到
IntStream.of(nums).parallel();
// 由集合得到
Arrays.asList(nums).parallelStream();

```

#### 中间操作

中间操作分为：无状态和有状态操作

###### 无状态

| 操作方法 | 说明 |
| --- | --- |
| map/mapToXxx | 对所有元素都做一些操作 |
| flatMap/flatMapToXxx | 对每个元素中的所有元素进行压平展开到当前流中 |
| filter | 过滤，输出满足条件的元素 |
| peek | 和forEach类似，常用于打印调试 |

map/mapToXxx，map能否改变元素类型要看map方法的入参

```java
int[] nums = {1, 2, 3, 4, 5};
// map对每个元素执行一次操作，不可改变元素的类型，因为此map入参是UnaryOperator<T>
IntStream.of(nums).map(it -> it + 1).forEach(System.out::println);
// 改变元素类型
IntStream.of(nums).mapToObj(it -> String.format("[%d]", it)).forEach(System.out::println);

// map可以改变元素类型，因为map入参是Function<T, R>
ArrayList<Test> list1 = Lists.newArrayList(new Test(1, "a"), new Test(2, "b"));
List<Test2> list2 = list1.parallelStream().map(it -> {
  	return new Test2(it.getId());
}).collect(Collectors.toList());

```

flatMap/flatMapToXxx

```java
@Data
@AllArgsConstructor
public static class Test {
  	private int id;
  	private int[] data;
}

ArrayList<Test> list1 = Lists.newArrayList(new Test(1, new int[]{1, 2, 3}), new Test(2, new int[]{4, 5, 6}));
list1.parallelStream()  // Stream<Test>流
        .flatMapToInt(it -> IntStream.of(it.getData()))  // 将data数组转成IntStream流
  			// 上面执行flatMapToInt之后得到一个完整的IntStream流，接下来可以进行IntStream流的一些操作了
        .map(it -> it + 1)
        .forEach(System.out::println);

```

filter

```java
int[] nums = {1, 2, 3, 4, 5};
IntStream.of(nums).filter(it -> it > 3).forEach(System.out::println);

```

peek

```java
int[] nums = {1, 2, 3, 4, 5};
IntStream.of(nums).peek(System.out::println).forEach(System.out::println);

```

###### 有状态

| 操作方法 | 说明 |
| --- | --- |
| distinct | 去重 |
| sorted | 排序 |
| limit | 截取 |
| skip | 跳过 |

#### 终止操作

| 操作方法 | 说明 |
| --- | --- |
| forEach/forEachOrdered | forEach不保证顺序，forEachOrdered则保证顺序 |
| collect/toArray | 收集到集合或数组 |
| reduce | 归约 |
| min/max/count | 求最小/最大和总数 |
| findAny/findFirst | 找到任意一个/找到第一个 |
| allMatch/anyMatch/noneMatch | 所有都匹配时返回true/任意一个匹配时返回true/没有一个匹配时返回true |

注意：并行流调用forEach是不保证顺序的

reduce

```java
int[] nums = {1, 2, 3, 4, 5};
// reduce第一个参数是初始值，第二个参数是BinaryOperator，输入两个数返回相加的结果，然后再和上一次的结果相加，最终得到整个流中所有元素相加的和
System.out.println(IntStream.of(nums).reduce(0, Integer::sum));

```

BigDecimal求和

```java
ArrayList<Test3> list3 = Lists.newArrayList(new Test3(1L, 1, 5.5), new Test3(2L, 2, 3.2));
BigDecimal reduce = list3.parallelStream()
        .map(it -> BigDecimal.valueOf(it.getPrice()).multiply(BigDecimal.valueOf(it.getQty())))
        .reduce(BigDecimal.ZERO, BigDecimal::add);
System.out.println(reduce);

```

#### 线程池

并行流默认使用自带的ForkJoinPool，线程数是CPU核心数，可以指定自定义的线程池

```java
int[] nums = {1, 2, 3, 4, 5};

// 使用默认的ForkJoinPool
IntStream.of(nums).parallel().forEach(it -> {
    System.out.println(Thread.currentThread().getName() + ":" + it);
});

// 自定义一个ForkJoinPool
ForkJoinPool pool = new ForkJoinPool(2);
pool.submit(() -> {
    IntStream.of(nums).parallel().forEach(it -> {
        System.out.println(Thread.currentThread().getName() + ":" + it);
    });
});

Thread.currentThread().join();
```
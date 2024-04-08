---
layout: post
title: Spring定时任务@Scheduled详解
categories: Java
description: Spring定时任务@Scheduled详解
keywords: Spring,Scheduled
---
在springMVC里使用spring的定时任务非常的简单,可以使用@scheduled定时任务的两种方式，一种是直接@scheduled（cron=”0 0 0 ? * * “），还有一种是使用xml配置

# 详细步骤
Spring配置文件xmlns加入
``` xml
xmlns:task="http://www.springframework.org/schema/task"
```

xsi:schemaLocation中加入
``` xml
http://www.springframework.org/schema/task
http://www.springframework.org/schema/task/spring-task-3.0.xsd"
```
Spring扫描注解的配置
``` xml
<context:component-scan base-package="com.imwoniu.*" />
```
任务扫描注解
``` xml
<task:executor id="executor" pool-size="5" />
<task:scheduler id="scheduler" pool-size="10" />
<task:annotation-driven executor="executor" scheduler="scheduler" />
```
代码实现：

注解@Scheduled 可以作为一个触发源添加到一个方法中，例如，以下的方法将以一个固定延迟时间5秒钟调用一次执行，这个周期是以上一个调用任务的完成时间为基准，在上一个任务完成之后，5s后再次执行：
``` java
@Scheduled(fixedDelay = 5000) 
public void doSomething() 
{ 
// something that should execute periodically
}
```

如果需要以固定速率执行，只要将注解中指定的属性名称改成fixedRate即可，以下方法将以一个固定速率5s来调用一次执行，这个周期是以上一个任务开始时间为基准，从上一任务开始执行后5s再次调用：
``` java
@Scheduled(fixedRate = 5000) 
public void doSomething() 
{
 // something that should execute periodically
}
```

如果简单的定期调度不能满足，那么cron表达式提供了可能

``` java
package com.imwoniu.task;
 import org.springframework.scheduling.annotation.Scheduled;
 import org.springframework.stereotype.Component;

@Component public class TaskDemo {

@Scheduled(cron = "0 0 2 * * ?")//每天凌晨两点执行
void doSomethingWith(){
logger.info("定时任务开始......"); long begin = System.currentTimeMillis(); //执行数据库操作了哦...

long end = System.currentTimeMillis();
logger.info("定时任务结束，共耗时：[" + (end-begin) / 1000 + "]秒");
}
}
```

关于Cron表达式
表达式网站生成:
[http://cron.qqe2.com/](http://cron.qqe2.com/) 直接点击

按顺序依次为
```
秒（0~59）
分钟（0~59）
小时（0~23）
天（月）（0~31，但是你需要考虑你月的天数）
月（0~11）
天（星期）（1~7 1=SUN 或 SUN，MON，TUE，WED，THU，FRI，SAT）
年份（1970－2099）
```
其中每个元素可以是一个值(如6),一个连续区间(9-12),一个间隔时间(8-18/4)(/表示每隔4小时),一个列表(1,3,5),通配符。由于"月份中的日期"和"星期中的日期"这两个元素互斥的,必须要对其中一个设置?.
```
0 0 10,14,16 * * ? 每天上午10点，下午2点，4点
0 0/30 9\-17 * * ? 朝九晚五工作时间内每半小时
0 0 12 ? * WED 表示每个星期三中午12点
"0012**?"每天中午12点触发
"01510?**"每天上午10:15触发
"01510**?"每天上午10:15触发
"01510**?*"每天上午10:15触发
"01510**?2005"2005年的每天上午10:15触发
"0*14**?"在每天下午2点到下午2:59期间的每1分钟触发
"00/514**?"在每天下午2点到下午2:55期间的每5分钟触发
"00/514,18**?"在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发
"00\-514**?"在每天下午2点到下午2:05期间的每1分钟触发
"010,4414?3WED"每年三月的星期三的下午2:10和2:44触发
"01510?*MON-FRI"周一至周五的上午10:15触发
"0151015*?"每月15日上午10:15触发
"01510L*?"每月最后一日的上午10:15触发
"01510?*6L"每月的最后一个星期五上午10:15触发
"01510?*6L2002\-2005"2002年至2005年的每月的最后一个星期五上午10:15触发
"01510?*6#3"每月的第三个星期五上午10:15触发
```
有些子表达式能包含一些范围或列表

例如：
```
子表达式（天（星期））可以为 “MON-FRI”，“MON，WED，FRI”，“MON-WED,SAT”
```
“*”字符代表所有可能的值
```
因此，“*”在子表达式（月）里表示每个月的含义，“*”在子表达式（天（星期））表示星期的每一天
```
“/”字符用来指定数值的增量

例如：
```
在子表达式（分钟）里的“0/15”表示从第0分钟开始，每15分钟

在子表达式（分钟）里的“3/20”表示从第3分钟开始，每20分钟（它和“3，23，43”）的含义一样
```
“？”字符仅被用于天（月）和天（星期）两个子表达式，表示不指定值
```
当2个子表达式其中之一被指定了值以后，为了避免冲突，需要将另一个子表达式的值设为“？”
```
“L” 字符仅被用于天（月）和天（星期）两个子表达式，它是单词“last”的缩写
但是它在两个子表达式里的含义是不同的。
```
在天（月）子表达式中，“L”表示一个月的最后一天

在天（星期）自表达式中，“L”表示一个星期的最后一天，也就是SAT
```
如果在“L”前有具体的内容，它就具有其他的含义了

例如：
```
“6L”表示这个月的倒数第６天，“ＦＲＩＬ”表示这个月的最一个星期五
```
注意：在使用“L”参数时，不要指定列表或范围，因为这会导致问题

| 字段 |  允许值 |  允许的特殊字符 | 
| `秒` | `0-59` |   `, - * /` |
| `分` |   `0-59` |   `, - * /` |
| `小时` |   `0-23` |   `, - * /` |
| `日期` |   `1-31` |   `, - * ? / L W C` |
| `月份` |   `1-12或者 JAN-DEC` |   `, - * /` |
| `星期` |   `1-7或者 SUN-SAT` |   `, - * ? / L C #` |
| `年（可选）` |   `留空, 1970-2099` |   `, - * /` |
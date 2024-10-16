---
layout: post
title: Newtonsoft.Json 序列化和反序列化
categories: C#
description: Newtonsoft.Json 序列化和反序列化
index_img: https://xn--6or.us.kg/api/?raw
date: 2015-08-06 09:09:09
tags: [C#, Json序列化, Json反序列化]
---

## 首先简单介绍一下什么是 Json ?
Json【javascript对象表示方法】，
它是一个轻量级的数据交换格式，我们可以很简单的来读取和写它，
并且它很容易被计算机转化和生成，它是完全独立于语言的。

** Json支持下面两种数据结构:

键值对的集合--各种不同的编程语言，都支持这种数据结构；
有序的列表类型值的集合--这其中包含数组，集合，矢量，或者序列，等等。
Json有下面几种表现形式:
1. 对象
-  一个没有顺序的“键/值”,一个对象以花括号“{”开始，并以花括号"}"结束，
- 在每一个“键”的后面，有一个冒号，并且使用逗号来分隔多个键值对。
> 例如：var user = {"name":"Manas","gender":"Male","birthday":"1987-8-8"}   
2. 数组
- 设置值的顺序，一个数组以中括号"["开始,并以中括号"]"结束，并且所有的值使用逗号分隔，
>例如：
>	var userlist = [
>			{"user":{"name":"Manas","gender":"Male","birthday":"1987-8-8"}}, 
>			{"user":{"name":"Mohapatra","Male":"Female","birthday":"1987-7-7"}}]
3. 字符串
- 任意数量的Unicode字符，使用引号做标记，并使用反斜杠来分隔。(注意: 引号  逗号  冒号  均为英文状态下半角符号, 且只能是双引号 )
>例如： var userlist = "{\"ID\":1,\"Name\":\"Manas\",\"Address\":\"India\"}" 

---

## C#中具体如何使用:           
 在C#中我们经常使用下面的工具来解析Json格式的内容
 Newtonsoft.Json，是.Net中开源的Json序列化和反序列化工具，
 官方地址：<http://www.newtonsoft.com/json>

** 具体使用:

1. 右键项目=>Nuget包管理=>添加  Newtonsoft.Json

2. 引入命名空间
> using Newtonsoft.Json;


** 使用方法和例子:
1. JSON序列化
```c#
string JsonStr= JsonConvert.SerializeObject(Entity);
```
eg:
```c#
A a=new A();
a.Name="Elain00";
a.Hobby="eat eat";
string jsonStr=JsonConvert.SerializeObject(a);
```

2. JSON反序列化
```c#
string jsonstr = "jsonString";
Class model = JsonConvert.DeserializeObject<Class>(jsonstr);
```
eg:
```c#
string JsonStr='"{\'Name\':\'Elaine00\',\'Hobby\':\'eat eat\'}";
A a=JsonConvert.DeserializeObject<A>(JsonStr); 
```

3. 时间格式处理 
```c#
IsoDateTimeConverter timeFormat = new IsoDateTimeConverter();
timeFormat.DateTimeFormat = "yyyy-MM-dd HH:mm:ss";
Response.Write(JsonConvert.SerializeObject(bll.GetModelList(strWhere), Newtonsoft.Json.Formatting.Indented, timeFormat)); 
```

4. 扩展方法
```c#
public static class NewtonJSONHelper
    {        
		public static string SerializeObject(this object obj)
        {            
			return JsonConvert.SerializeObject(obj, Formatting.Indented, new JsonSerializerSettings{
                ReferenceLoopHandling = ReferenceLoopHandling.Ignore});
        } 
		public static T DeserializeObject<T>(this string data)
        {            
			return JsonConvert.DeserializeObject<T>(data, new JsonSerializerSettings
            {
                ReferenceLoopHandling = ReferenceLoopHandling.Ignore
            });
        }
    }
```

5. 日期处理
```c#
public class LogEntry
{  
	public string Details { get; set; }  
	public DateTime LogDate { get; set; }
}
public void WriteJsonDates()
{
	LogEntry entry = new LogEntry
 	 {
   		 LogDate = new DateTime(2009, 2, 15, 0, 0, 0, DateTimeKind.Utc),
   		 Details = "Application started."
 	 };  
	// default as of Json.NET 4.5  
	string isoJson = JsonConvert.SerializeObject(entry);  
	// {"Details":"Application started.","LogDate":"2009-02-15T00:00:00Z"}
 	JsonSerializerSettings microsoftDateFormatSettings = new JsonSerializerSettings{
		DateFormatHandling = DateFormatHandling.Micropublic class LimitPropsContractResolver : DefaultContractResolver
    {        
		private string[] props = null;        
		public LimitPropsContractResolver(string[] props)
        {            
			this.props = props;
        }       
		protected override IList<JsonProperty> CreateProperties(Type type, MemberSerialization memberSerialization)
        {
            IList<JsonProperty> list = base.CreateProperties(type, memberSerialization);
            IsoDateTimeConverter iso = new IsoDateTimeConverter() { DateTimeFormat = "yyyy-MM-dd HH:mm:ss" };
            IList<JsonProperty> listWithConver = new List<JsonProperty>();            
			foreach (var item in list)
            {                
				if (props.Contains(item.PropertyName))
                {                   
					 if (item.PropertyType.ToString().Contains("System.DateTime"))
                    {
                        item.Converter = iso;
                    }
                    listWithConver.Add(item);
                }
            }            
			return listWithConver;
        }
    }
}
```

6. 自定义序列化的字段名称
默认情况下,Json.Net序列化后结果中的字段名称和类中属性的名称一致.
如果想自定义序列化后的字段名称,可以使用JsonProperty.例如：
 
```c#
public class Person
 {
       public int Id { get; set; }

       public string Name { get; set; }
 }
```

默认序列化的结果为: 

> {"Id":1,"Name":"杨过"}

如果不想用默认的字段名称，可以使用如下方式：

```c#
public class Person
{
    [JsonProperty(PropertyName = "PersonId")]
    public int Id { get; set; }

    [JsonProperty(PropertyName = "PersonName")]
    public string Name { get; set; }
}
```

这样序列化的结果为：
> {"PersonId":1,"PersonName":"杨过"}

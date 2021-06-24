---
title: iOS中JSON数据的解析
date: 2015-04-27
categories: iOS逻辑初窥
tags: []
---
## iOS中JSON数据解析

官方为我们提供的解析JSON数据的类是NSJSONSerialization，首先我们先来看下这个类的几个方法：

\+ (BOOL)isValidJSONObject:(id)obj;

判断一个数据对象是否可以转化为JSON数据

\+ (NSData *)dataWithJSONObject:(id)obj options:(NSJSONWritingOptions)opt error:(NSError **)error;

将JSON数据写为NSData数据，其中opt参数的枚举如下，这个参数可以设置，也可以不设置，如果设置，则会输出视觉美观的JSON数据，否则输出紧凑的JSON数据。

```
typedef NS_OPTIONS(NSUInteger, NSJSONWritingOptions) {
    NSJSONWritingPrettyPrinted = (1UL << 0)
}
```

\+ (id)JSONObjectWithData:(NSData *)data options:(NSJSONReadingOptions)opt error:(NSError **)error;

这个方法是解析中数据的核心方法，data是JSON数据对象，可以设置一个opt参数，具体用法如下：

```
typedef NS_OPTIONS(NSUInteger, NSJSONReadingOptions) {
    //将解析的数组和字典设置为可变对象
    NSJSONReadingMutableContainers = (1UL << 0),
    //将解析数据的子节点创建为可变字符串对象
    NSJSONReadingMutableLeaves = (1UL << 1),
    //允许解析对象的最上层不是字典或者数组
    NSJSONReadingAllowFragments = (1UL << 2)
}
```

\+ (NSInteger)writeJSONObject:(id)obj toStream:(NSOutputStream *)stream options:(NSJSONWritingOptions)opt error:(NSError **)error;

将JSON数据写入到输出流，返回的是写入流的字节数

\+ (id)JSONObjectWithStream:(NSInputStream *)stream options:(NSJSONReadingOptions)opt error:(NSError **)error;

从输入流读取JSON数据

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

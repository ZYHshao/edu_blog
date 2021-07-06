---
title: iOS网络编程之五——请求回执类NSURLResponse属性简介
date: 2016-02-25
categories: iOS逻辑初窥
tags: []
---
## iOS网络编程之五——请求回执类NSURLResponse属性简介

        NSURLResponse类中存放请求的回执信息，在发送网络请求时，如果请求成功，首先会接收到服务端的回执信息，直接开始接收具体的返回数据。NSURLResponse对象中主要有以下属性：

```
//请求的URL地址
@property (nullable, readonly, copy) NSURL *URL;
//返回数据的数据类型
@property (nullable, readonly, copy) NSString *MIMEType;
//获取返回数据的内容长度
@property (readonly) long long expectedContentLength;
//获取返回数据的编码方式
@property (nullable, readonly, copy) NSString *textEncodingName;
//返回拼接的数据文件名 以url为名 数据没醒MIMEType为扩展名
@property (nullable, readonly, copy) NSString *suggestedFilename;
```

对于HTTP请求，请求回执会被封装为NSHTTPURLResponse对象，其中除了有上面那些属性外，还有如下的扩展属性：

```
//请求的状态码
@property (readonly) NSInteger statusCode;
//请求头中所有的字段
@property (readonly, copy) NSDictionary *allHeaderFields;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

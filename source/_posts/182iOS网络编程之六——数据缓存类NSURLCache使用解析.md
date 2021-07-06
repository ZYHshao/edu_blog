---
title: iOS网络编程之六——数据缓存类NSURLCache使用解析
date: 2016-02-26
categories: iOS逻辑初窥
tags: []
---
## iOS网络编程之六——数据缓存类NSURLCache使用解析

### 一、引言

        在前面博客中，介绍了NSURLRequest请求类的相关使用方法，其中有介绍关于请求返回数据的缓存策略，实际上，iOS中具体缓存操作的管理是由NSURLCache类来实现的。NSURLRequest类介绍的博客地址如下：

iOS中NSURLRequest相关使用：[http://my.oschina.net/u/2340880/blog/620225](http://my.oschina.net/u/2340880/blog/620225)。

### 二、NSURLCache中方法与属性

```
//获取当前应用的缓存管理对象
+ (NSURLCache *)sharedURLCache;
//设置自定义的NSURLCache作为应用缓存管理对象
+ (void)setSharedURLCache:(NSURLCache *)cache;
//初始化一个应用缓存对象
/*
memoryCapacity 设置内存缓存容量
diskCapacity 设置磁盘缓存容量
path 磁盘缓存路径
内容缓存会在应用程序退出后 清空 磁盘缓存不会
*/
- (instancetype)initWithMemoryCapacity:(NSUInteger)memoryCapacity diskCapacity:(NSUInteger)diskCapacity diskPath:(nullable NSString *)path;
//获取某一请求的缓存
- (nullable NSCachedURLResponse *)cachedResponseForRequest:(NSURLRequest *)request;
//给请求设置指定的缓存
- (void)storeCachedResponse:(NSCachedURLResponse *)cachedResponse forRequest:(NSURLRequest *)request;
//移除某个请求的缓存
- (void)removeCachedResponseForRequest:(NSURLRequest *)request;
//移除所有缓存数据
- (void)removeAllCachedResponses;
//移除某个时间起的缓存设置
- (void)removeCachedResponsesSinceDate:(NSDate *)date NS_AVAILABLE(10_10, 8_0);
//内存缓存容量大小
@property NSUInteger memoryCapacity;
//磁盘缓存容量大小
@property NSUInteger diskCapacity;
//当前已用内存容量
@property (readonly) NSUInteger currentMemoryUsage;
//当前已用磁盘容量
@property (readonly) NSUInteger currentDiskUsage;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

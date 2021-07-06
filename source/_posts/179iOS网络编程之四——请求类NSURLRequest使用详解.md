---
title: iOS网络编程之四——请求类NSURLRequest使用详解
date: 2016-02-24
categories: iOS逻辑初窥
tags: []
---
## iOS网络编程之四——请求类NSURLRequest使用详解

### 一、引言

        在前面几篇博客中，介绍了iOS开发中的网络编程相关内容并且介绍了常用了两个平行的网络框架NSURLSession和NSURLConnection。无论是NSURLSession类还是NSURLConnection类，其网络请求都是通过NSURLRequest类进行发起的，本篇博客将介绍NSURLRequest类的用法和其中请求属性的设置。

        前几篇博客地址如下：

1.iOS网络框架介绍：[http://my.oschina.net/u/2340880/blog/618460](http://my.oschina.net/u/2340880/blog/618460)。

2.NSURLSesstion的使用：[http://my.oschina.net/u/2340880/blog/618888](http://my.oschina.net/u/2340880/blog/618888)。

3.NSURLConnection的使用：[http://my.oschina.net/u/2340880/blog/618920](http://my.oschina.net/u/2340880/blog/618920)。

### 二、NSURLRequest类中常用方法和属性总结

```
//通过类方法创建默认的请求对象
/*
通过这种方式创建的请求对象 默认使用NSURLRequestUseProtocolCachePolicy缓存逻辑 默认请求超时时限为60s
*/
+ (instancetype)requestWithURL:(NSURL *)URL;
//返回一个BOOL值 用于判断是否支持安全编码
+ (BOOL)supportsSecureCoding;
//请求对象的初始化方法 创建时设置缓存逻辑和超时时限
+ (instancetype)requestWithURL:(NSURL *)URL cachePolicy:(NSURLRequestCachePolicy)cachePolicy timeoutInterval:(NSTimeInterval)timeoutInterval;
//init方法进行对象的创建 默认使用NSURLRequestUseProtocolCachePolicy缓存逻辑 默认请求超时时限为60s
- (instancetype)initWithURL:(NSURL *)URL;
//init方法进行对象的创建
- (instancetype)initWithURL:(NSURL *)URL cachePolicy:(NSURLRequestCachePolicy)cachePolicy timeoutInterval:(NSTimeInterval)timeoutInterval;
//只读属性 获取请求对象的URL
@property (nullable, readonly, copy) NSURL *URL;
//只读属性 缓存策略枚举
/*
NSURLRequestCachePolicy枚举如下：
typedef NS_ENUM(NSUInteger, NSURLRequestCachePolicy)
{
    //默认的缓存协议
    NSURLRequestUseProtocolCachePolicy = 0,
    //无论有无本地缓存数据 都进行从新请求
    NSURLRequestReloadIgnoringLocalCacheData = 1,
    //忽略本地和远程的缓存数据 未实现的策略
    NSURLRequestReloadIgnoringLocalAndRemoteCacheData = 4, 
    //无论有无缓存数据 都进行从新请求
    NSURLRequestReloadIgnoringCacheData = NSURLRequestReloadIgnoringLocalCacheData,
    //先检查缓存 如果没有缓存再进行请求
    NSURLRequestReturnCacheDataElseLoad = 2,
    //类似离线模式，只读缓存 无论有无缓存都不进行请求
    NSURLRequestReturnCacheDataDontLoad = 3,
    //未实现的策略
    NSURLRequestReloadRevalidatingCacheData = 5, // Unimplemented
};
*/
@property (readonly) NSURLRequestCachePolicy cachePolicy;
//只读属性 获取请求的超时时限
@property (readonly) NSTimeInterval timeoutInterval;
//请求主文档地址 
@property (nullable, readonly, copy) NSURL *mainDocumentURL;
//获取网络请求的服务类型 枚举如下
/*
typedef NS_ENUM(NSUInteger, NSURLRequestNetworkServiceType)
{
    NSURLNetworkServiceTypeDefault = 0,    // Standard internet traffic
    NSURLNetworkServiceTypeVoIP = 1,    // Voice over IP control traffic
    NSURLNetworkServiceTypeVideo = 2,    // Video traffic
    NSURLNetworkServiceTypeBackground = 3, // Background traffic
    NSURLNetworkServiceTypeVoice = 4       // Voice data
};
*/
@property (readonly) NSURLRequestNetworkServiceType networkServiceType;
//获取是否允许使用服务商蜂窝网络
@property (readonly) BOOL allowsCellularAccess;
```

NSURLRequest请求类除了在初始化时可以设定一些属性，创建出来后则大部分属性都为只读的，无法设置与修改。另一个类NSMutableURLRequest可以更加灵活的设置请求的相关属性。

### 三、NSMutableURLRequest类中常用方法与属性总结

```
//设置请求的URL
@property (nullable, copy) NSURL *URL;
//设置请求的缓存策略
@property NSURLRequestCachePolicy cachePolicy;
//设置超时时间
@property NSTimeInterval timeoutInterval;
//请求主文档地址
@property (nullable, copy) NSURL *mainDocumentURL;
//设置网络服务类型
@property NSURLRequestNetworkServiceType networkServiceType NS_AVAILABLE(10_7, 4_0);
//设置是否允许使用服务商蜂窝网
@property BOOL allowsCellularAccess NS_AVAILABLE(10_8, 6_0);
```

### 四、NSURLRequest请求对象与HTTP/HTTPS协议相关请求的属性设置

        一下属性的设置必须使用NSMutableURLRequest类，如果是NSURLRequest，则只可以读，不可以修改。

```
//设置HPPT请求方式 默认为“GET”
@property (copy) NSString *HTTPMethod;
//通过字典设置HTTP请求头的键值数据
@property (nullable, copy) NSDictionary<NSString *, NSString *> *allHTTPHeaderFields;
//设置http请求头中的字段值
- (void)setValue:(nullable NSString *)value forHTTPHeaderField:(NSString *)field;
//向http请求头中添加一个字段
- (void)addValue:(NSString *)value forHTTPHeaderField:(NSString *)field;
//设置http请求体 用于POST请求
@property (nullable, copy) NSData *HTTPBody;
//设置http请求体的输入流
@property (nullable, retain) NSInputStream *HTTPBodyStream;
//设置发送请求时是否发送cookie数据
@property BOOL HTTPShouldHandleCookies;
//设置请求时是否按顺序收发 默认禁用 在某些服务器中设为YES可以提高网络性能
@property BOOL HTTPShouldUsePipelining;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

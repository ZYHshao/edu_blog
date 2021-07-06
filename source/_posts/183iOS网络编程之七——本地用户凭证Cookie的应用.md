---
title: iOS网络编程之七——本地用户凭证Cookie的应用
date: 2016-03-01
categories: iOS逻辑初窥
tags: []
---
## iOS网络编程之七——本地用户凭证Cookie的应用

### 一、何为Cookie

    Cookie是网站为了便是终端身份，保存在终端本地的用户凭证信息。Cookie中的字段与意义由服务端进行定义。例如，当用户在某个网站进行了登录操作后，服务端会将Cookie信息返回给终端，终端会将这些信息进行保存，在下一次再次访问这个网站时，终端会将保存的Cookie信息一并发送到服务端，服务端根据Cookie信息是否有效来判断此用户是否可以自动登录。

### 二、iOS中进行Cookie管理的两个类

    iOS中进行HTTP网络请求Cookie管理主要由两个类负责，一个类是NSHTTPCookieStorage类，一个是NSHTTPCookie类。

#### 1.NSHTTPCookieStorage

    NSHTTPCookieStorage类采用单例的设计模式，其中管理着所有HTTP请求的Cookie信息，常用方法如下：

```
//获取单例对象
+ (NSHTTPCookieStorage *)sharedHTTPCookieStorage;
//所有Cookie数据数组 其中存放NSHTTPCookie对象
@property (nullable , readonly, copy) NSArray<NSHTTPCookie *> *cookies;
//手动设置一条Cookie数据
- (void)setCookie:(NSHTTPCookie *)cookie;
//删除某条Cookie信息
- (void)deleteCookie:(NSHTTPCookie *)cookie;
//删除某个时间后的所有Cookie信息 iOS8后可用
- (nullable NSArray<NSHTTPCookie *> *)cookiesForURL:(NSURL *)URL;
//获取某个特定URL的所有Cookie数据
- (void)removeCookiesSinceDate:(NSDate *)date NS_AVAILABLE(10_10, 8_0);
//为某个特定的URL设置Cookie
- (void)setCookies:(NSArray<NSHTTPCookie *> *)cookies forURL:(nullable NSURL *)URL mainDocumentURL:(nullable NSURL *)mainDocumentURL;
//Cookie数据的接收协议
/*
枚举如下：
typedef NS_ENUM(NSUInteger, NSHTTPCookieAcceptPolicy) {
    NSHTTPCookieAcceptPolicyAlways,//接收所有Cookie信息
    NSHTTPCookieAcceptPolicyNever,//不接收所有Cookie信息
    NSHTTPCookieAcceptPolicyOnlyFromMainDocumentDomain//只接收主文档域的Cookie信息
};
*/
@property NSHTTPCookieAcceptPolicy cookieAcceptPolicy;
```

系统下面的两个通知与Cookie管理有关：

```
//Cookie数据的接收协议改变时发送的通知
FOUNDATION_EXPORT NSString * const NSHTTPCookieManagerAcceptPolicyChangedNotification;
//管理的Cookie数据发生变化时发送的通知
FOUNDATION_EXPORT NSString * const NSHTTPCookieManagerCookiesChangedNotification;
```

#### 2.NSHTTPCookie

    NSHTTPCookie是具体的HTTP请求Cookie数据对象，其中属性方法如下：

```
//下面两个方法用于对象的创建和初始化 都是通过字典进行键值设置
- (nullable instancetype)initWithProperties:(NSDictionary<NSString *, id> *)properties;
+ (nullable NSHTTPCookie *)cookieWithProperties:(NSDictionary<NSString *, id> *)properties;
//返回Cookie数据中可用于添加HTTP头字段的字典
+ (NSDictionary<NSString *, NSString *> *)requestHeaderFieldsWithCookies:(NSArray<NSHTTPCookie *> *)cookies;
//从指定的响应头和URL地址中解析出Cookie数据
+ (NSArray<NSHTTPCookie *> *)cookiesWithResponseHeaderFields:(NSDictionary<NSString *, NSString *> *)headerFields forURL:(NSURL *)URL;
//Cookie数据中的属性字典
@property (nullable, readonly, copy) NSDictionary<NSString *, id> *properties;
//请求响应的版本
@property (readonly) NSUInteger version;
//请求相应的名称
@property (readonly, copy) NSString *name;
//请求相应的值
@property (readonly, copy) NSString *value;
//过期时间
@property (nullable, readonly, copy) NSDate *expiresDate;
//请求的域名
@property (readonly, copy) NSString *domain;
//请求的路径
@property (readonly, copy) NSString *path;
//是否是安全传输
@property (readonly, getter=isSecure) BOOL secure;
//是否只发送HTTP的服务
@property (readonly, getter=isHTTPOnly) BOOL HTTPOnly;
//响应的文档
@property (nullable, readonly, copy) NSString *comment;
//相应的文档URL
@property (nullable, readonly, copy) NSURL *commentURL;
//服务端口列表
@property (nullable, readonly, copy) NSArray<NSNumber *> *portList;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

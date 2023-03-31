---
title: 在iOS中使用NSURLProtocol进行网络代理
date: 2022-11-27
categories: iOS之逻辑初窥
tags: []
---
# 在iOS中使用NSURLProtocol进行网络代理

# 一 引言

网络能力是互联网应用程序必不可少的功能。随着应用程序的复杂，对网络的依赖性也会逐渐增高。如何统一的处理请求头，统一的处理回执数据，统一的进行网络请求过程的监控和修改等都是开发者要考虑的处理的问题。

通常，对于新项目，我们会统一封装网络框架在处理应用中的请求，整个网络的发起和收到回执的过程都可以很好的在底层的框架中进行监控和数据处理。但是这种方式对于网络请求不统一的老项目可能成本较高，要统一的修改网络框架，且对于WebView中的网络请求也需要单独处理，比较繁琐。这种情况下，不妨试一试Foundation框架中自带的NSURLProtocol来进行网络代理，完全无侵入的实现网络全过程的监控和修改处理。

# 二 牛刀小试

在系统的介绍NSURLProtocol之前，我们先来通过一个小例子体验下其使用过程。

首先，NSURLProtocol虽然名字中有Protocol，但是它并不是一个协议，其是继承于NSObject的类。其次，虽然其实一个继承为NSObejct的类，但它更像是一个抽象类，我们不会直接拿这个类进行使用，而是会通过子类的方式来实现它，并且其内的很多方法也都是抽象的，必须由子类来实现。

我们新建一个测试工程，先实现一个简单的GET请求，如下：

```objectivec
NSURL *url = [NSURL URLWithString:
           @"https://www.baidu.com"];
[[[NSURLSession sharedSession] dataTaskWithURL:url completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        NSLog(@"%@", response);
}] resume] ;
```

运行代码，通过控制台的输出，可以看到能够正常的获取到回执数据。

新建一个命名为NetworkProtocl的类，使其继承自NSURLProtocol，实现如下：

```objectivec
#import "NetworkProtcol.h"

@implementation NetworkProtcol

// 网络请求的代理需要注册，在load方法中进行注册
+ (void)load {
    [NSURLProtocol registerClass:self];
}

// 1. 这个方法是第一步，需要判断是否要拦截当前的请
+ (BOOL)canInitWithRequest:(NSURLRequest *)request {
    NSLog(@"canInitWithRequest: %@", request.URL.absoluteString);
    // 根据标记判断，如果处理过了就不再拦截了
    if ([self propertyForKey:@"Handle" inRequest:request]) {
        return false;
    }
    return YES;
}

// 2. 处理请求，这里可以返回一个新的Request对象，可以对原Request进行修改
+ (NSURLRequest *)canonicalRequestForRequest:(NSURLRequest *)request {
    NSLog(@"canonicalRequestForRequest: %@", request.URL.absoluteString);
    return request;
}

// 3. 这个方法处理拦截后的行为，可以做数据修改，本地mock或请求其他接口
- (void)startLoading {
    NSLog(@"startLoading");
    [NSURLProtocol setProperty:@YES forKey:@"Handle" inRequest:self.request];
    [[[NSURLSession sharedSession] dataTaskWithRequest:self.request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        [self.client URLProtocol:self didReceiveResponse:response cacheStoragePolicy:NSURLCacheStorageAllowed];
        [self.client URLProtocolDidFinishLoading:self];
        }] resume];
}

// 整个代理行为结束回调
- (void)stopLoading {
    NSLog(@"stopLoading");
}

@end

```

再次运行，通过控制台的打印，可以看到网络代理已经可以正常的工作，如上面的示例代码所示，整个代理过程最重要的即是三步：

1\. 判断某个请求是否要进行代理拦截。

2\. 处理请求，可以进行修改。

3\. 执行真正的拦截行为，并通过回调来返回结果给原请求方。

# 三 NSURLProtocol详解

NSURLProtocol本身比较简单，其暴露的接口和属性也比较简洁，解释如下：

```objectivec
@interface NSURLProtocol : NSObject

// 初始化方法
- (instancetype)initWithRequest:(NSURLRequest *)request cachedResponse:(nullable NSCachedURLResponse *)cachedResponse client:(nullable id <NSURLProtocolClient>)client;
- (instancetype)initWithTask:(NSURLSessionTask *)task cachedResponse:(nullable NSCachedURLResponse *)cachedResponse client:(nullable id <NSURLProtocolClient>)client;

// SessionTask对象
@property (nullable, readonly, copy) NSURLSessionTask *task

// 客户端对象，用来回调原请求方
@property (nullable, readonly, retain) id <NSURLProtocolClient> client;

// 当前处理的请求对象
@property (readonly, copy) NSURLRequest *request;

// Response缓存
@property (nullable, readonly, copy) NSCachedURLResponse *cachedResponse;

// 这个方法必须子类实现，用来判断是否要拦截某个请求
+ (BOOL)canInitWithRequest:(NSURLRequest *)request;
+ (BOOL)canInitWithTask:(NSURLSessionTask *)task;

// 这个方法必须子类实现，用来对请求进行修改
+ (NSURLRequest *)canonicalRequestForRequest:(NSURLRequest *)request;

// 这个方法用来判断是否要使用缓存请求，判断要进行的请求与缓存的是否相同
+ (BOOL)requestIsCacheEquivalent:(NSURLRequest *)a toRequest:(NSURLRequest *)b;

// 开始拦截行为
- (void)startLoading;

// 整个拦截行为结束
- (void)stopLoading;

// 用来给某个请求打标签，可以防止请求拦截出现递归
+ (void)setProperty:(id)value forKey:(NSString *)key inRequest:(NSMutableURLRequest *)request;
// 获取标签
+ (nullable id)propertyForKey:(NSString *)key inRequest:(NSURLRequest *)request;
// 删除标签
+ (void)removePropertyForKey:(NSString *)key inRequest:(NSMutableURLRequest *)request;

// 注册网络代理类
+ (BOOL)registerClass:(Class)protocolClass;
// 注销注册的代理
+ (void)unregisterClass:(Class)protocolClass;

@end
```

其中，client属性用来与原请求方交互，其协议的方法如下：

```objectivec
@protocol NSURLProtocolClient <NSObject>

// 触发客户端的重定向回调
- (void)URLProtocol:(NSURLProtocol *)protocol wasRedirectedToRequest:(NSURLRequest *)request redirectResponse:(NSURLResponse *)redirectResponse;
// 触发客户端的缓存有效性回调
- (void)URLProtocol:(NSURLProtocol *)protocol cachedResponseIsValid:(NSCachedURLResponse *)cachedResponse;
// 触发客户端的接收回执回调
- (void)URLProtocol:(NSURLProtocol *)protocol didReceiveResponse:(NSURLResponse *)response cacheStoragePolicy:(NSURLCacheStoragePolicy)policy;
// 触发客户端的接收数据回调
- (void)URLProtocol:(NSURLProtocol *)protocol didLoadData:(NSData *)data;
// 触发客户端的请求完成回调
- (void)URLProtocolDidFinishLoading:(NSURLProtocol *)protocol;
// 触发客户端的请求失败回调
- (void)URLProtocol:(NSURLProtocol *)protocol didFailWithError:(NSError *)error;
// 触发客户端的用户认证回调
- (void)URLProtocol:(NSURLProtocol *)protocol didReceiveAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge;
// 触发客户端的取消用户认证回调
- (void)URLProtocol:(NSURLProtocol *)protocol didCancelAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge;

@end
```

# 四 对网页视图的网络请求进行拦截

如果是使用UIWebView加载网页，则NSURLProtocol默认支持拦截，示例代码如下：

```objectivec
#import "NetworkProtcol.h"

@interface NetworkProtcol ()<NSURLSessionDelegate>

@property (atomic,strong,readwrite) NSURLSessionDataTask *task;
@property (nonatomic,strong) NSURLSession *session;

@end

@implementation NetworkProtcol

+ (void)load {
    [NSURLProtocol registerClass:self];
}

+ (BOOL)canInitWithRequest:(NSURLRequest *)request
{
    //只处理http和https请求
    NSString *scheme = [[request URL] scheme];
    if ( ([scheme caseInsensitiveCompare:@"http"] == NSOrderedSame ||
          [scheme caseInsensitiveCompare:@"https"] == NSOrderedSame)) {
        //看看是否已经处理过了，防止无限循环
        if ([NSURLProtocol propertyForKey:@"Handle" inRequest:request]) {
            return NO;
        }
        return YES;
    }
    return NO;
}

+ (NSURLRequest *) canonicalRequestForRequest:(NSURLRequest *)request {
    return request;
}
- (void)startLoading
{
    NSMutableURLRequest *mutableReqeust = [[self request] mutableCopy];
    //打标签，防止无限循环
    [NSURLProtocol setProperty:@YES forKey:@"Handle" inRequest:mutableReqeust];
    
    NSURLSessionConfiguration *configure = [NSURLSessionConfiguration defaultSessionConfiguration];
    
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    
    self.session  = [NSURLSession sessionWithConfiguration:configure delegate:self delegateQueue:queue];
    self.task = [self.session dataTaskWithRequest:mutableReqeust];
    [self.task resume];
}

- (void)stopLoading
{
    [self.session invalidateAndCancel];
    self.session = nil;
}

- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
    if (error != nil) {
        [self.client URLProtocol:self didFailWithError:error];
    }else
    {
        [self.client URLProtocolDidFinishLoading:self];
    }
}

- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask
didReceiveResponse:(NSURLResponse *)response
 completionHandler:(void (^)(NSURLSessionResponseDisposition disposition))completionHandler
{
    [self.client URLProtocol:self didReceiveResponse:response cacheStoragePolicy:NSURLCacheStorageNotAllowed];
    
    completionHandler(NSURLSessionResponseAllow);
}

- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data
{
    [self.client URLProtocol:self didLoadData:data];
}

@end

```

如果使用的是WKWebView，则无法直接被拦截，需要做如下的额外处理：

```objectivec
Class cls = [[[WKWebView new] valueForKey:@"browsingContextController"] class];
SEL sel = NSSelectorFromString(@"registerSchemeForCustomProtocol:");
if ([cls respondsToSelector:sel]) {
    // 通过 http 和 https 的请求，同理可通过其他的 Scheme 但是要满足 URL Loading System
    [cls performSelector:sel withObject:@"http"];
    [cls performSelector:sel withObject:@"https"];
}

WKWebView *web = [[WKWebView alloc] initWithFrame:self.view.frame];
[self.view addSubview:web];
NSURL *url = [NSURL URLWithString:
           @"https://www.baidu.com"];
[web loadRequest:[NSURLRequest requestWithURL:url]];
```

需要注意，这里会涉及到一些私有属性和方法，可能会存在提审风险。

# 五 一些思考

NSURLProtocol的使用非常简单，但是可以做的事情却并不少。例如我们可以用其来做统一的网络处理，来做无侵入的网络监控等。

-   作为底层工具，统一的为Request添加通用Header字段。
-   统一的进行自定义用户认证。
-   做为环境切换工具，根据环境做URL的映射。
-   请求参数的整理和修改，统一增加通用参数。
-   统一修改或增加Response Header数据。
-   根据需求，做为本地的Mock工具，访问到本地服务或Mock远程数据。
-   监控端到端的网络请求性能，进行时间统计。
-   等等...

> 专注技术，懂的热爱，愿意分享，做个朋友
> 
> QQ：316045346
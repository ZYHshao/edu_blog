---
title: 深入理解HTTPS及在iOS系统中适配HTTPS类型网络请求(下)
date: 2016-12-18
categories: iOS逻辑初窥
tags: []
---
## 深入理解HTTPS及在iOS系统中适配HTTPS类型网络请求(下)

### 一、引言

     上一篇博客详细讨论了HTTPS协议的原理，搭建HTTPS测试环境以及证书的相关基础。本篇博客将继续探讨更多在iOS开发中适配HTTPS类型请求的内容。上篇博客的地址如下：

[https://my.oschina.net/u/2340880/blog/807358](https://my.oschina.net/u/2340880/blog/807358)。

### 二、关于NSURLAuthenticationChallenge相关类

    我们在实现URLSession的认证协议方法时，会接收到一个NSURLAuthenticationChallenge类型的参数。简单理解，这个参数就是服务端发起的一个验证挑战，客户端需要根据挑战的类型提供相应的挑战凭证。当然，挑战凭证不一定都是进行HTTPS证书的信任，也可能是需要客户端提供用户密码或者提供双向验证时的客户端证书。当这个挑战凭证被验证通过时，请求便可以继续顺利进行。NSURLAuthenticationChallenge类对象中有一个sender代理实例，开发者通过这个实例来可控采用怎样的验证方式。解析如下：

```objectivec
//使用凭证进行验证
- (void)useCredential:(NSURLCredential *)credential forAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge;
//试图不提供凭证继续请求
- (void)continueWithoutCredentialForAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge;
//取消凭证验证
- (void)cancelAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge;
//使用默认提供的凭证行为
- (void)performDefaultHandlingForAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge;
//拒绝当前提供的受保护控件并且尝试不提供凭证继续请求
- (void)rejectProtectionSpaceAndContinueWithChallenge:(NSURLAuthenticationChallenge *)challenge;
```

可以看到，上面的协议方法中如果要进行凭证的验证，需要客户端提供一个凭证对象NSURLCredential。这个类可以简单理解为客户端创建的凭证信息，解析如下：

```objectivec
//通过用户名和密码进行凭证的创建
- (instancetype)initWithUser:(NSString *)user password:(NSString *)password persistence:(NSURLCredentialPersistence)persistence;
//同上
+ (NSURLCredential *)credentialWithUser:(NSString *)user password:(NSString *)password persistence:(NSURLCredentialPersistence)persistence;
//用户名属性 只读
@property (nullable, readonly, copy) NSString *user;
//密码属性 只读
@property (nullable, readonly, copy) NSString *password;
//是否有密码 只读
@property (readonly) BOOL hasPassword;
//通过客户端提供证书进行双向验证
- (instancetype)initWithIdentity:(SecIdentityRef)identity certificates:(nullable NSArray *)certArray persistence:(NSURLCredentialPersistence)persistence NS_AVAILABLE(10_6, 3_0);
//同上
+ (NSURLCredential *)credentialWithIdentity:(SecIdentityRef)identity certificates:(nullable NSArray *)certArray persistence:(NSURLCredentialPersistence)persistence NS_AVAILABLE(10_6, 3_0);
//创建证书信任凭证 用户自签名的HTTPS请求
- (instancetype)initWithTrust:(SecTrustRef)trust NS_AVAILABLE(10_6, 3_0);
//同上
+ (NSURLCredential *)credentialForTrust:(SecTrustRef)trust NS_AVAILABLE(10_6, 3_0);

```

上面方法中的NSURLCredentialPersistence枚举用来设置凭证的存储方式，解析如下：

```objectivec
typedef NS_ENUM(NSUInteger, NSURLCredentialPersistence) {
    NSURLCredentialPersistenceNone,  //不保存
    NSURLCredentialPersistenceForSession, //在本URLSession中有效
    NSURLCredentialPersistencePermanent, //保存在钥匙串中 ，永久有效
    NSURLCredentialPersistenceSynchronizable NS_ENUM_AVAILABLE(10_8, 6_0) //永久有效 并且被所有APPID设备共享
};
```

### 三、使用AFNetworking进行自签名证书HTTPS请求的认证

    使用AFNetworking也可以很方便的进行自签名证书的认证，还以上一节博客搭建的HTTPS环境为例，示例代码如下：

```objectivec
-(void)afHttps{
    NSURLRequest * req = [NSURLRequest requestWithURL:[NSURL URLWithString:@"https://localhost:8080/users"]];
    AFSecurityPolicy *securityPolicy = [AFSecurityPolicy policyWithPinningMode:AFSSLPinningModeCertificate];
    //    securityPolicy.allowInvalidCertificates = YES;//是否允许使用自签名证书
    securityPolicy.validatesDomainName = NO;//是否需要验证域名，默认YES
    
    AFHTTPSessionManager *_manager = [AFHTTPSessionManager manager];
    _manager.responseSerializer = [AFHTTPResponseSerializer serializer];
    _manager.securityPolicy = securityPolicy;
    //设置超时
    [_manager.requestSerializer willChangeValueForKey:@"timeoutinterval"];
    _manager.requestSerializer.timeoutInterval = 20.f;
    [_manager.requestSerializer didChangeValueForKey:@"timeoutinterval"];
    _manager.requestSerializer.cachePolicy = NSURLRequestReloadIgnoringCacheData;
    _manager.responseSerializer.acceptableContentTypes  = [NSSet setWithObjects:@"application/xml",@"text/html",@"text/plain",@"application/json",nil];
    [_manager setSessionDidReceiveAuthenticationChallengeBlock:^NSURLSessionAuthChallengeDisposition(NSURLSession *session, NSURLAuthenticationChallenge *challenge, NSURLCredential *__autoreleasing *_credential) {
        
        SecTrustRef serverTrust = [[challenge protectionSpace] serverTrust];
        /**
         *  导入多张CA证书
         */
        NSString *cerPath = [[NSBundle mainBundle] pathForResource:@"cert" ofType:@"der"];//自签名证书
        NSData* caCert = [NSData dataWithContentsOfFile:cerPath];
        NSArray *cerArray = @[caCert];
        _manager.securityPolicy.pinnedCertificates = cerArray;
        SecCertificateRef caRef = SecCertificateCreateWithData(NULL, (__bridge CFDataRef)caCert);
        NSArray *caArray = @[(__bridge id)(caRef)];
        
        SecTrustSetAnchorCertificates(serverTrust, (__bridge CFArrayRef)caArray);
        SecTrustSetAnchorCertificatesOnly(serverTrust,NO);
        NSURLSessionAuthChallengeDisposition disposition = NSURLSessionAuthChallengePerformDefaultHandling;
        __autoreleasing NSURLCredential *credential = nil;
        if ([challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust]) {
            if ([_manager.securityPolicy evaluateServerTrust:challenge.protectionSpace.serverTrust forDomain:challenge.protectionSpace.host]) {
                                credential = [NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust];
                                if (credential) {
                                    disposition = NSURLSessionAuthChallengeUseCredential;
                                } else {
                                    disposition = NSURLSessionAuthChallengePerformDefaultHandling;
                                }
            } else {
                                disposition = NSURLSessionAuthChallengeCancelAuthenticationChallenge;
            }
        } else {
                        disposition = NSURLSessionAuthChallengePerformDefaultHandling;
        }
        
        return disposition;
    }];
    [[_manager dataTaskWithRequest:req completionHandler:^(NSURLResponse * _Nonnull response, id  _Nullable responseObject, NSError * _Nullable error) {
        NSLog(@"%@,%@",[[NSString alloc]initWithData:responseObject encoding:NSUTF8StringEncoding],error);
    }]resume];
}

```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: iOS开发之Accounts框架详解
date: 2018-08-06
categories: iOS逻辑初窥
tags: []
---
## iOS开发之Accounts框架详解

    Accounts框架是iOS原生提供的一套账户管理框架，其支持Facebook，新浪微博，腾讯微博，Twitter和领英账户管理的功能。需要注意，在iOS 11及以上系统中，将此功能已经删除，因此Accounts.framework实际上已经没有太大的意义，其只在iOS 11之前的系统上可用。

### 一、Accounts框架概览

![](https://oscimg.oschina.net/oscnet/0dfea13c8f268130bdaee34d6aa7ce16e9f.jpg)

从上图可以看出，Accounts框架中最重要的3个类是ACAccountCredential类、ACAccount类和ACAccountStore类。后面我们着重介绍这3个类。

      首先先来看ACAccountType类，这个类用来定义账户类型，如下：

```objectivec
@interface ACAccountType : NSObject
//类型描述
@property (readonly, nonatomic) NSString *accountTypeDescription;
//标识符
@property (readonly, nonatomic) NSString *identifier;
//此类账户是否已经授权
@property (readonly, nonatomic) BOOL     accessGranted;
@end
```

需要注意，这个类中的属性都是只读的，这也就是说，开发者不能够直接使用这个类进行对象的构建，需要借助ACAccountStore类来创建ACAccountType实例，后面会介绍。

      ACErrorCode定义了错误码的意义，如下：

```objectivec
typedef enum ACErrorCode {
    ACErrorUnknown = 1,//未知错误
    ACErrorAccountMissingRequiredProperty,  // 缺少必选属性错误
    ACErrorAccountAuthenticationFailed,     // 授权失败
    ACErrorAccountTypeInvalid,              // 授权无效
    ACErrorAccountAlreadyExists,            // 账户已经存在
    ACErrorAccountNotFound,                 // 账户未找到
    ACErrorPermissionDenied,                // 没有权限
    ACErrorAccessInfoInvalid,               // 信息失效
    ACErrorClientPermissionDenied,          // 客户端没有权限
    ACErrorAccessDeniedByProtectionPolicy,  // 无法取得证书
    ACErrorCredentialNotFound,              // 证书未找到
    ACErrorFetchCredentialFailed,           // 请求证书失败
    ACErrorStoreCredentialFailed,           // 存储证书失败
    ACErrorRemoveCredentialFailed,          // 删除证书失败
    ACErrorUpdatingNonexistentAccount,      // 更新失败
    ACErrorInvalidClientBundleID,           // 无效的BundleID
    ACErrorDeniedByPlugin,                  // 权限被阻止
    ACErrorCoreDataSaveFailed,              // 数据库存储失败
    ACErrorFailedSerializingAccountInfo,    //序列化数据失败
    ACErrorInvalidCommand,                  //无效的命令
    ACErrorMissingTransportMessageID,       //缺少安全信息
    ACErrorCredentialItemNotFound,          // 证书缺少字段
    ACErrorCredentialItemNotExpired,        // 证书字段失效
} ACErrorCode;
```

### 二、进行账户操作

    在iOS 11以下的系统中，在设置中可以找到账户管理选项，如下图：

![](https://oscimg.oschina.net/oscnet/95234bea81c13c4205dfef3af400f38a630.jpg)

点击相应的社交平台，通过账号密码可以进行登录。这里一旦设置登录，那么在第三方应用程序中便可以通过Accounts框架来获取授权信息。

    首先，要使用Accounts框架，需要导入相应头文件，如下：

```objectivec
#import <Accounts/Accounts.h>
```

但应用程序首次使用用户社交平台的账户时，需要获取用户的授权，示例代码如下：

```objectivec
//创建操作对象
ACAccountStore *accountStore = [[ACAccountStore alloc] init];
//通过操作对象 构建社交平台类型示例  这里采用的新浪微博
ACAccountType *accountType = [accountStore accountTypeWithAccountTypeIdentifier:ACAccountTypeIdentifierSinaWeibo];
//进行用户授权请求
[accountStore requestAccessToAccountsWithType: accountType options:nil completion:^(BOOL granted, NSError *error) {
   if (error) {
        NSLog(@"error = %@", [error localizedDescription]);
   }      
   dispatch_async(dispatch_get_main_queue(), ^{
      if(granted){
         NSLog(@"授权通过了");
    });
}];

```

获取用户授权界面如下图所示：

![](https://oscimg.oschina.net/oscnet/79f558158776e92d11b3976b0343fdf31d6.jpg)

一旦用户授权通过，可以使用如下方法获取到用户的社交平台账户信息：

```objectivec
ACAccountStore *accountStore = [[ACAccountStore alloc] init];
ACAccountType *accountType = [accountStore accountTypeWithAccountTypeIdentifier:ACAccountTypeIdentifierSinaWeibo];
NSArray *twitterAccounts = [accountStore accountsWithAccountType:accountType];
```

Accounts框架支持的社交平台类型ID如下：

```objectivec
NSString * const ACAccountTypeIdentifierTwitter;//Twitter
NSString * const ACAccountTypeIdentifierFacebook;//Facebook
NSString * const ACAccountTypeIdentifierSinaWeibo;//新浪微博
NSString * const ACAccountTypeIdentifierTencentWeibo;//腾讯微博
NSString * const ACAccountTypeIdentifierLinkedIn;//领英
```

在调用requestAccessToAccountsWithType方法时，可以传入一个字典参数，某些社交平台的授权需要配置一些额外参数，例如：

```objectivec
//facebook相关
NSString * const ACFacebookAppIdKey;//设置appid
NSString * const ACFacebookPermissionsKey;//设置权限key
NSString * const ACFacebookAudienceKey; //权限key
NSString * const ACFacebookAudienceEveryone;//公开权限
NSString * const ACFacebookAudienceFriends;//好友权限
NSString * const ACFacebookAudienceOnlyMe;//私人权限
//领英相关
NSString * const ACLinkedInAppIdKey;//领英appid
NSString * const ACLinkedInPermissionsKey; //设置权限key
//腾讯微博
NSString *const ACTencentWeiboAppIdKey; //设置appid
```

### 三、用户信息相关类解析

ACAccount类解析如下：

```objectivec
@interface ACAccount : NSObject
- (instancetype)initWithAccountType:(ACAccountType *)type;//构造方法
@property (readonly, weak, NS_NONATOMIC_IOSONLY) NSString      *identifier;//标识符
@property (strong, NS_NONATOMIC_IOSONLY)   ACAccountType       *accountType;//账户类型
@property (copy, NS_NONATOMIC_IOSONLY)     NSString            *accountDescription;//账户描述
@property (copy, NS_NONATOMIC_IOSONLY)     NSString            *username;//用户名
@property (readonly, NS_NONATOMIC_IOSONLY)  NSString           *userFullName;//完整名称
@property (strong, NS_NONATOMIC_IOSONLY)   ACAccountCredential *credential;//授权凭证
@end
```

ACAccountCredential类解析如下：

```objectivec
@interface ACAccountCredential : NSObject
//构造方法
- (instancetype)initWithOAuthToken:(NSString *)token tokenSecret:(NSString *)secret;
- (instancetype)initWithOAuth2Token:(NSString *)token 
                       refreshToken:(NSString *)refreshToken
                         expiryDate:(NSDate *)expiryDate;
//token
@property (copy, nonatomic) NSString *oauthToken;
@end
```

### 四、ACAccountStore类解析

```objectivec
@interface ACAccountStore : NSObject
//所有账户列表
@property (readonly, weak, NS_NONATOMIC_IOSONLY) NSArray *accounts;
//根据id获取账户
- (ACAccount *)accountWithIdentifier:(NSString *)identifier;
//根据id获取账户类型对象
- (ACAccountType *)accountTypeWithAccountTypeIdentifier:(NSString *)typeIdentifier;
//获取指定类型的所有账户
- (NSArray *)accountsWithAccountType:(ACAccountType *)accountType;
//进行账户存储
- (void)saveAccount:(ACAccount *)account withCompletionHandler:(ACAccountStoreSaveCompletionHandler)completionHandler;
//进行账户使用权限申请
- (void)requestAccessToAccountsWithType:(ACAccountType *)accountType
                  withCompletionHandler:(ACAccountStoreRequestAccessCompletionHandler)handler NS_DEPRECATED(NA, NA, 5_0, 6_0);
- (void)requestAccessToAccountsWithType:(ACAccountType *)accountType
                                options:(NSDictionary *)options
                             completion:(ACAccountStoreRequestAccessCompletionHandler)completion;
//如果账户权限已经过期 调用此方法进行刷新
- (void)renewCredentialsForAccount:(ACAccount *)account completion:(ACAccountStoreCredentialRenewalHandler)completionHandler;
//删除账户
- (void)removeAccount:(ACAccount *)account withCompletionHandler:(ACAccountStoreRemoveCompletionHandler)completionHandler;
@end
```

> 千里之遥始于足下，冰冻三尺非一日之寒
> 
> ——共勉

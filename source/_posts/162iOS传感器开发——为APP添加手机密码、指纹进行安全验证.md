---
title: iOS传感器开发——为APP添加手机密码、指纹进行安全验证
date: 2015-12-10
categories: iOS逻辑初窥
tags: []
---
## iOS传感器开发——为APP添加手机密码、指纹进行安全验证

### 一、引言

        iPhone5s之后，iPhone硬件上已支持进行指纹识别的功能，相应的，一些新的api也可以应用于APP中，进行用户安全的验证。目前，开发者可以使用的安全验证方式有两种，一种是通过手机密码进行验证，一种是通过识别指纹进行验证。

### 二、为APP添加安全验证

要使用安全验证的相关api，我们需要引入如下头文件：

```
#import <LocalAuthentication/LocalAuthentication.h>
```

添加手机密码验证：

```
    //创建安全验证对象
    LAContext * con = [[LAContext alloc]init];
    NSError * error;
    //判断是否支持密码验证
    /**
    *LAPolicyDeviceOwnerAuthentication 手机密码的验证方式
    *LAPolicyDeviceOwnerAuthenticationWithBiometrics 指纹的验证方式
    */
    BOOL can = [con canEvaluatePolicy:LAPolicyDeviceOwnerAuthentication error:&error];
    if (can) {
        [con evaluatePolicy:LAPolicyDeviceOwnerAuthentication localizedReason:@"验证信息" reply:^(BOOL success, NSError * _Nullable error) {
            NSLog(@"%d,%@",success,error);
        }];
        
    }
```

canEvaluatePolicy是用来判断是否支持手机密码验证的，如果没有设置手机密码，会返回NO，如果启用了，会出现如下界面：

![](http://static.oschina.net/uploads/space/2015/1210/162201_ZXmj_2340880.png)

密码验证的提示信息，我们可以自定义设置。

进行指纹验证：

```
LAContext * con = [[LAContext alloc]init];
    NSError * error;
    BOOL can = [con canEvaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics error:&error];
    NSLog(@"%d",can);
    if (can) {
        [con evaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics localizedReason:@"验证指纹" reply:^(BOOL success, NSError * _Nullable error) {
            NSLog(@"%d,%@",success,error);
        }];
        
    }
```

回调中的success用来判断是否验证成功：

![](http://static.oschina.net/uploads/space/2015/1210/163655_ufUv_2340880.jpg)

通过这些验证方式，可以使用户的数据更加安全，在做敏感操作时，可以确保是手机的持有者。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

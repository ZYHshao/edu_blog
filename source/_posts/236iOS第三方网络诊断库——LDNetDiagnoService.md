---
title: iOS第三方网络诊断库——LDNetDiagnoService
date: 2016-07-08
categories: iOS第三方库
tags: []
---
## iOS第三方网络诊断库——LDNetDiagnoService_IOS

        LDNetDiagnoService\_IOS是一个开源的第三方网络诊断框架，它通过ping与traceroute原理来对指定域名进行网络诊断。并且这个库一直在跟进维护，进行IPV\_6-Only的支持。    

        LDNetDiagnoService_IOS的git地址如下：[https://github.com/Lede-Inc/LDNetDiagnoService_IOS](https://github.com/Lede-Inc/LDNetDiagnoService_IOS)。

        LDNetDiagnoService的使用十分简单，只需要3步即可完成。

        首先需要对服务引擎进行初始化，代码如下：

```objectivec
    //进行服务引擎的初始化 其中AppCode，AppName，UserID与dormain参数必须填写，其他参数会自动生成
    service = [[LDNetDiagnoService alloc]initWithAppCode:@"app编码" 
                                                 appName:@"demo" 
                                              appVersion:nil 
                                                  userID:@"UserID" 
                                                deviceID:nil 
                                                 dormain:@"www.baidu.com" 
                                             carrierName:nil 
                                          ISOCountryCode:nil 
                                       MobileCountryCode:nil 
                                           MobileNetCode:nil];
    //设置代理
    service.delegate = self;


```

初始化完成服务引擎后，需要开启检测，如下：

```objectivec
//开始诊断网络
- (void)startNetDiagnosis;
//停止诊断网络
- (void)stopNetDialogsis;
```

开始诊断网络后，会通过代理方法将诊断信息回调给开发者，代码如下：

```objectivec
/**
 * 告诉调用者诊断开始
 */
- (void)netDiagnosisDidStarted{
    NSLog(@"开始进行诊断~~");
}


/**
 * 逐步返回监控信息，
 * 如果需要实时显示诊断数据，实现此接口方法
 */
- (void)netDiagnosisStepInfo:(NSString *)stepInfo{
    NSLog(@"正在诊断：%@",stepInfo);
}


/**
 * 因为监控过程是一个异步过程，当监控结束后告诉调用者；
 * 在监控结束的时候，对监控字符串进行处理
 */
- (void)netDiagnosisDidEnd:(NSString *)allLogInfo{
    NSLog(@"诊断结束");
    NSLog(@"%@",allLogInfo);
}

```

Xcode调试区信息如下：

![](http://static.oschina.net/uploads/space/2016/0708/102620_RuwC_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

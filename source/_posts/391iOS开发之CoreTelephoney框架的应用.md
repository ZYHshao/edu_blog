---
title: iOS开发之CoreTelephoney框架的应用
date: 2019-02-26
categories: iOS逻辑初窥
tags: []
---
## iOS开发之CoreTelephoney框架的应用

      CoreTelephoney框架用来获取手机网络状态以及运营商相关信息。

### 一、CTTelephonyNetworkInfo类

      这个类是CoreTelephoney框架的核心，使用它来获取手机的运营商、网络等状态信息。使用示例如下：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    CTTelephonyNetworkInfo *info = [[CTTelephonyNetworkInfo alloc] init];
    //获取运营商信息
    CTCarrier *carrier = info.subscriberCellularProvider;
    NSLog(@"carrier:%@", [carrier description]);
}
```

运营商信息示例如下：

```
Carrier name: [中国移动]
Mobile Country Code: [460]
Mobile Network Code:[02]
ISO Country Code:[cn]
Allows VOIP? [YES]
```

CTTelephonyNetworkInfo类解析如下：

```objectivec
//获取所有运营商信息  iOS 12 后支持
@property(readonly, retain, nullable) NSDictionary<NSString *, CTCarrier *> *serviceSubscriberCellularProviders;
//当前获取运营商信息
@property(readonly, retain, nullable) CTCarrier *subscriberCellularProvider;
//无线网络提供信息
@property (nonatomic, readonly, retain, nullable) NSDictionary<NSString *, NSString *> * serviceCurrentRadioAccessTechnology;
//当前无线网络信息
/*
CTRadioAccessTechnologyGPRS      //2.5g
CTRadioAccessTechnologyEdge      //2.7G
CTRadioAccessTechnologyWCDMA     //3G
CTRadioAccessTechnologyHSDPA     //3.5G
CTRadioAccessTechnologyHSUPA     //3G与4G之间的过度技术
CTRadioAccessTechnologyCDMA1x    //3G
CTRadioAccessTechnologyCDMAEVDORev0  
CTRadioAccessTechnologyCDMAEVDORevA  
CTRadioAccessTechnologyCDMAEVDORevB 
CTRadioAccessTechnologyeHRPD   
CTRadioAccessTechnologyLTE       //4G
*/  
@property (nonatomic, readonly, retain, nullable) NSString* currentRadioAccessTechnology;

```

CTCattier类中定义了运营商相关的信息，解析如下：

```objectivec
//运营商名字
@property (nonatomic, readonly, retain, nullable) NSString *carrierName;
//国家编码
@property (nonatomic, readonly, retain, nullable) NSString *mobileCountryCode;
//网络编码
@property (nonatomic, readonly, retain, nullable) NSString *mobileNetworkCode;
//ISO编码
@property (nonatomic, readonly, retain, nullable) NSString* isoCountryCode;
//是否允许VOIP
@property (nonatomic, readonly, assign) BOOL allowsVOIP;
```

CTCellularData类用来监听用户的网络状态，可以设置当网络状态发生变化后回调的方法，例如：

```objectivec
cellularData = [[CTCellularData alloc] init];
    // 状态发生变化时调用
cellularData.cellularDataRestrictionDidUpdateNotifier = ^(CTCellularDataRestrictedState restrictedState) {
    switch (restrictedState) {
        case kCTCellularDataRestrictedStateUnknown:
            NSLog(@"蜂窝移动网络状态：未知");
            break;
        case kCTCellularDataRestricted:
            NSLog(@"蜂窝移动网络状态：关闭");
            break;
        case kCTCellularDataNotRestricted:
            NSLog(@"蜂窝移动网络状态：开启");
            break;     
        default:
            break;
    }
};
```

需要注意，在iOS中使用网络需要获取用户权限，如果用户没有给网络权限，获取到的状态也将是未开启。

### 二、CTCallCenter

      使用CTCallCenter相关类可以获取当前通话电话的相关信息，CTCallCenter通过管理中心，其中提供了一个方法来获取当前进行中的通话：

```objectivec
//获取当前所有激活中的通话
@property(readonly, retain, nullable) NSSet<CTCall*> *currentCalls;
```

通话被抽象成CTCall对象，解析如下：

```objectivec
//当前通话状态
/*
CTCallStateDialing  拨号
CTCallStateIncoming 来电
CTCallStateConnected 接通
CTCallStateDisconnected 挂断
*/
@property(nonatomic, readonly, copy) NSString *callState;
//通话ID
@property(nonatomic, readonly, copy) NSString *callID;
```

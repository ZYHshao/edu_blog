---
title: iOS获取和监测设备基本信息——UIDevice的使用
date: 2015-12-18
categories: iOS逻辑初窥
tags: []
---
## iOS获取和监测设备基本信息——UIDevice的使用

```
//获取当前设备单例
+ (UIDevice *)currentDevice;
//获取当前设备名称 
@property(nonatomic,readonly,strong) NSString    *name;              // e.g. "My iPhone"
//获取当前设备模式
@property(nonatomic,readonly,strong) NSString    *model;             // e.g. @"iPhone", @"iPod touch"
//获取本地化的当前设备模式
@property(nonatomic,readonly,strong) NSString    *localizedModel;    // localized version of model
//获取系统名称
@property(nonatomic,readonly,strong) NSString    *systemName;        // e.g. @"iOS"
//获取系统版本
@property(nonatomic,readonly,strong) NSString    *systemVersion;     // e.g. @"4.0"
//获取设备方向
@property(nonatomic,readonly) UIDeviceOrientation orientation;       
//获取设备UUID对象
@property(nullable, nonatomic,readonly,strong) NSUUID      *identifierForVendor;
//是否开启监测电池状态 开启后 才可以正常获取电池状态
@property(nonatomic,getter=isBatteryMonitoringEnabled) BOOL batteryMonitoringEnabled NS_AVAILABLE_IOS(3_0);  // default is NO
//获取电池状态
@property(nonatomic,readonly) UIDeviceBatteryState          batteryState NS_AVAILABLE_IOS(3_0);  
//获取电量
@property(nonatomic,readonly) float                         batteryLevel NS_AVAILABLE_IOS(3_0);
```

设备方向的枚举如下：

```
typedef NS_ENUM(NSInteger, UIDeviceOrientation) {
    UIDeviceOrientationUnknown,
    UIDeviceOrientationPortrait,            // home键在下
    UIDeviceOrientationPortraitUpsideDown,  // home键在上
    UIDeviceOrientationLandscapeLeft,       // home键在右
    UIDeviceOrientationLandscapeRight,      // home键在左
    UIDeviceOrientationFaceUp,              // 屏幕朝上
    UIDeviceOrientationFaceDown             // 屏幕朝下
};
```

电池状态的枚举如下：

```
typedef NS_ENUM(NSInteger, UIDeviceBatteryState) {
    UIDeviceBatteryStateUnknown,
    UIDeviceBatteryStateUnplugged,   // 放电状态
    UIDeviceBatteryStateCharging,    // 充电未充满状态
    UIDeviceBatteryStateFull,        // 充电已充满
};
```

下面的方法关于监测屏幕状态：

```
//获取是否开启屏幕状态更改通知
@property(nonatomic,readonly,getter=isGeneratingDeviceOrientationNotifications) BOOL generatesDeviceOrientationNotifications;
//开始监测通知
- (void)beginGeneratingDeviceOrientationNotifications;     
//结束监测通知
- (void)endGeneratingDeviceOrientationNotifications;
```

下面这两个放大与距离传感器应用相关，可参考：[http://my.oschina.net/u/2340880/blog/544341](http://my.oschina.net/u/2340880/blog/544341).

```
@property(nonatomic,getter=isProximityMonitoringEnabled) BOOL proximityMonitoringEnabled NS_AVAILABLE_IOS(3_0); //开启距离传感器
//是否触发了距离传感器
@property(nonatomic,readonly)                            BOOL proximityState
```

相关通知：

```
//设备方向改变时发送的通知
UIKIT_EXTERN NSString *const UIDeviceOrientationDidChangeNotification;
//电池状态改变时发送的通知
UIKIT_EXTERN NSString *const UIDeviceBatteryStateDidChangeNotification   NS_AVAILABLE_IOS(3_0);
//电量改变时发送的通知
UIKIT_EXTERN NSString *const UIDeviceBatteryLevelDidChangeNotification   NS_AVAILABLE_IOS(3_0);
//距离传感器状态改变时发送的通知
UIKIT_EXTERN NSString *const UIDeviceProximityStateDidChangeNotification NS_AVAILABLE_IOS(3_0);
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

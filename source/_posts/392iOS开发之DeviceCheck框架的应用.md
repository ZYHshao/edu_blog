---
title: iOS开发之DeviceCheck框架的应用
date: 2019-06-24
categories: iOS逻辑初窥
tags: []
---
## iOS开发之DeviceCheck框架的应用

      DeviceCheck框架是iOS 11后提供的一个记录用户设备的工具框架。

在实际应用中，经常会遇到需要识别用户设备的需求，例如某些免费试用的应用程序，会根据设备判断用户是否已经试用过。Apple基于保护用户隐私的原则，开发者不能直接获取用户设备的相关标识信息，iOS 11后，Apple提供了DeviceCheck框架用来提供设备检查功能。

    DeviceCheck非常简单，大部分设备检查的逻辑要交给服务端调用Apple提供的接口来实现。

    DeviceCheck框架中只提供了一个类：DCDevice。其中定义如下：

```objectivec
@interface DCDevice : NSObject
// 类属性 获取实例对象
@property (class, readonly) DCDevice *currentDevice;
// 检查框架是否可用
@property (getter=isSupported, readonly) BOOL supported;
// 请求Token
- (void)generateTokenWithCompletionHandler:(void(^)(NSData * _Nullable token, NSError * _Nullable error))completion;
@end
```

DeviceCheck框架的核心在于获取设备的Token数据，拿到Token数据后可以仿照服务端发送推送的相关流程进行用户设备检查信息的读或写。详细文档地址如下：

[https://developer.apple.com/documentation/devicecheck/accessing\_and\_modifying\_per-device\_data](https://developer.apple.com/documentation/devicecheck/accessing_and_modifying_per-device_data)

    使用token进行设备检查时需要发送Query请求，参数如下：

    ![](https://oscimg.oschina.net/oscnet/83369ca91f61f83ab1eb4e79fda6f074e47.jpg)

在Apple返回的数据中会包含两个二进制的位和一个时间戳：

![](https://oscimg.oschina.net/oscnet/e3077e0b3e64c44201a4a05e8b51e9d53d7.jpg)

可以发现，其实Apple提供给开发者标记用户设备的能力十分有限，满打满算，开发者只能对用户设备标记4种状态。通过两个布尔位，用来获取当前设备是否参加了活动或者是否已经使用过试用资格等等。开发者也可以对这两个布尔值进行修改，上传请求的参数如下：

![](https://oscimg.oschina.net/oscnet/a7031da16b218ed79c778d237a916672d70.jpg)

---
title: iOS开发之AdSupport框架使用
date: 2018-09-10
categories: iOS逻辑初窥
tags: []
---
## iOS开发之AdSupport框架使用

    AdSupport从字面意思上理解是用来进行广告支持，这个框架十分简单，里面只有一个类，类中只有一个方法和两个属性。

    AdSupport的唯一用途是用来获取设备唯一的一个广告标识符。可以使用此标识符用来标记用户是否来源于某个广告推广，设备重启，重装应用程序都不会使广告标识符修改。

```objectivec
@interface ASIdentifierManager : NSObject
//获取单例管理类
+ (ASIdentifierManager * _Nonnull)sharedManager;
//获取广告标识符
@property (nonnull, nonatomic, readonly) NSUUID *advertisingIdentifier;
//用户是否同意跟踪广告标识符
@property (nonatomic, readonly, getter=isAdvertisingTrackingEnabled) BOOL advertisingTrackingEnabled;

@end
```

---
title: iOS10中更加灵活的色彩API
date: 2016-10-09
categories: iOS10专题
tags: []
---
## iOS10中更加灵活的色彩API

### 一、创建sRGB模式的色彩

      在iOS10中，UIColor类中新增加了两个方法，用来创建sRGB模式的色彩。与RGB相比，sRGB是更加标准的色彩模式，RGB色彩在不同设备上可能存在颜色偏差，sRGB则更加精准但同时色域范围也更窄一些。UIColor中新添加的方法如下：

```objectivec
//类方法创建sRGB模式色彩
+ (UIColor *)colorWithDisplayP3Red:(CGFloat)displayP3Red green:(CGFloat)green blue:(CGFloat)blue alpha:(CGFloat)alpha NS_AVAILABLE_IOS(10_0);
//初始化方法创建sRGB模式色彩
- (UIColor *)initWithDisplayP3Red:(CGFloat)displayP3Red green:(CGFloat)green blue:(CGFloat)blue alpha:(CGFloat)alpha NS_AVAILABLE_IOS(10_0);
```

### 二、全局的设置色彩风格

    一般情况下，iOS系统会根据用户所在环境的光线进行屏幕色彩的调节，在iOS10系统中，开发者可以在info.plist文件中全局的配置色彩风格来设置外界光线对APP内色彩的影响程度。

在info.plist文件中可以添加如下键：

White Point Adaptivity Style

这个键可以设置的值列举如下：

Standard White Point Adaptivity Style  标准色彩模式

Reading White Point Adaptivity Style   阅读色彩模式

Photo White Point Adaptivity Style      照片色彩模式

Video White Point Adaptivity Style      视频色彩模式

Game White Point Adaptivity Style      游戏色彩模式

上面几种模式从上到下，对色彩的保真度依次提高。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: iOS UISwitch 用法总结
date: 2015-04-15
categories: iOS之UI控件
tags: [UISwitch,iOS编程]              
---
iOS 系统开关控件简单使用总结：

初始化：

- (instancetype)initWithFrame:(CGRect)frame; 

这个frame是没有意义的，系统的开关控件大小是确定的。

设置开关开启状态时的颜色

@property(nonatomic, retain) UIColor *onTintColor;

设置开关风格颜色

@property(nonatomic, retain) UIColor *tintColor;

设置开关按钮颜色

@property(nonatomic, retain) UIColor *thumbTintColor;

设置开关开启状态时的图片（注意：在IOS7后不再起任何作用）

@property(nonatomic, retain) UIImage *onImage;

设置开关关闭状态时的图片（注意：在IOS7后不再起任何作用）

@property(nonatomic, retain) UIImage *offImage;

开关的状态

@property(nonatomic,getter=isOn) BOOL on;

手动设置开关状态

- (void)setOn:(BOOL)on animated:(BOOL)animated;

一点感想：iOS的系统的UISwitch控件虽然定制性很差，配合IOS7之后的扁平化和俭约的风格，在美观上确实不逊色于任何私人定制的开关控件，在没有特殊需求的情况下，对于开关逻辑，这是一个非常不错的UI交互选择。

学习使用 欢迎转载



专注技术，热爱生活，交流技术，也做朋友。

——珲少 QQ群：203317592

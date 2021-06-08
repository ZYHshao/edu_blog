---
title: iOS UIProgressView控件用法
date: 2015-04-16
categories: iOS之UI控件
tags: [UIProgressView,iOS编程]              
---
进度条控件是IOS开发中一个简单的系统控件，使用总结如下：

初始化一个进度条：



- (instancetype)initWithProgressViewStyle:(UIProgressViewStyle)style;

注意：1.用这个方式初始化的进度条系统会默认给一个长度。

         2.进度条的长度可以通过frame来设置，但是只有前三个参数有效。

         3.风格枚举如下：

typedef NS_ENUM(NSInteger, UIProgressViewStyle) {
    UIProgressViewStyleDefault,     // 普通样式
    UIProgressViewStyleBar,         // 用于工具条的样式
};



设置进度条风格样式



@property(nonatomic) UIProgressViewStyle progressViewStyle; 



设置进度条进度(0.0-1.0之间，默认为0.0)





@property(nonatomic) float progress;



设置已走过进度的进度条颜色





@property(nonatomic, retain) UIColor* progressTintColor;



设置未走过进度的进度条颜色





@property(nonatomic, retain) UIColor* trackTintColor;



设置进度条已走过进度的背景图案和为走过进度的背景图案(IOS7后好像没有效果了)





@property(nonatomic, retain) UIImage* progressImage;



@property(nonatomic, retain) UIImage* trackImage;



设置进度条进度和是否动画显示(动画显示会平滑过渡)





- (void)setProgress:(float)progress animated:(BOOL)animated;





学习使用 欢迎转载

专注技术，热爱生活，交流技术，也做朋友。

——珲少 QQ群：203317592

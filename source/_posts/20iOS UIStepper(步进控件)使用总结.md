---
title: iOS UIStepper(步进控件)使用总结
date: 2015-04-16
categories: iOS之UI控件
tags: [UIStepper,iOS编程]              
---
iOS中步进控件的简单使用

初始化控件

```
UIStepper * step = [[UIStepper alloc]initWithFrame:CGRectMake(100, 100, 100, 100)];
```

设置控制器值是否连续触发变化

@property(nonatomic,getter=isContinuous) BOOL continuous;

若设置为YES，则长按会连续触发变化，若设置为NO，只有在按击结束后，才会触发。

设置长按是否一直触发变化

@property(nonatomic) BOOL autorepeat; 

若设置为YES，则长按值会一直改变，若设置为NO，则一次点击只会改变一次值

设置控制器的值是否循环(到达边界后，重头开始，默认为NO)

@property(nonatomic) BOOL wraps;

设置控制器的值

@property(nonatomic) double value; 

设置控制器的最大值和最小值

@property(nonatomic) double minimumValue;//默认为0

@property(nonatomic) double maximumValue; //默认为100

设置控制器的步长

@property(nonatomic) double stepValue; 

设置控制器风格颜色

@property(nonatomic,retain) UIColor *tintColor;

设置控制器背景图片

\- (void)setBackgroundImage:(UIImage*)image forState:(UIControlState)state;

获取背景图片

\- (UIImage*)backgroundImageForState:(UIControlState)state;

通过左右按钮的状态设置分割线的图片

\- (void)setDividerImage:(UIImage*)image forLeftSegmentState:(UIControlState)leftState rightSegmentState:(UIControlState)rightState;

获取分割线图片

\- (UIImage*)dividerImageForLeftSegmentState:(UIControlState)state rightSegmentState:(UIControlState)state;

设置和获取加号按钮的图片

\- (void)setIncrementImage:(UIImage *)image forState:(UIControlState)state;

\- (UIImage *)incrementImageForState:(UIControlState)state;

  
设置和获取减号按钮的图片

\- (void)setDecrementImage:(UIImage *)image forState:(UIControlState)state;

\- (UIImage *)decrementImageForState:(UIControlState)state;

学习使用 欢迎转载

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

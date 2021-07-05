---
title: iOS动画开发之二——UIView动画执行的另一种方式
date: 2015-07-28
categories: iOS逻辑初窥
tags: []
---
## iOS动画开发之二——UIView动画执行的另一种方式

        上一篇博客中介绍了UIView的一些常用动画，通过block块，我们可以很方便简洁的创建出动画效果：[http://my.oschina.net/u/2340880/blog/484457](http://my.oschina.net/u/2340880/blog/484457)，这篇博客再介绍一种更加传统的执行UIView的动画的方法。

        这种方式相比如block的方式，显得要麻烦一些，apple官方也推荐我们使用带block的创建动画的方式，我们可以将编程重心更多的放在动画逻辑的实现上。使用begin和commit方式主要分为三个步骤：

    一、设置动画开始

```
[UIView beginAnimations:@"test" context:nil];
```

这个函数中的两个参数，第一个用于设置一个动画的标识id，通常第二个参数写为nil。

    二、动画执行的参数设置

\+ (void)setAnimationDelegate:(id)delegate; 

    设置这个动画的代理，用于执行动画开始或者结束后的动作

\+ (void)setAnimationWillStartSelector:(SEL)selector; 

    设置动画开始时执行的回调

\+ (void)setAnimationDidStopSelector:(SEL)selector;

    设置动画结束后执行的回调

\+ (void)setAnimationDuration:(NSTimeInterval)duration;

    设置动画执行的时间

\+ (void)setAnimationDelay:(NSTimeInterval)delay; 

    设置延时执行的延时

\+ (void)setAnimationStartDate:(NSDate *)startDate; 

    给动画设置一个启示时间

\+ (void)setAnimationCurve:(UIViewAnimationCurve)curve; 

    设置动画播放的线性效果，UIViewAnimationCurve的枚举如下：

```
typedef NS_ENUM(NSInteger, UIViewAnimationCurve) {
    UIViewAnimationCurveEaseInOut,         // 淡入淡出
    UIViewAnimationCurveEaseIn,            // 淡入
    UIViewAnimationCurveEaseOut,           // 淡出
    UIViewAnimationCurveLinear            //线性
}
```

\+ (void)setAnimationRepeatCount:(float)repeatCount; 

    设置动画循环播放次数

\+ (void)setAnimationRepeatAutoreverses:(BOOL)repeatAutoreverses;

    设置动画逆向执行

    三、提交动画

\+ (void)commitAnimations;

    例如：

```
UIView * view = [[UIView alloc]initWithFrame:CGRectMake(100, 100, 100, 100)];
    [self.view addSubview:view];
    view.backgroundColor=[UIColor redColor];
    [UIView beginAnimations:@"test" context:nil];
    [UIView setAnimationDuration:3];
    view.backgroundColor=[UIColor orangeColor];
    [UIView commitAnimations];//执行commit后，动画即开始执行
```

一点建议：这种创建UIView动画的方式和上一篇博客中的block方式效果相同，然而效率并不高，写的代码也会繁琐冗长，在开发中，如果没有特殊的兼容要求，使用block的方式会更高效方便。   

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

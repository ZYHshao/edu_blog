---
title: iOS动画开发之一——UIViewAnimation动画的使用
date: 2015-07-27
categories: iOS逻辑初窥
tags: []
---
## iOS动画开发之一——UIViewAnimation动画的使用

### 一、简介

      一款APP的成功与否，除了完善的功能外，用户体验也占有极大的比重，动画的合理运用，可以很好的增强用户体验。iOS开发中，常用的动画处理有UIView动画编程和核心动画编程，其中UIView动画使用简便，开发中应用十分广泛。这篇博客，主要讨论UIView的动画使用。

### 二、UIView动画的几个方法

\+ (void)animateWithDuration:(NSTimeInterval)duration animations:(void (^)(void))animations;

       这个是参数最少的一个方法，我们可以通过设置一个时间和block块来完成动画，时间参数是动画执行的时长，block块中为要执行的动画动作，具体可以执行那些动作，我们会在后面说。例如在1S内将view渐变透明：

```
[UIView animateWithDuration:1 animations:^{
        _myView.alpha=0;
    }];
```

\+ (void)animateWithDuration:(NSTimeInterval)duration animations:(void (^)(void))animations completion:(void (^)(BOOL finished))completion;

       这个函数会带两个block块，用法和第一个函数相似，设置一个执行时间和一个执行动作，第二个block块中可以添加一个动画执行结束后的动作，作为补充，例如下面代码的效果，在1S内将view渐变为透明，动画结束后，view在瞬间变回不透明：

```
[UIView animateWithDuration:1 animations:^{
        _myView.alpha=0;
    } completion:^(BOOL finished) {
        if (finished) {
            _myView.alpha=1;
        }
    }];
```

\+ (void)animateWithDuration:(NSTimeInterval)duration delay:(NSTimeInterval)delay options:(UIViewAnimationOptions)options animations:(void (^)(void))animations completion:(void (^)(BOOL finished))completion;

       这个函数除了上面的属性外，可以设置延时执行，同时可以设置一个动画效果参数，这个参数是个枚举，它可以影响动画的执行效果，后面会再总结。

\+ (void)animateWithDuration:(NSTimeInterval)duration delay:(NSTimeInterval)delay usingSpringWithDamping:(CGFloat)dampingRatio initialSpringVelocity:(CGFloat)velocity options:(UIViewAnimationOptions)options animations:(void (^)(void))animations completion:(void (^)(BOOL finished))completion;

     这个函数是iOS7之后的一个新函数，通过这个函数，我们可以方便的制作出效果炫酷的动画，这个函数的核心是两个阻尼参数，参数dampingRatio可以理解为弹簧效果的强弱，设置1则没有回弹效果，设置0则会剧烈的阻尼回弹。velocity参数用于设置弹簧的初始速度。

### 三、UIView动画可以操作的视图属性

       通过上面的介绍，我们了解了几个使用动画的函数，那么那些属性可以产生动画效果呢？

官方文档告诉我们这些属性是可以通过上述方法进行动画的：

![](http://static.oschina.net/uploads/space/2015/0727/220518_Bkqg_2340880.png)

### 四、动画执行选项设置

   在UIView执行动画的相关函数中，有UIViewAnimationOptions这个参数可以对动画的执行效果进行设置，这个枚举非常多，可分为三部分，如下：

```
enum {
   //这部分是基础属性的设置
   UIViewAnimationOptionLayoutSubviews            = 1 <<  0,//设置子视图随父视图展示动画
   UIViewAnimationOptionAllowUserInteraction      = 1 <<  1,//允许在动画执行时用户与其进行交互
   UIViewAnimationOptionBeginFromCurrentState     = 1 <<  2,//允许在动画执行时执行新的动画
   UIViewAnimationOptionRepeat                    = 1 <<  3,//设置动画循环执行
   UIViewAnimationOptionAutoreverse               = 1 <<  4,//设置动画反向执行，必须和重复执行一起使用
   UIViewAnimationOptionOverrideInheritedDuration = 1 <<  5,//强制动画使用内层动画的时间值
   UIViewAnimationOptionOverrideInheritedCurve    = 1 <<  6,//强制动画使用内层动画曲线值
   UIViewAnimationOptionAllowAnimatedContent      = 1 <<  7,//设置动画视图实时刷新
   UIViewAnimationOptionShowHideTransitionViews   = 1 <<  8,//设置视图切换时隐藏，而不是移除
   UIViewAnimationOptionOverrideInheritedOptions  = 1 <<  9,//
   //这部分属性设置动画播放的线性效果
   UIViewAnimationOptionCurveEaseInOut            = 0 << 16,//淡入淡出 首末减速
   UIViewAnimationOptionCurveEaseIn               = 1 << 16,//淡入 初始减速
   UIViewAnimationOptionCurveEaseOut              = 2 << 16,//淡出 末尾减速
   UIViewAnimationOptionCurveLinear               = 3 << 16,//线性 匀速执行   
   //这部分设置UIView切换效果
   UIViewAnimationOptionTransitionNone            = 0 << 20,
   UIViewAnimationOptionTransitionFlipFromLeft    = 1 << 20,//从左边切入
   UIViewAnimationOptionTransitionFlipFromRight   = 2 << 20,//从右边切入
   UIViewAnimationOptionTransitionCurlUp          = 3 << 20,//从上面立体进入
   UIViewAnimationOptionTransitionCurlDown        = 4 << 20,//从下面立体进入
   UIViewAnimationOptionTransitionCrossDissolve   = 5 << 20,//溶解效果
   UIViewAnimationOptionTransitionFlipFromTop     = 6 << 20,//从上面切入
   UIViewAnimationOptionTransitionFlipFromBottom  = 7 << 20,//从下面切入
};
```

提示：1，属性可以使用|进行多项合并。

          2，这类的动画可以进行嵌套，其中有一点需要注意，内层动画的执行时间和曲线模式会默认继承外层动的，若要强制使用新的参数，使用如下的两个参数：

```
UIViewAnimationOptionOverrideInheritedDuration = 1 <<  5,//强制动画使用内层动画的时间值
   UIViewAnimationOptionOverrideInheritedCurve    = 1 <<  6,//强制动画使用内层动画曲线值
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

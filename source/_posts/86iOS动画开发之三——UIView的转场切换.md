---
title: iOS动画开发之三——UIView的转场切换
date: 2015-07-28
categories: iOS逻辑初窥
tags: []
---
## iOS动画开发之三——UIView的转场切换

        前两篇博客中，我们分别介绍了UIView动画的两种使用方式，分别为，带block的方式：[**http://my.oschina.net/u/2340880/blog/484457**](http://my.oschina.net/u/2340880/blog/484457) ,传统的属性配置的方式：[http://my.oschina.net/u/2340880/blog/484538](http://my.oschina.net/u/2340880/blog/484538)。通过UIView动画的类方法，我们可以十分方便的使View某些属性改变的同时拥有动画效果。这篇博客主要讨论View切换的动画操作。

        两个方法：

\+ (void)transitionWithView:(UIView *)view duration:(NSTimeInterval)duration options:(UIViewAnimationOptions)options animations:(void (^)(void))animations completion:(void (^)(BOOL finished))completion;

   通过这个方法，我们可以重绘View视图，任何其子视图的改变或者其自身的改变都会触发转场动画的效果， 系统提供的转场效果在第一篇博客中已经介绍过。

       这个方法常用于类似小说软件的翻页效果。

\+ (void)transitionFromView:(UIView *)fromView toView:(UIView *)toView duration:(NSTimeInterval)duration options:(UIViewAnimationOptions)options completion:(void (^)(BOOL finished))completion;

    这个方法会作用于fromView的父视图，用于切换两个view，通过执行这个方法，会将formView从其父视图上移除，将toView重新粘在其父视图上，展现一个动画效果。

    通过使用上述两个方法，你会发现某些效果会非常突兀，比如想要改变视图的颜色，它会在转场动画播放完成后，颜色突然的变化，要改善这一效果，我们需要设置options参数包含：UIViewAnimationOptionAllowAnimatedContent这个枚举。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: iOS中UIImageView用法总结
date: 2015-06-03
categories: iOS之UI控件
tags: []
---
## iOS中UIImageView用法总结

\- (instancetype)initWithImage:(UIImage *)image;

通过一个图片UIImage对象进行初始化

\- (instancetype)initWithImage:(UIImage *)image highlightedImage:(UIImage *)highlightedImage;

通过一个正常状态下的图片和高亮状态下的图片初始化对象

[@property](http://my.oschina.net/property)(nonatomic,retain) UIImage *image;

设置正常状态下的图片

[@property](http://my.oschina.net/property)(nonatomic,retain) UIImage *highlightedImage;

设置高亮状态下的图片

[@property](http://my.oschina.net/property)(nonatomic,getter=isUserInteractionEnabled) BOOL userInteractionEnabled; 

设置是否开启用户交互

[@property](http://my.oschina.net/property)(nonatomic,getter=isHighlighted) BOOL highlighted;

设置是否为高亮状态

[@property](http://my.oschina.net/property)(nonatomic,copy) NSArray *animationImages;

设置正常状态下的动画图片数组

@property(nonatomic,copy) NSArray *highlightedAnimationImages;

设置高亮状态下的动画图片数组

@property(nonatomic) NSTimeInterval animationDuration;

设置动画播放时长 默认频率为30帧每秒

@property(nonatomic) NSInteger      animationRepeatCount; 

设置动画循环播放次数 默认为无限循环

\- (void)startAnimating;

开始播放帧动画

\- (void)stopAnimating;

停止播放帧动画

\- (BOOL)isAnimating;

是否正在播放动画

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

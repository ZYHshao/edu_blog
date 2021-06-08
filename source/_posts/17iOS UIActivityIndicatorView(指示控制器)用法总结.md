---
title: iOS UIActivityIndicatorView(指示控制器)用法总结
date: 2015-04-15
categories: iOS之UI控件
tags: [UIActivityIndicatorView,iOS编程]              
---
对于UIActivityIndicatorView的使用，我们一般会创建一个背景View,设置一定的透明度，然后将UIActivityIndicatorView贴在背景View上，在我们需要的时候将这个view呼出。

初始化UIActivityIndicatorView

- (instancetype)initWithActivityIndicatorStyle:(UIActivityIndicatorViewStyle)style;

这个风格是一个枚举，如下

typedef NS_ENUM(NSInteger, UIActivityIndicatorViewStyle) {
    //大号白色
    UIActivityIndicatorViewStyleWhiteLarge,
    //白色
    UIActivityIndicatorViewStyleWhite,
    //灰色
    UIActivityIndicatorViewStyleGray,
};

初始化之后，还需要给它一个Frame，但是只有前两个位置参数有效，大小参数将没有任何影响。


设置指示器风格：

@property(nonatomic) UIActivityIndicatorViewStyle activityIndicatorViewStyle; 

设置指示器是否停止动画时隐藏

@property(nonatomic) BOOL  hidesWhenStopped; 

设置指示器颜色

@property (readwrite, nonatomic, retain) UIColor *color；

让指示器开始动画

- (void)startAnimating;

让指示器停止动画

- (void)stopAnimating;

获取指示器动画状态

- (BOOL)isAnimating;



学习使用 欢迎转载

专注技术，热爱生活，交流技术，也做朋友。

——珲少 QQ群：203317592

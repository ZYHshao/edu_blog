---
title: iOS开发UINavigation系列三——工具栏UIToolBar
date: 2015-11-09
categories: iOS之UI控件
tags: []
---
## iOS开发UINavigation系列三——工具栏UIToolBar

        iOS中除了UINavinationBar之外，还有工具栏UIToolBar可以供我们使用，工具栏和导航栏十分类似，只是功能更加简单，工具栏中也有UIBarButtonItem按钮，在前两篇博客中，对导航栏和导航项都进行的讨论，地址如下：

UINavigationBar：[http://my.oschina.net/u/2340880/blog/527706](http://my.oschina.net/u/2340880/blog/527706)

UINavigationItem:[http://my.oschina.net/u/2340880/blog/527781](http://my.oschina.net/u/2340880/blog/527781)

        导航栏一般会出现在视图的头部，与之相对，工具栏一般会出现在视图的的底部，上面可以填充一些按钮，提供给用户一些操作。创建一个工具栏如下：

```
    self.view.backgroundColor = [UIColor grayColor];
    UIToolbar * tool = [[UIToolbar alloc]initWithFrame:CGRectMake(0, self.view.frame.size.height-40, 320, 40)];
    [self.view addSubview:tool];
```

![](http://static.oschina.net/uploads/space/2015/1109/174300_XgJo_2340880.png)

下面是UIToolBar中的一些方法，其中大部分在UINavigationBar中都有涉及，这里只做简单的介绍：

```
//工具栏的风格，和导航栏类似，有黑白两种
@property(nonatomic) UIBarStyle barStyle; 
//设置工具栏上按钮数组
@property(nullable,nonatomic,copy) NSArray<UIBarButtonItem *> *items; 
//设置工具栏是否透明
@property(nonatomic,assign,getter=isTranslucent) BOOL translucent; 
//设置工具栏按钮
- (void)setItems:(nullable NSArray<UIBarButtonItem *> *)items animated:(BOOL)animated; 
//设置item风格颜色
@property(null_resettable, nonatomic,strong) UIColor *tintColor;
//设置工具栏背景色
@property(nullable, nonatomic,strong) UIColor *barTintColor;
//设置工具栏背景和阴影图案
- (void)setBackgroundImage:(nullable UIImage *)backgroundImage forToolbarPosition:(UIBarPosition)topOrBottom barMetrics:(UIBarMetrics)barMetrics;
- (nullable UIImage *)backgroundImageForToolbarPosition:(UIBarPosition)topOrBottom barMetrics:(UIBarMetrics)barMetrics;
- (void)setShadowImage:(nullable UIImage *)shadowImage forToolbarPosition:(UIBarPosition)topOrBottom;
- (nullable UIImage *)shadowImageForToolbarPosition:(UIBarPosition)topOrBottom;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: iOS中UIActionSheet使用详解
date: 2015-05-03
categories: iOS之UI控件
tags: []
---
## IOS中UIActionSheet使用方法详解

### 一、初始化方法

\- (instancetype)initWithTitle:(NSString *)title delegate:(id<UIActionSheetDelegate>)delegate cancelButtonTitle:(NSString *)cancelButtonTitle destructiveButtonTitle:(NSString *)destructiveButtonTitle otherButtonTitles:(NSString *)otherButtonTitles, ...;

参数说明：

title：视图标题

delegate：设置代理

cancelButtonTitle：取消按钮的标题

destructiveButtonTitle：特殊标记的按钮的标题

otherButtonTitles：其他按钮的标题

## 二、常用方法和属性介绍

[@property](http://my.oschina.net/property)(nonatomic,copy) NSString *title;

设置标题

[@property](http://my.oschina.net/property)(nonatomic) UIActionSheetStyle actionSheetStyle;

设置风格，枚举如下：

```
typedef NS_ENUM(NSInteger, UIActionSheetStyle) {
    UIActionSheetStyleAutomatic        = -1,      
    UIActionSheetStyleDefault          = UIBarStyleDefault,
    UIActionSheetStyleBlackTranslucent = UIBarStyleBlackTranslucent,
    UIActionSheetStyleBlackOpaque      = UIBarStyleBlackOpaque,
};
```

\- (NSInteger)addButtonWithTitle:(NSString *)title;

添加一个按钮，会返回按钮的索引

\- (NSString *)buttonTitleAtIndex:(NSInteger)buttonIndex;

获取按钮标题

[@property](http://my.oschina.net/property)(nonatomic,readonly) NSInteger numberOfButtons;

获取按钮数量

[@property](http://my.oschina.net/property)(nonatomic) NSInteger cancelButtonIndex;

设置取消按钮的索引值

[@property](http://my.oschina.net/property)(nonatomic) NSInteger destructiveButtonIndex;

设置特殊标记

@property(nonatomic,readonly,getter=isVisible) BOOL visible;

视图当前是否可见

下面是几种弹出方式，会根据风格不同展现不同的方式

\- (void)showFromToolbar:(UIToolbar *)view;

\- (void)showFromTabBar:(UITabBar *)view;

\- (void)showFromBarButtonItem:(UIBarButtonItem *)item animated:(BOOL)animated ;

\- (void)showFromRect:(CGRect)rect inView:(UIView *)view animated:(BOOL)animated ;

\- (void)showInView:(UIView *)view;

\- (void)dismissWithClickedButtonIndex:(NSInteger)buttonIndex animated:(BOOL)animated;

使用代码将视图收回

### 三、UIActionSheet代理方法

\- (void)actionSheet:(UIActionSheet *)actionSheet clickedButtonAtIndex:(NSInteger)buttonIndex;

点击按钮时触发的方法

\- (void)willPresentActionSheet:(UIActionSheet *)actionSheet; 

视图将要弹出时触发的方法

\- (void)didPresentActionSheet:(UIActionSheet *)actionSheet;

视图已经弹出式触发的方法

\- (void)actionSheet:(UIActionSheet *)actionSheet willDismissWithButtonIndex:(NSInteger)buttonIndex;

点击按钮后，视图将要收回时触发的方法

\- (void)actionSheet:(UIActionSheet *)actionSheet didDismissWithButtonIndex:(NSInteger)buttonIndex;

点击按钮后，视图已经收回时触发的方法

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

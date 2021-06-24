---
title: iOS UIAlertView（警告框）方法总结
date: 2015-05-01
categories: iOS之UI控件
tags: []
---
## IOS中UIAlertView(警告框)常用方法总结

### 一、初始化方法

### \- (instancetype)initWithTitle:(NSString *)title message:(NSString *)message delegate:(id /*<UIAlertViewDelegate>*/)delegate cancelButtonTitle:(NSString *)cancelButtonTitle otherButtonTitles:(NSString *)otherButtonTitles, ...;

这个方法通过设置一个标题，内容，代理和一些按钮的标题创建警告框，代码示例如下：

```
    UIAlertView * alert = [[UIAlertView alloc]initWithTitle:@"我的警告框" message:@"这是一个警告框" delegate:self cancelButtonTitle:@"取消" otherButtonTitles:@"确定", nil];
    [alert show];
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/0501/130508_XklY_2340880.png)

注意：如果按钮数超过两个，将会创建成如下样子：

![](http://static.oschina.net/uploads/space/2015/0501/130912_SSoj_2340880.png)

如果按钮数量超出屏幕显示范围，则会创建类似tableView的效果。

### 二、属性与方法解析

标题属性

[@property](http://my.oschina.net/property)(nonatomic,copy) NSString *title;

内容属性

[@property](http://my.oschina.net/property)(nonatomic,copy) NSString *message;

添加一个按钮，返回的是此按钮的索引值

\- (NSInteger)addButtonWithTitle:(NSString *)title;   

返回根据按钮索引按钮标题 

\- (NSString *)buttonTitleAtIndex:(NSInteger)buttonIndex;

获取按钮数量

[@property](http://my.oschina.net/property)(nonatomic,readonly) NSInteger numberOfButtons;

设置将某一个按钮设置为取消按钮

[@property](http://my.oschina.net/property)(nonatomic) NSInteger cancelButtonIndex;

返回其他类型按钮第一个的索引值

[@property](http://my.oschina.net/property)(nonatomic,readonly) NSInteger firstOtherButtonIndex;

警告框是否可见

@property(nonatomic,readonly,getter=isVisible) BOOL visible;

显现警告框

\- (void)show;

代码模拟点击按钮消失触发方法

\- (void)dismissWithClickedButtonIndex:(NSInteger)buttonIndex animated:(BOOL)animated;

设置警告框风格

@property(nonatomic,assign) UIAlertViewStyle alertViewStyle;

风格的枚举如下

```
typedef NS_ENUM(NSInteger, UIAlertViewStyle) {
    UIAlertViewStyleDefault = 0,//默认风格
    UIAlertViewStyleSecureTextInput,//密码输入框风格
    UIAlertViewStylePlainTextInput,//普通输入框风格
    UIAlertViewStyleLoginAndPasswordInput//账号密码框风格
};
```

这个方法设置文本输入框的索引

\- (UITextField *)textFieldAtIndex:(NSInteger)textFieldIndex;

### 三、UIAlertViewDelegate中的方法

点击按钮时触发的方法

\- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex;

将要展现警告框时触发的方法

\- (void)willPresentAlertView:(UIAlertView *)alertView;

已经展现警告框时触发的方法

\- (void)didPresentAlertView:(UIAlertView *)alertView;

警告框将要消失时触发的方法

\- (void)alertView:(UIAlertView *)alertView willDismissWithButtonIndex:(NSInteger)buttonIndex;

警告框已经消失时触发的方法

\- (void)alertView:(UIAlertView *)alertView didDismissWithButtonIndex:(NSInteger)buttonIndex; 

设置是否允许第一个按钮不是取消按钮

\- (BOOL)alertViewShouldEnableFirstOtherButton:(UIAlertView *)alertView;

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

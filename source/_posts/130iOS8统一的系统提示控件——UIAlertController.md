---
title: iOS8统一的系统提示控件——UIAlertController
date: 2015-10-17
categories: iOS之UI控件
tags: []
---
## iOS8统一的系统提示控件——UIAlertController

### 一、引言

        相信在iOS开发中，大家对UIAlertView和UIActionSheet一定不陌生，这两个控件在UI设计中发挥了很大的作用。然而如果你用过，你会发现这两个控件的设计思路有些繁琐，通过创建设置代理来进行界面的交互，将代码逻辑分割了，并且很容易形成冗余代码。在iOS8之后，系统吸引了UIAlertController这个类，整理了UIAlertView和UIActionSheet这两个控件，在iOS中，如果你扔使用UIAlertView和UIActionSheet，系统只是会提示你使用新的方法，iOS9中，这两个类被完全弃用，但这并不说明旧的代码将不能使用，旧的代码依然可以工作很好，但是会存在隐患，UIAlertController，不仅系统推荐，使用更加方便，结构也更加合理，作为开发者，使用新的警示控件，我们何乐而不为呢。这里有旧的代码的使用方法：

UIAlertView使用：[http://my.oschina.net/u/2340880/blog/408873](http://my.oschina.net/u/2340880/blog/408873)。

UIActionSheet使用：[http://my.oschina.net/u/2340880/blog/409907](http://my.oschina.net/u/2340880/blog/409907)。

### 二、UIAlertController的使用

        从这个类的名字我们就可以看出，对于警示控件，设计的思路不再是View而是Controller。通过present和push进行呼出，而不是以前的show方法。另一个机制改变的地方是，其中按钮的触发方法不再通过代理处理，而是将按钮封装成了类:UIAlertAction。详细方法及使用如下：

```
 UIAlertController * con = [UIAlertController alertControllerWithTitle:@"新的" message:@"看看样子" preferredStyle:UIAlertControllerStyleAlert];
    [con addAction:[UIAlertAction actionWithTitle:@"仔细看" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
       //按钮触发的方法
    }]];
     [self presentViewController:con animated:YES completion:nil];
```

上面的代码，会在屏幕上呼出警告框，如下：

![](http://static.oschina.net/uploads/space/2015/1017/194732_LhTJ_2340880.png)

初始化方法中的preferref参数是一个枚举，决定是提示框或者抽屉列表：

```
typedef NS_ENUM(NSInteger, UIAlertControllerStyle) {
    UIAlertControllerStyleActionSheet = 0,//抽屉
    UIAlertControllerStyleAlert//警告框
}
```

上面的addAction方法添加了一个封装了方法的按钮，UIAlertAction类的构造十分简单，如下：

```
//初始化方法
+ (instancetype)actionWithTitle:(nullable NSString *)title style:(UIAlertActionStyle)style handler:(void (^ __nullable)(UIAlertAction *action))handler;
//获取标题
@property (nullable, nonatomic, readonly) NSString *title;
//获取风格
@property (nonatomic, readonly) UIAlertActionStyle style;
//设置是否有效
@property (nonatomic, getter=isEnabled) BOOL enabled;
```

AlertAction的风格是如下的枚举：

```
typedef NS_ENUM(NSInteger, UIAlertActionStyle) {
    UIAlertActionStyleDefault = 0,//默认的风格
    UIAlertActionStyleCancel,//取消按钮的风格
    UIAlertActionStyleDestructive//警告的风格
}
```

风格效果如下：

![](http://static.oschina.net/uploads/space/2015/1017/195742_Kaxz_2340880.png)

### 三、UIAlertController其他属性和方法

[@property](http://my.oschina.net/property) (nonatomic, readonly) NSArray<UIAlertAction *\> *actions;

获取所有AlertAction

[@property](http://my.oschina.net/property) (nonatomic, strong, nullable) UIAlertAction *preferredAction NS\_AVAILABLE\_IOS(9_0);

iOS9后新增加的属性，可以使某个按钮更加突出，只能设置已经在actions数组中的AkertAction，会使设置的按钮更加显眼，如下：

![](http://static.oschina.net/uploads/space/2015/1017/200351_DeEZ_2340880.png)     

\- (void)addTextFieldWithConfigurationHandler:(void (^ __nullable)(UITextField *textField))configurationHandler;

添加一个textField，以前的相关控件，虽然也可以添加textField，但是定制化能力非常差，这个新的方法中有一个configurationHandler代码块，可以将textField的相关设置代码放入这个代码块中，并且这个方法添加的textField个数不再限制于2个：

```
 [con addTextFieldWithConfigurationHandler:^(UITextField * _Nonnull textField) {
        textField.placeholder=@"第1个";
    }];
    [con addTextFieldWithConfigurationHandler:^(UITextField * _Nonnull textField) {
        textField.placeholder=@"第2个";
    }];
    [con addTextFieldWithConfigurationHandler:^(UITextField * _Nonnull textField) {
        textField.placeholder=@"第3个";
    }];
```

![](http://static.oschina.net/uploads/space/2015/1017/201411_W9NY_2340880.png)

[@property](http://my.oschina.net/property) (nullable, nonatomic, readonly) NSArray<UITextField *\> *textFields;

获取所有textField的数组

[@property](http://my.oschina.net/property) (nullable, nonatomic, copy) NSString *title;

设置警示控件的标题

[@property](http://my.oschina.net/property) (nullable, nonatomic, copy) NSString *message;

设置警示控件的信息

@property (nonatomic, readonly) UIAlertControllerStyle preferredStyle;

获取警示控件的风格  

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

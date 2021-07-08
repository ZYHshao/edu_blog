---
title: iOS系统菜单控制器UIMenuController使用简介
date: 2016-04-06
categories: iOS逻辑初窥
tags: []
---
## iOS系统菜单控制器UIMenuController使用简介

### 一、引言

   在许多iOS应用中，当用户进行某文字或图片区域的长按操作时，都会弹出一个系统菜单控件，用户可以通过操作菜单控件上的按钮进行数据的复制、剪切、粘贴等操作。系统原生的某些控件已经支持了对UIMenuController的唤出操作，然而并不是所有控件都支持，开发者可以通过自定义UIMenuController来更加灵活的使用菜单控件，在前面博客中有介绍iOS剪切板相关知识，地址如下：

iOS剪切板UIPasteboard使用简介：[http://my.oschina.net/u/2340880/blog/653228](http://my.oschina.net/u/2340880/blog/653228)。

### 二、UIMenuController的使用

   UIMenuController的展现需要基于一个View视图，其交互则需要基于其所在View视图的Responder。举例来说，如果一个UIMenuController展现在当前ViewController的View上，则此UIMenuController的交互逻辑交由当前的ViewController进行管理。

    在界面展示出UIMenuController需要3个条件：

    1.当前的Responder处于第一响应。

    2.UIMenuController对象调用menuVisible方法。

    3.当前的Responder实现了如下两个方法：

```
//是否可以成为第一相应
-(BOOL)canBecomeFirstResponder{
    return YES;
}
//是否可以接收某些菜单的某些交互操作
-(BOOL)canPerformAction:(SEL)action withSender:(id)sender{
        return YES;
}
```

实现了上面的两个方法，使用如下的代码可以唤出UIMenuController控件：

```
    [self becomeFirstResponder];
    //设置菜单显示的位置 frame设置其文职 inView设置其所在的视图
    [[UIMenuController sharedMenuController] setTargetRect:frame inView:self.view];
    //将菜单控件设置为可见
    [UIMenuController sharedMenuController].menuVisible = YES;
```

在执行了上面的代码后，系统第一次调用canperformAction:withSender:方法会进行是否显示菜单栏的检测，如果返回为NO，则不能显示菜单栏，如果返回为YES，之后系统会多次调用canPerformAction:withSender:方法，用于检测当前Responder对象是否实现了菜单栏上某个选项的触发方法，如果实现了，菜单栏上面的相应按钮会显示，否则不会显示。开发者可以在这个方法中通过判断action来确定菜单控件中显示的按钮种类。系统默认为开发者提供了一系列的菜单按钮，例如要显示剪切和赋值操作的菜单按钮，示例代码如下：

```
-(BOOL)canPerformAction:(SEL)action withSender:(id)sender{
    if (action == @selector(cut:)||action == @selector(copy:)) {
        return YES;
   }
    return NO;
}
```

效果如下图所示：

![](http://static.oschina.net/uploads/space/2016/0406/183315_3uWx_2340880.png)

系统默认支持提供的按钮触发方法列举如下：

```
//剪切按钮的方法
- (void)cut:(nullable id)sender NS_AVAILABLE_IOS(3_0);
//复制按钮的方法
- (void)copy:(nullable id)sender NS_AVAILABLE_IOS(3_0);
//粘贴按钮的方法
- (void)paste:(nullable id)sender NS_AVAILABLE_IOS(3_0);
//选择按钮的方法
- (void)select:(nullable id)sender NS_AVAILABLE_IOS(3_0);
//全选按钮的方法
- (void)selectAll:(nullable id)sender NS_AVAILABLE_IOS(3_0);
//删除按钮的方法
- (void)delete:(nullable id)sender NS_AVAILABLE_IOS(3_2);
//改变书写模式为从左向右按钮触发的方法
- (void)makeTextWritingDirectionLeftToRight:(nullable id)sender NS_AVAILABLE_IOS(5_0);
//改变书写模式为从右向左按钮触发的方法
- (void)makeTextWritingDirectionRightToLeft:(nullable id)sender NS_AVAILABLE_IOS(5_0);
```

上面所列举的方法声明在UIResponder头文件中，实际上，除了上面的方法，关于UIMenuController上面的按钮，系统中还有许多私有方法，列举如下：

```
//替换按钮
- (void)_promptForReplace:(id)arg1{
    NSLog(@"promptForReplace");
}
//简体繁体转换按钮
-(void)_transliterateChinese:(id)sender{
    NSLog(@"transliterateChinese");
}
//文字风格按钮
-(void)_showTextStyleOptions:(id)sender{
    NSLog(@"showTextStyleOptions");
}
//定义按钮
-(void)_define:(id)sender{
    NSLog(@"define");
}
-(void)_addShortcut:(id)sender{
    NSLog(@"addShortcut");
}
-(void)_accessibilitySpeak:(id)sender{
    NSLog(@"accessibilitySpeak");
}
//语言选择按钮
-(void)_accessibilitySpeakLanguageSelection:(id)sender{
    NSLog(@"accessibilitySpeakLanguageSelection");
}
//暂停发音按钮
-(void)_accessibilityPauseSpeaking:(id)sender{
    NSLog(@"accessibilityPauseSpeaking");
}
//分享按钮
-(void)_share:(id)sender{
    NSLog(@"share");
}
```

   在实际开发中，开发这完全不需要使用这些私有的方法，UIMenuItem类提供给开发者进行自定义菜单按钮与触发方法，示例如下：

```
[self becomeFirstResponder];
    UIMenuItem * item = [[UIMenuItem alloc]initWithTitle:@"自定义" action:@selector(newFunc)];
    [[UIMenuController sharedMenuController] setTargetRect:[sender frame] inView:self.view];
    [UIMenuController sharedMenuController].menuItems = @[item];
    [UIMenuController sharedMenuController].menuVisible = YES;
```

```
-(BOOL)canBecomeFirstResponder{
    return YES;
}
-(BOOL)canPerformAction:(SEL)action withSender:(id)sender{
    if (action == @selector(newFunc)) {
        return YES;
   }
    return NO;
}
-(void)newFunc{
    NSLog(@"自定义方法");
}
```

效果如下图所示：

![](http://static.oschina.net/uploads/space/2016/0406/184947_k1fB_2340880.png)

UIMenuController还有如下的属性用来设置其显示的位置：

```
//显示的位置
@property(nonatomic) UIMenuControllerArrowDirection arrowDirection;
//枚举如下：
/*
typedef NS_ENUM(NSInteger, UIMenuControllerArrowDirection) {
    //默认 基于当前屏幕状态
    UIMenuControllerArrowDefault, // up or down based on screen location
    //箭头在上的显示模式
    UIMenuControllerArrowUp NS_ENUM_AVAILABLE_IOS(3_2),
    //箭头在下的显示模式
    UIMenuControllerArrowDown NS_ENUM_AVAILABLE_IOS(3_2),
    //箭头在左的显示模式
    UIMenuControllerArrowLeft NS_ENUM_AVAILABLE_IOS(3_2),
    //箭头在右的显示模式
    UIMenuControllerArrowRight NS_ENUM_AVAILABLE_IOS(3_2),
};
*/
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: NSAlert组件应用总结
date: 2017-07-13
categories: macOS开发
tags: []
---
## NSAlert组件应用总结

### 一、引言

    在桌面软件开发中，当用户进行非法的操作或有风险的操作时，时长需要弹出警告框来提示用户。在OS X系统上，NSAlert是专门的警告框组件。其提供了简洁的接口供开发者进行使用。

### 二、NSAlert的简单使用

    使用警告框最简单的使用方式是提示错误信息，错误信息警告只起到提示用户的作用，其只有一个OK按钮，点击后警告框会关闭。示例如下：

```objectivec
- (IBAction)alert:(id)sender {
    NSError * error = [NSError errorWithDomain:@"testError" code:1001 userInfo:@{@"userid":@"1000"}];
    NSAlert * alert = [NSAlert alertWithError:error];
    [alert runModal];
}
```

效果图如下：

![](https://static.oschina.net/uploads/space/2017/0713/113916_zbny_2340880.png)

警告框的展现有两种方式，分别为模态窗与弹出抽屉。弹出抽屉会显示在当前绑定的窗口上，模态窗则会自成窗口，弹出在屏幕中央。

    你也可以对警告框进行自定义设置，例如文本，标题，图标等，示例如下：

```objectivec
- (IBAction)alert:(id)sender {
    NSAlert * alert = [[NSAlert alloc]init];
    alert.icon = [NSImage imageNamed:@"icon"];
    alert.messageText = @"警告信息";
    alert.informativeText = @"额外提供的内容";
    alert.showsHelp = YES;
    alert.helpAnchor = @"mac";
    alert.alertStyle = NSAlertStyleInformational;
    alert.showsSuppressionButton = YES;
    [alert.suppressionButton setTarget:self];
    [alert.suppressionButton setAction:@selector(click)];
    [alert addButtonWithTitle:@"1"];
    [alert addButtonWithTitle:@"2"];
    [alert addButtonWithTitle:@"3"];
    [alert addButtonWithTitle:@"4"];
    long res =  [alert runModal];
    NSLog(@"%ld",res);
}
```

效果如下：

![](https://static.oschina.net/uploads/space/2017/0713/153817_EVm5_2340880.png)

### 三、NSAlert属性与方法解析

NSAlert类中的属性和方法解析如下：

```objectivec
//直接使用错误信息创建警告框
+ (NSAlert *)alertWithError:(NSError *)error;
//设置警告框信息
@property (copy) NSString *messageText;
//设置额外信息内容
@property (copy) NSString *informativeText;
//设置警告框图标
@property (null_resettable, strong) NSImage *icon;
//向警告框中添加按钮
- (NSButton *)addButtonWithTitle:(NSString *)title;
//按钮数组
@property (readonly, copy) NSArray<NSButton *> *buttons;
//是否显示帮助按钮
@property BOOL showsHelp;
//设置帮助手册锚点 用于定于
@property (nullable, copy) NSString *helpAnchor;
//设置警告框风格
/*
typedef NS_ENUM(NSUInteger, NSAlertStyle) {
    NSAlertStyleWarning = 0,     //警告风格
    NSAlertStyleInformational = 1,//提示信息风格
    NSAlertStyleCritical = 2    //验证风格
};
*/
@property NSAlertStyle alertStyle;
//是否显示不再提示按钮
@property BOOL showsSuppressionButton NS_AVAILABLE_MAC(10_5);
//获取不再提示按钮
@property (nullable, readonly, strong) NSButton *suppressionButton NS_AVAILABLE_MAC(10_5);
//代理对象
@property (nullable, weak) id<NSAlertDelegate> delegate;
//以模态窗口的方式弹出警告框，这个方法是同步的，当用户点击警告框中按钮后会返回，返回的NSModalResponse实际上是
//整型数据，第1个按钮为1000，后面一次递增，如1001，1002...
- (NSModalResponse)runModal;
//以窗口抽屉的方式弹出警告框，这个方法是异步的，当用户点击警告框中的按钮后会回调block
- (void)beginSheetModalForWindow:(NSWindow *)sheetWindow completionHandler:(void (^ __nullable)(NSModalResponse returnCode))handler NS_AVAILABLE_MAC(10_9);
```

NSAlertDelegate协议中只定义了一个方法，如下：

```objectivec
@protocol NSAlertDelegate <NSObject>
@optional
//当用户点击帮助按钮后回调的方法 返回值决定是否弹出帮助窗口
- (BOOL)alertShowHelp:(NSAlert *)alert;
@end
```

除了上面列出的方法外，NSAlert中还有两个已经弃用的便捷构造和弹出方法，如下：

```objectivec
//创建警告框
+ (NSAlert *)alertWithMessageText:(nullable NSString *)message defaultButton:(nullable NSString *)defaultButton alternateButton:(nullable NSString *)alternateButton otherButton:(nullable NSString *)otherButton informativeTextWithFormat:(NSString *)format, ...;
//弹出警告框
- (void)beginSheetModalForWindow:(NSWindow *)window modalDelegate:(nullable id)delegate didEndSelector:(nullable SEL)didEndSelector contextInfo:(nullable void *)contextInfo;
```

---
title: OS X开发：NSButton按钮控件应用
date: 2017-07-18
categories: macOS开发
tags: []
---
## OS X开发：NSButton按钮控件应用

    NSButton控件用来创建功能按钮，和UIButton相比，其样式要丰富许多。NSButton继承自NSControl，其使用setTarget与setAction来添加触发方法，如下：

```objectivec
    NSButton * btn = [[NSButton alloc]initWithFrame:CGRectMake(50, 300, 90, 25)];
    [btn setTarget:self];
    [btn setAction:@selector(click)];
    [self.view addSubview:btn];
```

NSButton类中常用属性和方法解析如下:

```objectivec
//设置按钮标题
@property (copy) NSString *title;
//设置按钮开启状态的标题
@property (copy) NSString *alternateTitle;
//设置按钮图片
@property (nullable, strong) NSImage *image;
//设置按钮开启状态图片
@property (nullable, strong) NSImage *alternateImage;
//设置按钮图片位置
/*
typedef NS_ENUM(NSUInteger, NSCellImagePosition) {
    NSNoImage                           = 0,            //无图片
    NSImageOnly                         = 1,            //只有图片
    NSImageLeft                         = 2,            //图片在左侧
    NSImageRight                        = 3,            //图片在右侧
    NSImageBelow                        = 4,            //图片在下侧
    NSImageAbove                        = 5,            //图片在上侧
    NSImageOverlaps                     = 6,            //图片重叠
    NSImageLeading  API_AVAILABLE(macosx(10.12)) = 7,   //正方向
    NSImageTrailing API_AVAILABLE(macosx(10.12)) = 8    //逆方向
};
*/
@property NSCellImagePosition imagePosition;
//设置图片缩放模式
/*
typedef NS_ENUM(NSUInteger, NSImageScaling) {
    NSImageScaleProportionallyDown = 0, // 下方缩放
    NSImageScaleAxesIndependently,      // 整体缩放
    NSImageScaleNone,                   // 不缩放.
    NSImageScaleProportionallyUpOrDown, // 上下缩放
    
    NSScaleProportionally NS_ENUM_DEPRECATED_MAC(10_0, 10_10, "Use NSImageScaleProportionallyDown instead") = 0,
    NSScaleToFit NS_ENUM_DEPRECATED_MAC(10_0, 10_10, "Use NSImageScaleAxesIndependently instead"),
    NSScaleNone NS_ENUM_DEPRECATED_MAC(10_0, 10_10, "Use NSImageScaleNone instead")
};
*/
@property NSImageScaling imageScaling NS_AVAILABLE_MAC(10_5);
//图片是否环绕标题
@property BOOL imageHugsTitle;
//设置按钮状态
/*
enum {
    NSMixedState = -1,  //混合状态
    NSOffState   =  0,  //关闭状态
    NSOnState    =  1,  //开启状态
};
*/
@property NSInteger state;
//设置是否显示边框
@property (getter=isBordered) BOOL bordered;
//设置按钮是否透明
@property (getter=isTransparent) BOOL transparent;
//设置快捷键
@property (copy) NSString *keyEquivalent;
//设置富文本标题
@property (copy) NSAttributedString *attributedTitle;
@property (copy) NSAttributedString *attributedAlternateTitle;
//设置边框风格
@property NSBezelStyle bezelStyle;
//设置是否当鼠标移动到按钮上时显示边框
@property BOOL showsBorderOnlyWhileMouseInside;
//设置按钮声音
@property (nullable, strong) NSSound *sound;

```

下面是一些便捷创建按钮的方法：

```objectivec
//创建标准的按钮 包括标题和图片
+ (instancetype)buttonWithTitle:(NSString *)title image:(NSImage *)image target:(nullable id)target action:(nullable SEL)action;
//创建文字按钮
+ (instancetype)buttonWithTitle:(NSString *)title target:(nullable id)target action:(nullable SEL)action;
//创建图片按钮
+ (instancetype)buttonWithImage:(NSImage *)image target:(nullable id)target action:(nullable SEL)action;
//创建复选框按钮
+ (instancetype)checkboxWithTitle:(NSString *)title target:(nullable id)target action:(nullable SEL)action;
//创建单选框按钮
+ (instancetype)radioButtonWithTitle:(NSString *)title target:(nullable id)target action:(nullable SEL)action;
```

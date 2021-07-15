---
title: NSTextField控件应用详解
date: 2017-04-27
categories: macOS开发
tags: []
---
## NSTextField控件应用详解

    NSTextField用来接收用户文本输入，其可以接收键盘事件。创建NSTextFiled的示例代码如下：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    //创建TextField对象
    _textField = [[NSTextField alloc]initWithFrame:NSMakeRect(50, 30, 200, 50)];
    //设置默认显示的提示字符串
    _textField.placeholderString = @"请填写你的梦想";
    //设置默认显示的提示字符串 使用的带属性的字符串
    NSMutableAttributedString * attriString = [[NSMutableAttributedString alloc]initWithString:@"请填写你的梦想"];
    [attriString addAttribute:NSForegroundColorAttributeName value:[NSColor redColor] range:NSMakeRange(5, 2)];
    _textField.placeholderAttributedString = attriString;
    //设置文本框背景颜色
    _textField.backgroundColor = [NSColor greenColor];
    //设置是否绘制背景
    _textField.drawsBackground = YES;
    //设置文字颜色
    _textField.textColor = [NSColor blueColor];
    //设置是否显示边框
    _textField.bordered = YES;
    //设置是否绘制贝塞尔风格的边框
    _textField.bezeled = YES;
    //设置是否可以编辑
    _textField.editable = YES;
    //设置文本框是否可以选中
    _textField.selectable = YES;
    //设置贝塞尔风格
    _textField.bezelStyle = NSTextFieldSquareBezel;
    //设置倾向布局宽度
    _textField.preferredMaxLayoutWidth = 100;
    //设置最大行数
    _textField.maximumNumberOfLines = 5;
    //设置断行模式
    [[_textField cell] setLineBreakMode:NSLineBreakByCharWrapping];
    //设置是否启用单行模式
    [[_textField cell]setUsesSingleLineMode:NO];
    //设置超出行数是否隐藏
    [[_textField cell] setTruncatesLastVisibleLine: YES ];
    [self.view addSubview:_textField];
}
```

需要注意，在AppKit坐标体系中，原点在左下角，这和数学中的坐标系一致。运行工程，效果如下图所示：

![](https://static.oschina.net/uploads/space/2017/0427/173629_GXhP_2340880.png)

NSTextField类中常用的属性和方法列举如下：

```objectivec
//设置默认显示的提示文字
@property (nullable, copy) NSString *placeholderString NS_AVAILABLE_MAC(10_10);
//设置默认显示的提示文字 带属性的文本
@property (nullable, copy) NSAttributedString *placeholderAttributedString NS_AVAILABLE_MAC(10_10);
//设置背景颜色
@property (nullable, copy) NSColor *backgroundColor;
//设置是否绘制背景
@property BOOL drawsBackground;
//设置文字颜色
@property (nullable, copy) NSColor *textColor;
//设置是否绘制边框
@property (getter=isBordered) BOOL bordered;
//设置是否贝塞尔绘制
@property (getter=isBezeled) BOOL bezeled;
//设置是否允许编辑
@property (getter=isEditable) BOOL editable;
//设置是否允许文本框选中
@property (getter=isSelectable) BOOL selectable;
//设置代理
@property (nullable, assign) id<NSTextFieldDelegate> delegate;
//获取是否接受第一响应
@property (readonly) BOOL acceptsFirstResponder;
//设置贝塞尔风格
/*
typedef NS_ENUM(NSUInteger, NSTextFieldBezelStyle) {
    NSTextFieldSquareBezel  = 0,
    NSTextFieldRoundedBezel = 1
};
*/
@property NSTextFieldBezelStyle bezelStyle;
//设置一个预定的最大宽度
@property CGFloat preferredMaxLayoutWidth;
//设置最大行数
@property NSInteger maximumNumberOfLines;
//设置是否允许编辑文本属性
@property BOOL allowsEditingTextAttributes;
//设置是否允许用户向文本框中拖拽图片
@property BOOL importsGraphics;

//下面这些方法用于子类进行重写
//选择文本框时调用
- (void)selectText:(nullable id)sender;
//询问是否允许开始编辑文本框
- (BOOL)textShouldBeginEditing:(NSText *)textObject;
//询问是否允许结束编辑文本框
- (BOOL)textShouldEndEditing:(NSText *)textObject;
//文本框已经开始进入编辑的通知
- (void)textDidBeginEditing:(NSNotification *)notification;
//文本框已经结束编辑的通知
- (void)textDidEndEditing:(NSNotification *)notification;
//文本框中文字发生变化的通知
- (void)textDidChange:(NSNotification *)notification;

//下面两个属性与TouchBar相关 只有再较高版本的mac电脑中有效
//自动完成编辑
@property (getter=isAutomaticTextCompletionEnabled) BOOL automaticTextCompletionEnabled NS_AVAILABLE_MAC(10_12_2);
//字符选择按钮
@property BOOL allowsCharacterPickerTouchBarItem NS_AVAILABLE_MAC(10_12_2);

//下面是一些便捷创建NSTextField对象的方法
+ (instancetype)labelWithString:(NSString *)stringValue NS_SWIFT_NAME(init(labelWithString:)) NS_AVAILABLE_MAC(10_12);
+ (instancetype)wrappingLabelWithString:(NSString *)stringValue NS_SWIFT_NAME(init(wrappingLabelWithString:)) NS_AVAILABLE_MAC(10_12);
+ (instancetype)labelWithAttributedString:(NSAttributedString *)attributedStringValue NS_SWIFT_NAME(init(labelWithAttributedString:)) NS_AVAILABLE_MAC(10_12);
+ (instancetype)textFieldWithString:(nullable NSString *)stringValue NS_AVAILABLE_MAC(10_12);


```

NSTextField类继承自NSControl类，NSControl类中定义了许多属性可以获取到文本框中的文本，例如stringValue属性，本文中不再赘述。

    关于NSTextFieldDelegate协议，其实际上是继承自NSControlTextEditingDelegate协议，这个协议中定义了NSTextField控件在活动过程中的回调方法，例如开始编辑，结束编辑等。

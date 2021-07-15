---
title: OS X开发：NSTextField文本输入框
date: 2017-07-19
categories: macOS开发
tags: []
---
## OS X开发：NSTextField文本输入框

    NSTextField组件可以接收用户的输入，和UITextField不同，其可以将用户的输入进行多行显示。示例代码如下：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    //创建
    MyTextField * textField = [[MyTextField alloc]initWithFrame:NSMakeRect(100, 200, 300, 40)];
    //设置默认提示文字
    textField.placeholderString = @"默认文字";
    //设置背景
    textField.backgroundColor = [NSColor redColor];
    //设置是否渲染背景
    textField.drawsBackground = YES;
    //设置文字颜色
    textField.textColor = [NSColor blueColor];
    //设置是否边框
    textField.bordered = YES;
    //设置是否贝塞尔
    textField.bezeled = YES;
    //设置代理
    textField.delegate = self;
    [self.view addSubview:textField];
}
```

NSTextField类解析如下：

```objectivec
//设置默认提示文字
@property (nullable, copy) NSString *placeholderString;
//富文本提示文字
@property (nullable, copy) NSAttributedString *placeholderAttributedString;
//设置背景色
@property (nullable, copy) NSColor *backgroundColor;
//设置是否渲染背景色
@property BOOL drawsBackground;
//设置文字颜色
@property (nullable, copy) NSColor *textColor;
//设置是否有边框
@property (getter=isBordered) BOOL bordered;
//设置贝塞尔边框
@property (getter=isBezeled) BOOL bezeled;
//设置是否可编辑
@property (getter=isEditable) BOOL editable;
//设置是否可选择文字
@property (getter=isSelectable) BOOL selectable;
//选择文本
- (void)selectText:(nullable id)sender;
//代理
@property (nullable, assign) id<NSTextFieldDelegate> delegate;
//是否允许称为第一响应
@property (readonly) BOOL acceptsFirstResponder;
//设置贝塞尔风格
/*
typedef NS_ENUM(NSUInteger, NSTextFieldBezelStyle) {
    NSTextFieldSquareBezel  = 0,
    NSTextFieldRoundedBezel = 1
};
*/
@property NSTextFieldBezelStyle bezelStyle;
//子类可以重写如下方法：
//即将进入编辑状态时被调用 返回值决定是否允许编辑
- (BOOL)textShouldBeginEditing:(NSText *)textObject;
//即将结束编辑状态时调用 返回值决定是否允许结束编辑
- (BOOL)textShouldEndEditing:(NSText *)textObject;
//已经开始编辑时调用
- (void)textDidBeginEditing:(NSNotification *)notification;
//已经结束编辑时调用
- (void)textDidEndEditing:(NSNotification *)notification;
//文本改变时调用
- (void)textDidChange:(NSNotification *)notification;

//下面这些方法用来快捷创建NSTextField
+ (instancetype)labelWithString:(NSString *)stringValue;
+ (instancetype)wrappingLabelWithString:(NSString *)stringValue;
+ (instancetype)textFieldWithString:(nullable NSString *)stringValue;
+ (instancetype)labelWithAttributedString:(NSAttributedString *)attributedStringValue;
```

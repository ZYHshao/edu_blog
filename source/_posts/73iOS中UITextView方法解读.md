---
title: iOS中UITextView方法解读
date: 2015-06-04
categories: iOS之UI控件
tags: []
---
## iOS中UITextView方法解读

常用属性解读：

[@property](http://my.oschina.net/property)(nonatomic,assign) id<UITextViewDelegate\> delegate;

设置代理属性

[@property](http://my.oschina.net/property)(nonatomic,copy) NSString *text;

textView上的文本

[@property](http://my.oschina.net/property)(nonatomic,retain) UIFont *font;

设置文本字体

[@property](http://my.oschina.net/property)(nonatomic,retain) UIColor *textColor;

设置文本颜色

[@property](http://my.oschina.net/property)(nonatomic) NSTextAlignment textAlignment; 

设置文本对齐模式

@property(nonatomic) NSRange selectedRange;

设置选中的文本范围(只有当textView是第一响应时才有效)

@property(nonatomic,getter=isEditable) BOOL editable;

设置是否可以编辑

@property(nonatomic,getter=isSelectable) BOOL selectable;

设置是否可以选中

@property(nonatomic) UIDataDetectorTypes dataDetectorTypes;

这个属性可以将本文中的电话，邮件等变为链接，长按会调用响应响应的程序(textView必须为不可编辑状态)，属性的枚举如下：

```
typedef NS_OPTIONS(NSUInteger, UIDataDetectorTypes) {
    UIDataDetectorTypePhoneNumber   = 1 << 0,          // 电话变为链接
    UIDataDetectorTypeLink          = 1 << 1,          // 网址变为链接   
    UIDataDetectorTypeAddress       = 1 << 2,          // 地址变为链接
    UIDataDetectorTypeCalendarEvent = 1 << 3,          // 日历变为链接
    UIDataDetectorTypeNone          = 0,               // 无连接
    UIDataDetectorTypeAll           = NSUIntegerMax    // 所有类型链接
};
```

@property(nonatomic) BOOL allowsEditingTextAttributes;

设置是否允许编辑属性字符串文本

@property(nonatomic,copy) NSAttributedString *attributedText;

设置属性字符串文本

@property(nonatomic,copy) NSDictionary *typingAttributes;

设置属性字符串文本属性字典

\- (void)scrollRangeToVisible:(NSRange)range;

滚动textView使其显示在本一段文本

@property (readwrite, retain) UIView *inputView;

设置成为第一响应时弹出的视图，键盘视图

@property (readwrite, retain) UIView *inputAccessoryView;

设置成为第一响应时弹出的副视图，副键盘视图

@property(nonatomic) BOOL clearsOnInsertion;

设置是否显示删除按钮

UITextViewDelegate中的方法

\- (BOOL)textViewShouldBeginEditing:(UITextView *)textView;

是否开始编辑

\- (BOOL)textViewShouldEndEditing:(UITextView *)textView;

是否结束编辑

\- (void)textViewDidBeginEditing:(UITextView *)textView;

开始编辑时触发的方法

\- (void)textViewDidEndEditing:(UITextView *)textView;

结束编辑时触发的方法

\- (BOOL)textView:(UITextView *)textView shouldChangeTextInRange:(NSRange)range replacementText:(NSString *)text;

是否允许字符改变

\- (void)textViewDidChange:(UITextView *)textView;

字符内容改变触发的方法

\- (void)textViewDidChangeSelection:(UITextView *)textView;

选中内容改变触发的方法

\- (BOOL)textView:(UITextView *)textView shouldInteractWithURL:(NSURL *)URL inRange:(NSRange)characterRange;

当文本中的URL进行链接时触发的方法

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

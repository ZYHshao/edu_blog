---
title: iOS中使用NSAttributedString灵活创建标签
date: 2015-04-08  
categories: iOS之UI控件
tags: [NSAttributedString,iOS编程]              
---
灵活使用NSAttributedString可以更轻松的创建出内容复杂的标签。需要注意一点：如果一个label设置了这个属性，那它其他的设置都将失效。

首先，我们初始化一个NSMutableAttributedString对象。

```
//通过字符串初始化
//- (instancetype)initWithString:(NSString *)str;
//通过字符串和属性字典直接初始化
//- (instancetype)initWithString:(NSString *)str attributes:(NSDictionary *)attrs;
//通过自身对象初始化
//- (instancetype)initWithAttributedString:(NSAttributedString *)attrStr;

 NSMutableAttributedString * attribute = [[NSMutableAttributedString alloc]initWithString:@"123!@#你好么QWE"];
```

可以通过下面两个函数对attrebute字符串进行设置与修改

```
//可以替换字符
- (void)replaceCharactersInRange:(NSRange)range withString:(NSString *)str;
//属性设置
- (void)setAttributes:(NSDictionary *)attrs range:(NSRange)range;
//设置一定范围内字符属性
- (void)addAttribute:(NSString *)name value:(id)value range:(NSRange)range;
```

字典的键值对应如下：

```
//kCTFontAttributeName 这个键是字体的名称 必须传入CTFont对象
//kCTKernAttributeName 这个键设置字体间距 传入必须是数字对象 默认为0
//kCTLigatureAttributeName  这个键设置连字方式 必须传入CFNumber对象
//kCTParagraphStyleAttributeName  段落对其方式
//kCTForegroundColorAttributeName 字体颜色 必须传入CGColor对象
//kCTStrokeWidthAttributeName 笔画宽度 必须是CFNumber对象
//kCTStrokeColorAttributeName 笔画颜色
//kCTSuperscriptAttributeName 控制垂直文本定位 CFNumber对象
//kCTUnderlineColorAttributeName 下划线颜色
[attribute addAttribute:(NSString*)kCTKernAttributeName value:@5 range:NSMakeRange(0, 5)];
[attribute addAttribute:(NSString *)kCTFontAttributeName
                        value:(id)CFBridgingRelease(CTFontCreateWithName((CFStringRef)[UIFont boldSystemFontOfSize:14].fontName,
                                                       14,
                                                       NULL))
                        range:NSMakeRange(0, 4)];
    [attribute addAttribute:(NSString *)kCTUnderlineStyleAttributeName
                        value:(id)[NSNumber numberWithInt:kCTUnderlineStyleDouble]
                        range:NSMakeRange(0, 4)];
```

通过测试，发现上面有些键值并没有作用，可以替换下面的方法，效果相同，不同的地方在于其传值的类型不同，下面的方法更加方便（使用UIFont UIColor NSString 和一些系统枚举）

```
 NSParagraphStyleAttributeName
NSForegroundColorAttributeName
NSBackgroundColorAttributeName
NSLigatureAttributeName
NSKernAttributeName
NSStrikethroughStyleAttributeName
NSUnderlineStyleAttributeName
NSStrokeColorAttributeName
 NSStrokeWidthAttributeName
 NSShadowAttributeName
 NSTextEffectAttributeName
NSAttachmentAttributeName
 NSLinkAttributeName
 NSBaselineOffsetAttributeName
 NSUnderlineColorAttributeName
NSStrikethroughColorAttributeName
NSObliquenessAttributeName
 NSExpansionAttributeName
 NSWritingDirectionAttributeName
NSVerticalGlyphFormAttributeName
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

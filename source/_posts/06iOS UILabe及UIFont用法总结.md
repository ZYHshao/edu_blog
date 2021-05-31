---
title: iOS UILabe及UIFont用法总结
date: 2015-04-07  
categories: iOS之UI控件
tags: [UILabel,UIFont,iOS编程]              
---
初始化一个UILabel对象，并初始化大小

UILabel \* label = \[\[UILabel  alloc\]initWithFrame:CGRectMake(100, 100, 100, 100)\];

设置显示的文字

label.text=@"123";

和字体相关的一个类，字号大小默认17

[@property](http://my.oschina.net/property)(nonatomic,retain) UIFont*font; 

```
//7.0之后可用 设置字体风格
//    NSString *const UIFontTextStyleHeadline; 用于标题的风格
//    NSString *const UIFontTextStyleSubheadline;用于副标题的风格
//    NSString *const UIFontTextStyleBody;用于正文的字体
//    NSString *const UIFontTextStyleFootnote;用于脚注的字体
//    NSString *const UIFontTextStyleCaption1;用于标准字幕字体
//    NSString *const UIFontTextStyleCaption2;用于替换字幕字体
    label.font=[UIFont preferredFontForTextStyle:UIFontTextStyleCaption2];
//说实话，没看出什么太大的差别

//设置字体和字体大小
+ (UIFont *)fontWithName:(NSString *)fontName size:(CGFloat)fontSize;
//返回所有字体的字体家族名称数组
+ (NSArray *)familyNames;
//按字体家族名称返回字体名称数组
+ (NSArray *)fontNamesForFamilyName:(NSString *)familyName;
//设置普通字体字号大小
+ (UIFont *)systemFontOfSize:(CGFloat)fontSize;
//设置加粗字体字号大小
+ (UIFont *)boldSystemFontOfSize:(CGFloat)fontSize;
//设置斜体字号大小
+ (UIFont *)italicSystemFontOfSize:(CGFloat)fontSize;

//一些只读属性
//字体家族名称
@property(nonatomic,readonly,retain) NSString *familyName;
//字体名称
@property(nonatomic,readonly,retain) NSString *fontName;
//字号大小
@property(nonatomic,readonly)        CGFloat   pointSize;
//字体设计模型，表示距离最高点偏移余量
@property(nonatomic,readonly)        CGFloat   ascender;
//底部的模型偏移量
@property(nonatomic,readonly)        CGFloat   descender;
//字体模型的头高信息
@property(nonatomic,readonly)        CGFloat   capHeight;
//字体模型的xHeight信息
@property(nonatomic,readonly)        CGFloat   xHeight;
//字体行高
@property(nonatomic,readonly)        CGFloat   lineHeight NS_AVAILABLE_IOS(4_0);
//模型主体信息
@property(nonatomic,readonly)        CGFloat   leading;
//创建一个新字体与当前字体相同，除了指定的大小
- (UIFont *)fontWithSize:(CGFloat)fontSize;
//通过描述信息返回字体 7.0后可用
+ (UIFont *)fontWithDescriptor:(UIFontDescriptor *)descriptor size:(CGFloat)pointSize NS_AVAILABLE_IOS(7_0);
//返回字体的描述信息，7.0后可用
- (UIFontDescriptor *)fontDescriptor NS_AVAILABLE_IOS(7_0);
```

设置字体颜色

label.textColor=\[UIColor redColor\];

设置阴影偏移量

label.shadowOffset=CGSizeMake(20, 20);

设置阴影颜色

label.shadowColor=\[UIColor  blackColor\];

设置对齐模式

label.textAlignment=NSTextAlignmentJustified;

```
enum {
   //沿左边沿对齐文本
   NSTextAlignmentLeft      = 0,
   //中心对齐
   NSTextAlignmentCenter    = 1,
   //右边沿对齐
   NSTextAlignmentRight     = 2,
   //最后一行自然对齐
   NSTextAlignmentJustified = 3,
   //默认对齐
   NSTextAlignmentNatural   = 4,};typedef NSInteger NSTextAlignment;
```

多行文本设置

label.lineBreakMode=NSLineBreakByCharWrapping;

```
enum {
   //文本边缘处理
   NSLineBreakByWordWrapping = 0,
   //提前处理不合适的字符
   NSLineBreakByCharWrapping,
   //简单线性处理
   NSLineBreakByClipping,
   //丢失的开头用省略号表示
   NSLineBreakByTruncatingHead,
   //丢失的文本在末尾显示省略号
   NSLineBreakByTruncatingTail,
   //丢失的文本在中间显示省略号
   NSLineBreakByTruncatingMiddle };typedef NSUInteger NSLineBreakMode
```

使用attributedText绘制

[@property](http://my.oschina.net/property)(nonatomic,copy)   NSAttributedString *attributedText 

设置高亮的字体颜色

label.highlightedTextColor=\[UIColor  blueColor\];

//设置是否高亮

label.highlighted=YES;

用户交互 默认关闭

label.userInteractionEnabled=NO;

是否有效，默认是YES，无效为灰色

label.enabled=NO;

显示的行数，0为无限

[@property](http://my.oschina.net/property)(nonatomic) NSInteger numberOfLines;

宽度自适应大小 默认是NO

[@property](http://my.oschina.net/property)(nonatomic) BOOL adjustsFontSizeToFitWidth;

字符适应宽度：不赞成使用

@property(nonatomic) BOOL adjustsLetterSpacingToFitWidth

最小适应大小2.0-6.0

@property(nonatomic) CGFloat minimumFontSize

最小适应大小 6.0 之后

@property(nonatomic) CGFloat minimumScaleFactor

垂直方向的调整

@property(nonatomic) UIBaselineAdjustment baselineAdjustment;

```
typedef enum {
   //调整文本对应基线位置
   UIBaselineAdjustmentAlignBaselines,
   //调整文本相对其边框的中心
   UIBaselineAdjustmentAlignCenters,
   //调整文本相对于边界的左上角 默认的
   UIBaselineAdjustmentNone,} UIBaselineAdjustment;
```

返回文本绘制矩形

\- (CGRect)textRectForBounds:(CGRect)bounds limitedToNumberOfLines:(NSInteger)numberOfLines;

文本绘制函数

\- (void)drawTextInRect:(CGRect)rect

文本自动布局参数

@property(nonatomic) CGFloat preferredMaxLayoutWidth 

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

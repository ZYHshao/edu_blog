---
title: iOS中支持HTML文本的标签控件——MDHTMLLabel
date: 2016-06-30
categories: iOS第三方库
tags: []
---
## iOS中支持HTML文本的标签控件——MDHTMLLabel

### 一、引言

        在iOS开发中对HTML的处理很多时候除了使用WebView外，还需要原生的控件对其进行渲染，例如将HTML字符串渲染为图文混排的View视图。Git上有很多轻量级的HTML渲染框架，列举一些如下：

RTLabel：基于UIView的HTML文本渲染控件，git地址：[https://github.com/honcheng/RTLabel](https://github.com/honcheng/RTLabel)。

RCLabel：与RTLabel思路相同，基于RCLabel之上，也是UIView的子类，支持了对HTML中的本地图片标签进行渲染。git地址：[https://github.com/Janak-Nirmal/RichContentLabel](https://github.com/Janak-Nirmal/RichContentLabel)。

MDHTMLLabel：与RTLabel和RCLabel不同的是，其是UILabel的子类，更加轻量级，不能支持图片标签。git地址：[https://github.com/mattdonnelly/MDHTMLLabel](https://github.com/mattdonnelly/MDHTMLLabel)。

    关于RCLabel对图片便签的支持，其只能支持本地的图片，不能支持远程URL图片链接，这在开发中将十分局限，以前我曾加RCLabel做了改造，加了支持远程图片URL的方法，我把它集成在了一个基础框架中，需要的伙伴可以参考下，git地址：[https://github.com/ZYHshao/YHBaseFoundationTest](https://github.com/ZYHshao/YHBaseFoundationTest)。配套的讲解博客地址如下：[http://my.oschina.net/u/2340880/blog/499311](http://my.oschina.net/u/2340880/blog/499311)。

    本篇博客主要讨论MDHTMLLabel的使用。

### 二、MDHTMLLabel的创建与设置

      MDHTMLLabel框架十分小巧，其中只有两个文件，总计2000余行代码。通过HTML字符串来创建一个MDHTMLLabel控件示例代码如下：

```objectivec
    NSString * kDemoText = @"<a href='http://github.com/mattdonnelly/MDHTMLLabel'>MDHTMLLabel</a> is a lightweight, easy to use replacement for <b>UILabel</b> which allows you to fully <font face='Didot-Italic' size='19'>customize</font> the appearence of the text using HTML (with a few added features thanks to <b>CoreText</b>), as well letting you handle whenever a user taps or holds down on link and automatically detects ones not wrapped in anchor tags/>";
    MDHTMLLabel *htmlLabel = [[MDHTMLLabel alloc] initWithFrame:self.view.frame];
    htmlLabel.numberOfLines = 0;
    htmlLabel.htmlText = kDemoText;
    [self.view addSubview:htmlLabel];
```

效果如下图所示：

![](http://static.oschina.net/uploads/space/2016/0630/105340_GU71_2340880.png)

MDHTMLLabel中可以设置的一些属性解析如下：

```objectivec
//设置超链接文字的属性字典 和设置AttributeString方法一致
@property (nonatomic, strong) NSDictionary *linkAttributes;
//设置超链接文字激活时的属性字典
@property (nonatomic, strong) NSDictionary *activeLinkAttributes;
//设置超链接非激活时的属性字典
@property (nonatomic, strong) NSDictionary *inactiveLinkAttributes;
//设置超链接文字触发长按事件的最小按下时间
@property (nonatomic, assign) NSTimeInterval minimumPressDuration;
//设置label文件阴影的模糊半径
@property (nonatomic, assign) CGFloat shadowRadius;
//设置label在高亮状态下的文字模糊半径 注：非高亮状态的由原生UILabel的属性设置
@property (nonatomic, assign) CGFloat highlightedShadowRadius;
//设置label在高亮状态下的文字阴影偏移 注：非高亮状态的由原生UILabel的属性设置
@property (nonatomic, assign) CGSize highlightedShadowOffset;
//设置在label高亮状态下的文字阴影颜色 注：非高亮状态的由原生UILabel的属性设置
@property (nonatomic, strong) UIColor *highlightedShadowColor;
//设置首行文字的缩进距离
@property (nonatomic, assign) CGFloat firstLineIndent;
//设置文字的行间距
@property (nonatomic, assign) CGFloat leading;
//设置行高的倍数
@property (nonatomic, assign) CGFloat lineHeightMultiple;
//设置文字内容的边距
@property (nonatomic, assign) UIEdgeInsets textInsets;
//设置文字垂直方向的对其模式 默认为居中对其 MDHTMLLabelVerticalAlignment枚举意义如下:
/*
typedef NS_ENUM(NSUInteger, MDHTMLLabelVerticalAlignment) {
    MDHTMLLabelVerticalAlignmentCenter   = 0, //居中对其
    MDHTMLLabelVerticalAlignmentTop      = 1, //顶部对其
    MDHTMLLabelVerticalAlignmentBottom   = 2, //底部对其
};
*/
@property (nonatomic, assign) MDHTMLLabelVerticalAlignment verticalAlignment;
//设置文字的截断模式
@property (nonatomic, strong) NSString *truncationTokenString;
//根据内容获取控件尺寸
+ (CGFloat)sizeThatFitsHTMLString:(NSString *)htmlString
                         withFont:(UIFont *)font
                      constraints:(CGSize)size
           limitedToNumberOfLines:(NSUInteger)numberOfLines;

```

关于HTML数据中的超链接的相应，MDHTMLLabel是通过代理回调的方式处理的，如下：

```objectivec
@protocol MDHTMLLabelDelegate <NSObject>
@optional
//点击超链接的时候触发的方法
- (void)HTMLLabel:(MDHTMLLabel *)label didSelectLinkWithURL:(NSURL*)URL;
//长按超链接时触发的方法
- (void)HTMLLabel:(MDHTMLLabel *)label didHoldLinkWithURL:(NSURL*)URL;
@end

```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

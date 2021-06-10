---
title: iOS中图片(UIImage)拉伸技巧
date: 2015-04-20
categories: iOS逻辑初窥
tags: [UIImage,iOS编程]              
---
## iOS中图片拉伸技巧与方法总结

### 一、了解几个图像拉伸的函数和方法

#### 1、直接拉伸法

简单暴力，却是最最常用的方法，直接将图片设置为ImageView的image属性，图片便会随UIImageView对象的大小做自动拉伸。这种拉伸的方法有一个致命的缺陷，它会使图像发生失真与形变。

#### 2、像素点的拉伸

\- (UIImage *)stretchableImageWithLeftCapWidth:(NSInteger)leftCapWidth topCapHeight:(NSInteger)topCapHeight;

这个函数我们可以用来拉伸类似QQ，微信的聊天气泡背景图，它的两个参数分别leftCapWidth和topCapHeight，这两个参数给定一个坐标，比如：

```
    UIImage * img= [UIImage imageNamed:@"11.png"];
    img = [img stretchableImageWithLeftCapWidth:1 topCapHeight:1];
```

这段代码的意思是将图片从左起第2列，上起第2行，坐标为(2,2)的像素点进行复制。将图片进行拉伸。这个方法和上面的方法比起来似乎灵活性更多了，但其也有它的一些局限，如果被拉伸的图片中间也有需要拉伸的像素，这个方法就无能为力了，例如，如下的一张图片，我们需要将其拉伸放大：

![](http://static.oschina.net/uploads/space/2015/0420/170142_h00s_2340880.png)

便会出现这样的效果：

![](http://static.oschina.net/uploads/space/2015/0420/170229_0mm9_2340880.png)

这明显和我们的意图是不符的，那么，我们可以使用下面的方法。

#### 3、区域的拉伸

\- (UIImage *)resizableImageWithCapInsets:(UIEdgeInsets)capInsets;

这个函数需要设置一个UIEdgeInsets参数，UIEdgeInsets结构体如下：

```
typedef struct UIEdgeInsets {
    CGFloat top, left, bottom, right; 
} UIEdgeInsets;
```

它分别对用了图片进行拉伸的区域距离顶部、左部、下部、右部的像素。比如，一个10\*10像素的图片，将UIEdgeInsets参数全部设置为1，则实际拉伸的部分就是中间的8\*8的区域的像素。有一点需要注意，这个方法默认使用的拉伸模式是区域复制，比如还是上面的图案，如下代码拉伸：

```
    UIImage * img= [UIImage imageNamed:@"11.png"];
    img = [img resizableImageWithCapInsets:UIEdgeInsetsMake(1, 1, 1, 1)];
```

结果如下：

![](http://static.oschina.net/uploads/space/2015/0420/171808_7ol6_2340880.png)

可以明显的看到中间的虚线，这便是区域复制的杰作。

那么问题又来了，如果某些图片中间有渐变，我们该怎么处理了，来看下一个函数。

#### 4、拉伸模式的设置

\- (UIImage *)resizableImageWithCapInsets:(UIEdgeInsets)capInsets resizingMode:(UIImageResizingMode)resizingMode;

这个函数和上一个函数相比，唯一的差别是多了一个参数。这个参数是个枚举，如下：

```
typedef NS_ENUM(NSInteger, UIImageResizingMode) {
    UIImageResizingModeTile,//进行区域复制模式拉伸
    UIImageResizingModeStretch,//进行渐变复制模式拉伸
};
```

现在就明了了，我们只需要设置一下模式，就可以实现渐变拉伸了：

```
    UIImage * img= [UIImage imageNamed:@"11.png"];
    img = [img resizableImageWithCapInsets:UIEdgeInsetsMake(1, 1, 1, 1) resizingMode:UIImageResizingModeStretch];
```

来看一下效果：

![](http://static.oschina.net/uploads/space/2015/0420/172658_DLXU_2340880.png)

### 二、拉伸的用武之地

圆角按钮，空心按钮，渐变的背景，内容可变的标签，聊天气泡等等这样的素材在APP中很可能会多次出现，并且每次出现的尺寸可能还会略微有些差异，如果仅仅依靠美工的素材，恐怕不仅很难达到要求，也会额外增加软件的内存开销，这时，我们使用恰当的拉伸技巧，能使我们的代码更加健壮，APP更加高效。

### 三、一点小经验

你是否注意观察过最细的线？

看到上面的问句，你可能有些差异。最细的线不就是一像素么？确实，能绘图画出来的最细的实心线确实是一像素，但在一个项目中，我们优秀的美工察觉到无论她把线做的多么细，无论我怎样控制拉伸方法，绘制出的登录框总是没有QQ的细，QQ的框线看起来更加干脆利索。后来索性用绘图画出登录框，结果很不幸，我依然无法将线做到像QQ登录框那样细致。后来偶然试了一种方法，不知原理是否正确，效果总算达到了，当然这也要归功于我们的美工，她将一个图片做的很大，适配最大的分辨率，然后让我手动缩，如此一来，那线就变得非常细。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592
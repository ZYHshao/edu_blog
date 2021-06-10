---
title: iOS使用Xcode可视化图像编辑功能进行图片拉伸
date: 2015-04-21
categories: iOS逻辑初窥
tags: [iOS编程]              
---
## iOS中可视化拉伸图片技巧

### 一、补充

在我的另一篇博客[http://my.oschina.net/u/2340880/blog/403996](http://my.oschina.net/u/2340880/blog/403996)中探讨了IOS拉伸图像(UIImage)的几种方法和一些小经验，这篇是一个补充，再将xcode中的另一种可视化拉伸图像的方法的使用介绍给大家。

### 二、如何使用

IOS开发文档中的描述：[https://developer.apple.com/library/ios/recipes/xcode\_help-image\_catalog-1.0/chapters/SlicinganImage.html](https://developer.apple.com/library/ios/recipes/xcode_help-image_catalog-1.0/chapters/SlicinganImage.html)

#### 1、xcode5的新特性

xcode5之后，IOS为我们提供了一个管理图片的新方法Asset Catalogs，简单说来，它相当于一个目录，专门用来管理我们项目中的图片素材，包括Icon和启动页，这样使项目管理更加方便也更加简洁。

创建一个AssetCatalogs：在xcode中新建一个文件，选择AssetCatalogs，如下：

![](http://static.oschina.net/uploads/space/2015/0421/114900_yHuK_2340880.png)

然后我们点开这个包，将图片直接拖入工具区即可：

![](http://static.oschina.net/uploads/space/2015/0421/115045_86qu_2340880.png)

#### 2、使用AssetCatalogs中的可视化工具进行图片拉伸

完成了上面的步骤之后，我们可以对管理的图片进行处理，点击右下角的show Slicing按钮，我们就会进入可视化编辑区，如下：

![](http://static.oschina.net/uploads/space/2015/0421/120521_aPQz_2340880.png)

如上图，有三条竖直线，其中边界的两条分别约束了图片两侧不被拉伸的区域范围，中间虚线和左侧虚线围成的部分，将是被复制拉伸的区域。水平方向的线同理。

很重要的一点：官方文档告诉我们，这个方法只能在iOS 7 或者 OS X v10.10之后使用。效果如下：

![](http://static.oschina.net/uploads/space/2015/0421/124916_HYQE_2340880.png)

#### 3、在xib文件中UIImage的拉伸

在xib文件中的UIImageView，在上面加上图片后，可以设置stretching这个属性：

![](http://static.oschina.net/uploads/space/2015/0421/125500_DBmH_2340880.png)

这个属性的四个值：X,Y,Width,Height的取值范围是0-1；X，Y，用来确定一个点，比如我们设置为X=0.1，Y=0.1，则这个点就是图片的左上角开始，水平1/10处和竖直1/10处，设置图片的拉伸点为从这个点开始。后两个参数分别设置图片拉伸区域的宽度和高度，比如我们这样设置：Width=0.8，Height=0.8，则图片拉伸时上下左右各1/10的宽度不会被拉伸，中间部分被拉伸，还是刚才的图片，效果如下：

![](http://static.oschina.net/uploads/space/2015/0421/130057_NBcG_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592
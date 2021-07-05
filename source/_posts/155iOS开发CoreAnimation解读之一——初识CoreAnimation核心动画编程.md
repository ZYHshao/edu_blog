---
title: iOS开发CoreAnimation解读之一——初识CoreAnimation核心动画编程
date: 2015-11-25
categories: iOS逻辑初窥
tags: []
---
## iOS开发CoreAnimation解读之一——初识CoreAnimation核心动画编程

### 一、引言

        众所周知，绚丽动画效果是iOS系统的一大特点，通过UIView层封装的动画，基本已经可以满足我们应用开发的所有需求，但若需要更加自由的控制动画的展示，我们就需要使用CoreAnimation框架中的一些类与方法。这里先附上前几篇与动画相关的博客地址，这一系列，我们抽出其中的CoreAnimation框架来详细解读。

UIViewAnimation动画的使用：[http://my.oschina.net/u/2340880/blog/484457 ](http://my.oschina.net/u/2340880/blog/484457)

UIView动画执行的另一种方式：[http://my.oschina.net/u/2340880/blog/484538](http://my.oschina.net/u/2340880/blog/484538)

UIView转场动画：[http://my.oschina.net/u/2340880/blog/484669](http://my.oschina.net/u/2340880/blog/484669)

CoreAnimation隐式动画的应用：[http://my.oschina.net/u/2340880/blog/484793](http://my.oschina.net/u/2340880/blog/484793)

粒子效果的使用：[http://my.oschina.net/u/2340880/blog/485095](http://my.oschina.net/u/2340880/blog/485095)

### 二、初识CoreAnimation

        CoreAnimation框架是基于OpenGL与CoreGraphics图像处理框架的一个跨平台的动画框架。简单来说，它使帮助我们将图像读取成位图，通过硬件的处理，实现动画效果。文档中的一张图片十分形象的描述了CoreAnimation与UIKit框架的关系：

![](http://static.oschina.net/uploads/space/2015/1124/131007_eqtz_2340880.png)

在CoreAnimation中，大部分的动画效果都是通过Layer层来实现的，通过CALayer，我们可以组织复杂的层级结构。

        在CoreAnimation中，大多数的动画效果是添加在图层属性的变化上，例如，改变图层的位置，大小，颜色，圆角半径等。Layer层并不决定视图的展现，它只是存储了视图的几何属性状态。

### 三、锚点对几何属性的影响

        关于Layer层，我们需要了解一个有关锚点的概念，锚点决定了图层的绘制位置以及动画展示时其参照的点，锚点的取值范围为0-1，锚点有两个地方在应用中会有很大影响：

1.layer层的position参照点始终和锚点重合

通过position决定了layer所在的位置，在Layer中，虽然也有frame这样的属性，但我们很少使用，一般我们会使用bounds和position确定Layer层的大小和位置。

2.锚点决定进行动作的参照点

例如一个旋转动作，锚点决定了层旋转的中心点，对于放大缩小的动作，锚点决定了放大或者缩小参照的中心点。

可以来看下边一组图：

![](http://static.oschina.net/uploads/space/2015/1125/094457_ON2t_2340880.png)

![](http://static.oschina.net/uploads/space/2015/1125/094457_Jafb_2340880.png)

![](http://static.oschina.net/uploads/space/2015/1125/094458_kcyV_2340880.png)

上面两个矩形，frame和bounds都是一样的，第一个矩形的锚点位置为(0.5,0.5)，第二个为(0,0),

因此，两个矩形的position点是不同的，第一个是(100,100),第二个是(40,60)。再看当产生动作时锚点的影响：

![](http://static.oschina.net/uploads/space/2015/1125/094923_rhb1_2340880.png)

![](http://static.oschina.net/uploads/space/2015/1125/094924_WAWk_2340880.png)

  
现在就很好理解了，锚点的不同直接影响了动作产生的参照点。

通过CALayer的如下属性，我们可以设置锚点，注意x，y的取值范围都是0~1，代表所占宽度和高度的比例：

```
@property CGPoint anchorPoint;
```

### 四、Layer与View之间的关系

        Layer是专门用于辅助我们绘制图像的层，它使支持三维坐标系的绘制的，通过每个坐标点与转换矩阵的运算，来决定最后绘制的状态，并且，Layer可以更高帧率的绘制动画效果。然而Layer与View依然有很大不同，首先，我们不可能只通过Layer来开发应用程序，Layer并没有接收事件和处理用户交互的能力，这些依然需要View来完成，每一个View中，都有一个Layer的属性来辅助进行图形的绘制。并且Layer是可以层级嵌套的，开发中，我们可以根据需求灵活选择。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

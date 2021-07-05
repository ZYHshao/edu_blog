---
title: iOS动画开发之四——核心动画编程(CoreAnimation)
date: 2015-07-28
categories: iOS逻辑初窥
tags: []
---
## iOS动画开发之四——核心动画编程(CoreAnimation)

### 一、引言

        前几篇博客详细介绍了有关UIView层的动画使用与相关的效果，然而这些动画是UIKit为我们封装好的核心动画层的方法，通过这些方法，我们可以用的更加简便，当然功能也十分强大，基本能达到我们项目的大多需求。但是如果你想更加自由的通过动画操作视图的属性，你就需要跳过UIKit的封装，使用CoreAnimation核心动画层的方法来实现动画。

### 二、开始前的准备

#### 1、认识一个的朋友

        在开始介绍核心动画的内容前，我们需要先搞明白一个东西：Layer。你可能很少听说他，可是他却无处不在，在iOS的UI开发中，任何一个View包括继承于UIView的子类上面都会有一个Layer，可以理解为Layer为单独的一层，专门负责视图的显示，而view除此之外更多负责触摸时间等逻辑处理。因此，iOS也将所有动画的操作都交给你Layer来负责。

#### 2、Layer层可以做到的事

        Layer如此神秘，那他究竟可以做到哪些事？他确实可以做很多view做不了的事情.

##### (1)设置view的圆角属性

```
 view = [[UIView alloc]initWithFrame:CGRectMake(100, 100, 100, 100)];
    [self.view addSubview:view];
    view.backgroundColor=[UIColor redColor];
    view.layer.masksToBounds=YES;//设置layer层的切割属性
    view.layer.cornerRadius=10;//设置layer层的圆角半径
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/0728/165350_D5Xf_2340880.png)

##### (2)设置view的边框

```
view = [[UIView alloc]initWithFrame:CGRectMake(100, 100, 100, 100)];
    [self.view addSubview:view];
    view.backgroundColor=[UIColor redColor];
    CALayer *layer=view.layer;
    layer.borderWidth=10;//设置边框的宽度
    layer.borderColor=[[UIColor magentaColor]CGColor];//设置边框的颜色
```

注意：因为CoreAnimation层是UI层的底层，所以这里的颜色为CGColor对象。

效果如下：

##### ![](http://static.oschina.net/uploads/space/2015/0728/165843_qR0U_2340880.png)

##### (3)设置视图阴影

```
 view = [[UIView alloc]initWithFrame:CGRectMake(100, 100, 100, 100)];
    [self.view addSubview:view];
    view.backgroundColor=[UIColor redColor];
    CALayer *layer=view.layer;
    layer.shadowOffset=CGSizeMake(30, 30);//设置阴影方向
    layer.shadowColor=[[UIColor blackColor] CGColor];//设置阴影颜色
    layer.shadowOpacity=0.5;//设置阴影透明度
    layer.shadowRadius=10;//设置阴影圆角
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/0728/170301_jCTn_2340880.png)

这样的立体效果是否已经很酷了？NO，在加上动画才对。

### 三、CoreAnimation的使用

#### 1、基础属性相关的动画CABasicAnimation

CABasicAnimation是核心动画中对属性操作需要用到了一个动画类，示例如下：

```
    CALayer *layer=view.layer;
    CABasicAnimation * ani= [CABasicAnimation animationWithKeyPath:@"opacity"];//创建对象，参数关键字为layer的属性
    ani.duration=3;//设置执行时间
    ani.repeatCount=1;//设置执行次数
    ani.timingFunction=[CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];//设置线性效果
    [layer addAnimation:ani forKey:@"test"];//添加动画
    layer.opacity=0;//透明度改变时执行动画动作
```

通过上面的示例，我们可以发现，layer的属性都可以来进行动画动作，这样，我们对动画的操作就自由的很多。

#### 2、关键帧动画CAKeyframeAnimation

关键帧动画除了动画改变layer的属性外，可以设置几个关键帧点，通过这些点，可以实现路径更加负责的动画，例如：

```
CALayer *layer=view.layer;
    CAKeyframeAnimation * ani = [CAKeyframeAnimation animationWithKeyPath:@"opacity"];//创建一个关键帧动画对象
    ani.duration=3;
    ani.values=@[@1,@0,@1];//传入三个关键帧，动画会将试图先慢慢隐藏，再慢慢展现
    [layer addAnimation:ani forKey:@"test"];
```

类比如上代码，我们还可以通过关键帧让试图按照我们预定的路线移动，同时我们还可以设置两个数组，分别为keyTimes和timingFunctions。这两个数组中的值可以设置动画每一段的运动线性特征和每一段的运动时间比例。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

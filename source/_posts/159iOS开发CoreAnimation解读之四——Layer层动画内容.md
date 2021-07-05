---
title: iOS开发CoreAnimation解读之四——Layer层动画内容
date: 2015-12-05
categories: iOS逻辑初窥
tags: []
---
## iOS开发CoreAnimation解读之四——Layer层动画内容

### 一、引言

        通过前几篇博客的介绍，我们可以了解到layer层可以设置许多与控件UI相关的属性，并且对于iOS开发，UIView层的属性是会映射到CALayer的，因此，可以通过UIKit和CoreAnimation两个框架来设置控件的UI相关属性，当属性发生变化时，我们可以使其展示一个动画效果。

### 二、CAAnimation动画体系的介绍

        CAAnimation是CoreAnimation框架中执行动画对象的基类，下面有一张图，是我手画的，不太美观，但是可以将与CAAnimation相关的几个动画类的关系表达清楚：

![](http://static.oschina.net/uploads/space/2015/1204/172312_5Zup_2340880.png)

从上图中可以看到，从CAAnimation中继承出三个子类，分别是用于创建属性动画的CAPropertyAnimation，创建转场动画的CATransition和创建组合动画的CAAnimationGroup。

我们就先从根类开始探讨。

#### 1.CAAnimation属性和方法

CAAnimation作为动画对象的基类，其中封装了动画的基础属性，如下：

```
//通过类方法创建一个CAAnimation对象
+ (instancetype)animation;
//动画执行的时序模式
@property(nullable, strong) CAMediaTimingFunction *timingFunction;
//代理
@property(nullable, strong) id delegate;
//是否动画完成时将动画对象移除掉
@property(getter=isRemovedOnCompletion) BOOL removedOnCompletion;
```

timingFunction定义了动画执行的时序效果，CAMediaTimingFunction的创建方式如下：

```
/*
name参数决定的执行的效果，可选参数如下
//线性执行
 NSString * const kCAMediaTimingFunctionLinear;
 //淡入  在动画开始时 淡入效果
 NSString * const kCAMediaTimingFunctionEaseIn;
 //淡出 在动画结束时 淡出效果
 NSString * const kCAMediaTimingFunctionEaseOut;
 //淡入淡出
 NSString * const kCAMediaTimingFunctionEaseInEaseOut;
 //默认效果
 NSString * const kCAMediaTimingFunctionDefault;
*/
+ (instancetype)functionWithName:(NSString *)name;
```

CAAnimation的代理方法入如下几个：

```
//动画开始时执行的回调
- (void)animationDidStart:(CAAnimation *)anim;
//动画结束后执行的回调
- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag;
```

#### 2.CAPropertyAnimation属性与方法

        CAPropertyAnimation是继承于CAAnimation专门用来创建与属性相关的动画的类：

```
//创建对象 参数中的path就是我们要执行动画的属性
//例如，如果传入@"backgroundColor" 当layer的背景颜色改变时，就会执行我们设置的动画
+ (instancetype)animationWithKeyPath:(nullable NSString *)path;
//这个属性确定动画执行的状态是否叠加在控件的原状态上
//默认设置为NO，如果我们执行两次位置移动的动画，会从同一位置执行两次
//如果设置为YES，则会在第一次执行的基础上执行第二次动画
@property(getter=isAdditive) BOOL additive;
//这个属性对重复执行的动画有效果
//默认为NO，重复执行的动画每次都是从起始状态开始
//如果设置为yes，则为此执行都会在上一次执行的基础上执行
@property(getter=isCumulative) BOOL cumulative;
//这个属性和transfron属性的动画执行相关
@property(nullable, strong) CAValueFunction *valueFunction;
```

上面这些属性中，只有一个需要我们注意，valueFunction是专门为了transform动画而设置的，因为我们没有办法直接改变transform3D中的属性，通过这个参数，可以帮助我们直接操作transfrom3D属性变化产生动画效果，举例如下，一个绕Z轴旋转的动画：

```
 //绕z轴旋转的动画
    CABasicAnimation * ani = [CABasicAnimation animationWithKeyPath:@"transform"];
    //从0度开始
    ani.fromValue = @0;
    //旋转到180度
    ani.toValue = [NSNumber numberWithFloat:M_PI];
    //时间2S
    ani.duration = 2;
    //设置为z轴旋转
    ani.valueFunction = [CAValueFunction functionWithName:kCAValueFunctionRotateZ];
    //执行动画
    [layer addAnimation:ani forKey:@""];
```

实际上，使用点的方式也是可以访问到相应属性的，如果不设置valueFunction，使用如下方法也是可以进行绕Z轴旋转的：

```
//绕z轴旋转的动画
    CABasicAnimation * ani = [CABasicAnimation animationWithKeyPath:@"transform.rotation.z"];
    //从0度开始
    ani.fromValue = @0;
    //旋转到180度
    ani.toValue = [NSNumber numberWithFloat:M_PI];
    //时间2S
    ani.duration = 2;
    //执行动画
    [layer addAnimation:ani forKey:@""];
```

#### 3.CABasicAnimation属性

        CABasicAnimaton是CAPropertyAnimation分出来的一个子类，创建基础的属性变化动画，例如我们上面的示例代码，其中属性如下：

```
@property(nullable, strong) id fromValue;
@property(nullable, strong) id toValue;
@property(nullable, strong) id byValue;
```

上面三个属性都是来确定动画的起始与结束位置，有如下的含义：

fromValue和toValue不为空：动画的值由fromValue变化到toValue

fromValue和byValue不为空：动画的值由fromValue变化到fromValue+byValue

byValue和toValue不为空：动画的值由toValue-byValue变化到toValue

只有fromValue不为空：动画的值由fromValue变化到layer的当前状态值

只有toValue不为空：动画的值由layer当前的值变化到toValue

只有byValue不为空：动画的值由layer当前的值变化到layer当前的值+byValue

#### 4.CAKeyframeAnimation关键帧动画

        CAKeyframeAnimation也是继承与CAPropertyAnimation的一个子类，其与CABasicAnimation的不同之处在于虽然其都是改变layer层属性的动画，但是CABasicAnimation只能设置初始与结束状态，这之间我们没办法控制，而CAKeyframeAnimation可以让我们设置一些关键帧再整个动画的过程中。属性方法如下：

```
//关键帧的值数组 例如我们想让控件沿某个路径移动，这里面存放每个移动的点
@property(nullable, copy) NSArray *values;
//直接设置路径，作用域values类似
@property(nullable) CGPathRef path;
//设置每一帧执行的时间长短 这个的取值为0-1，代表占用时间的比例
@property(nullable, copy) NSArray<NSNumber *> *keyTimes;
//每一帧执行过程中的时序效果 上面有提过
@property(nullable, copy) NSArray<CAMediaTimingFunction *> *timingFunctions;
/*
设置帧的中间值如何计算
 NSString * const kCAAnimationLinear;
 NSString * const kCAAnimationDiscrete;
 NSString * const kCAAnimationPaced;
 NSString * const kCAAnimationCubic;
 NSString * const kCAAnimationCubicPaced;
*/
@property(copy) NSString *calculationMode;
```

示例如下：

```
    CAKeyframeAnimation * ani = [CAKeyframeAnimation animationWithKeyPath:@"position"];
    ani.values = @[[NSValue valueWithCGPoint:CGPointMake(100, 100)],[NSValue valueWithCGPoint:CGPointMake(120, 100)],[NSValue valueWithCGPoint:CGPointMake(120, 200)],[NSValue valueWithCGPoint:CGPointMake(200, 200)]];
    ani.duration = 3;
    [layer addAnimation:ani forKey:@""];
```

#### 5.CASpringAnimation阻尼动画

        通过CASpringAnimation，可以帮助开发者很轻松的创建出有弹簧效果的动画，主要属性如下：

```
//这个属性设置弹簧重物的质量 会影响惯性 必须大于0 默认为1
@property CGFloat mass;
//设置弹簧的刚度系数，必须大于0 默认为100  这个越大 则回弹越快
@property CGFloat stiffness;
//阻尼系数 默认为10 必须大于0 这个值越大 回弹的幅度越小
@property CGFloat damping;
//初始速度
@property CGFloat initialVelocity;
//获取动画停下来需要的时间
@property(readonly) CFTimeInterval settlingDuration;
```

#### 6.CATransition转场动画

        CATransition和CAPropertyAnimation的不同之处在于当layer层出现时，会产生动画效果，而并不是属性改变时，属性如下：

```
/*
设置动画类型
//淡入
 NSString * const kCATransitionFade;
 //移入
 NSString * const kCATransitionMoveIn;
 //压入
 NSString * const kCATransitionPush;
 //溶解
 NSString * const kCATransitionReveal;
*/
@property(copy) NSString *type;
/*
设置动画的方向
//从右侧进
 NSString * const kCATransitionFromRight;
 //从左侧进
 NSString * const kCATransitionFromLeft;
 //从上侧进
 NSString * const kCATransitionFromTop;
 //从下侧进
 NSString * const kCATransitionFromBottom;
*/
@property(nullable, copy) NSString *subtype;
```

其实，关于type定义的动画效果，出来官方定义的，我们还可以使用一些私有的参数，如下：

```
pageCurl   翻页
rippleEffect 滴水效果
suckEffect 收缩效果，如一块布被抽走
cube 立方体效果
oglFlip 上下翻转效果
```

例如：

```
    CATransition * ani = [CATransition animation];
    ani.type =  @"pageCurl";
    ani.subtype = kCATransitionFromRight;
    [layer addAnimation:ani forKey:@""];
```

#### 7.CAAnimationGroup动画组

        CAAnimationGroup本身并没有定义动画，他可以将我们上面提到的相关动画进行组合：

```
@property(nullable, copy) NSArray<CAAnimation *> *animations;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

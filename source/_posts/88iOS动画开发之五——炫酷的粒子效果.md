---
title: iOS动画开发之五——炫酷的粒子效果
date: 2015-07-29
categories: iOS逻辑初窥
tags: []
---
## iOS动画开发之五——炫酷的粒子效果

        在上几篇博客中，我们对UIView层的动画以及iOS的核心动画做了介绍，基本已经可以满足iOS应用项目中所有的动画需求，如果你觉得那些都还不够炫酷，亦或是你灵光一现，想用UIKit框架写出一款炫酷的休闲游戏，那个有一个东西可以帮到你：iOS的粒子效果引擎。

### 一、粒子发射器

        iOS中的粒子效果有两部分组成，一部分为发射器，设置例子发射的宏观属性，另一部分是粒子单元，用于设置相应的粒子属性。粒子发射器是基于Layer层，没错，又是Layer，他的全名叫做：

CAEmitterLayer。其中常用的属性如下：

[@property](http://my.oschina.net/property)(copy) NSArray *emitterCells;

    粒子单元数组，例如你在绘制火焰的效果时，你可以创建两个单元，一个单元负责烟雾，一个单元负责火苗。

[@property](http://my.oschina.net/property)  float birthRate;

    粒子的创建速率，默认为1/s。

[@property](http://my.oschina.net/property)  float lifetime;

    粒子的存活时间。默认为1S。

[@property](http://my.oschina.net/property) CGPoint emitterPosition;

    发射器在xy平面的中心位置

[@property](http://my.oschina.net/property) CGFloat emitterZPosition;

    发射器在Z平面的位置

@property CGSize emitterSize;

    发射器的尺寸大小

@property CGFloat emitterDepth;

    发射器的深度，在某些模式下会产生立体效果

@property(copy) NSString *emitterShape;

    发射器的形状，这个参数的几个系统字符串如下：

```
CA_EXTERN NSString * const kCAEmitterLayerPoint
    __OSX_AVAILABLE_STARTING (__MAC_10_6, __IPHONE_5_0); //点的形状，粒子从一个点发出
CA_EXTERN NSString * const kCAEmitterLayerLine
    __OSX_AVAILABLE_STARTING (__MAC_10_6, __IPHONE_5_0);//线的形状，粒子从一条线发出
CA_EXTERN NSString * const kCAEmitterLayerRectangle
    __OSX_AVAILABLE_STARTING (__MAC_10_6, __IPHONE_5_0);//矩形形状，粒子从一个矩形中发出
CA_EXTERN NSString * const kCAEmitterLayerCuboid
    __OSX_AVAILABLE_STARTING (__MAC_10_6, __IPHONE_5_0);//立方体形状，会影响Z平面的效果
CA_EXTERN NSString * const kCAEmitterLayerCircle
    __OSX_AVAILABLE_STARTING (__MAC_10_6, __IPHONE_5_0);//圆形，粒子会在圆形范围发射
CA_EXTERN NSString * const kCAEmitterLayerSphere
    __OSX_AVAILABLE_STARTING (__MAC_10_6, __IPHONE_5_0);//球型
```

@property(copy) NSString *emitterMode;

    发射器的发射模式，参数如下：

```
CA_EXTERN NSString * const kCAEmitterLayerPoints
    __OSX_AVAILABLE_STARTING (__MAC_10_6, __IPHONE_5_0);//从发射器中发出
CA_EXTERN NSString * const kCAEmitterLayerOutline
    __OSX_AVAILABLE_STARTING (__MAC_10_6, __IPHONE_5_0);//从发射器边缘发出
CA_EXTERN NSString * const kCAEmitterLayerSurface
    __OSX_AVAILABLE_STARTING (__MAC_10_6, __IPHONE_5_0);//从发射器表面发出
CA_EXTERN NSString * const kCAEmitterLayerVolume
    __OSX_AVAILABLE_STARTING (__MAC_10_6, __IPHONE_5_0);//从发射器中点发出
```

@property(copy) NSString *renderMode;

    发射器渲染模式，参数如下：

```
CA_EXTERN NSString * const kCAEmitterLayerUnordered
    __OSX_AVAILABLE_STARTING (__MAC_10_6, __IPHONE_5_0);//这种模式下，粒子是无序出现的，多个发射源将混合
CA_EXTERN NSString * const kCAEmitterLayerOldestFirst
    __OSX_AVAILABLE_STARTING (__MAC_10_6, __IPHONE_5_0);//这种模式下，声明久的粒子会被渲染在最上层
CA_EXTERN NSString * const kCAEmitterLayerOldestLast
    __OSX_AVAILABLE_STARTING (__MAC_10_6, __IPHONE_5_0);//这种模式下，年轻的粒子会被渲染在最上层
CA_EXTERN NSString * const kCAEmitterLayerBackToFront
    __OSX_AVAILABLE_STARTING (__MAC_10_6, __IPHONE_5_0);//这种模式下，粒子的渲染按照Z轴的前后顺序进行
CA_EXTERN NSString * const kCAEmitterLayerAdditive
    __OSX_AVAILABLE_STARTING (__MAC_10_6, __IPHONE_5_0);//这种模式会进行粒子混合
```

@property BOOL preservesDepth;

    是否开启三维空间效果

@property  float velocity;

    粒子的运动速度

@property  float scale;

    粒子的缩放大小

@property  float spin;

    粒子的旋转位置

@property  unsigned  int seed;

    初始化随机的粒子种子

### 二、粒子单元

        设置好了粒子发射器，我们还需要初始化一些粒子单元，设置具体粒子的属性，我们使用到的类是CAEmitterCell这个类。

\+ (instancetype)emitterCell;

类方法创建发射单元

@property(copy) NSString *name;

设置发射单元的名称

@property(getter=isEnabled) BOOL enabled;

是否允许发射器渲染

@property  float birthRate;

粒子的创建速率

@property  float lifetime;

粒子的生存时间

@property float lifetimeRange;

粒子的生存时间容差

@property CGFloat emissionLatitude;

粒子在Z轴方向的发射角度

@property CGFloat emissionLongitude;

粒子在xy平面的发射角度

@property CGFloat emissionRange;

粒子发射角度的容差

@property CGFloat velocity;

粒子的速度

@property CGFloat velocityRange;

粒子速度的容差

@property CGFloat xAcceleration;

@property CGFloat yAcceleration;

@property CGFloat zAcceleration;

x，y，z三个方向的加速度

@property  CGFloat scale;

@property CGFloat scaleRange;

@property CGFloat scaleSpeed;

缩放大小，缩放容差和缩放速度

@property  CGFloat spin;

@property CGFloat spinRange;

旋转度与旋转容差

@property  CGColorRef color;

粒子的颜色

@property  float redRange;

@property  float greenRange;

@property  float blueRange;

@property  float alphaRange;

粒子在rgb三个色相上的容差和透明度的容差

@property  float redSpeed;

@property  float greenSpeed;

@property  float blueSpeed;

@property  float alphaSpeed;

粒子在RGB三个色相上的变化速度和透明度的变化速度

@property(strong) id contents;

渲染粒子，可以设置为一个CGImage的对象

@property CGRect contentsRect;

渲染的范围

### 三、让我们来“火”一把

        通过上面的介绍，我们来应用这些创造一团火，代码示例如下：

```
@interface ViewController ()
{
    CAEmitterLayer * _fireEmitter;//发射器对象
}
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    self.view.backgroundColor=[UIColor blackColor];
    //设置发射器
    _fireEmitter=[[CAEmitterLayer alloc]init];
    _fireEmitter.emitterPosition=CGPointMake(self.view.frame.size.width/2,self.view.frame.size.height-20);
    _fireEmitter.emitterSize=CGSizeMake(self.view.frame.size.width-100, 20);
    _fireEmitter.renderMode = kCAEmitterLayerAdditive;
    //发射单元
    //火焰
    CAEmitterCell * fire = [CAEmitterCell emitterCell];
    fire.birthRate=800;
    fire.lifetime=2.0;
    fire.lifetimeRange=1.5;
    fire.color=[[UIColor colorWithRed:0.8 green:0.4 blue:0.2 alpha:0.1]CGColor];
    fire.contents=(id)[[UIImage imageNamed:@"Particles_fire.png"]CGImage];
    [fire setName:@"fire"];
    
    fire.velocity=160;
    fire.velocityRange=80;
    fire.emissionLongitude=M_PI+M_PI_2;
    fire.emissionRange=M_PI_2;
    
    
    
    fire.scaleSpeed=0.3;
    fire.spin=0.2;
    
    //烟雾
    CAEmitterCell * smoke = [CAEmitterCell emitterCell];
    smoke.birthRate=400;
    smoke.lifetime=3.0;
    smoke.lifetimeRange=1.5;
    smoke.color=[[UIColor colorWithRed:1 green:1 blue:1 alpha:0.05]CGColor];
    smoke.contents=(id)[[UIImage imageNamed:@"Particles_fire.png"]CGImage];
    [fire setName:@"smoke"];
    
    smoke.velocity=250;
    smoke.velocityRange=100;
    smoke.emissionLongitude=M_PI+M_PI_2;
    smoke.emissionRange=M_PI_2;
    
    _fireEmitter.emitterCells=[NSArray arrayWithObjects:smoke,fire,nil];
    [self.view.layer addSublayer:_fireEmitter];
                
}
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/0729/115815_A13M_2340880.png)

看到效果了么？这次够炫酷了吧，改改其它属性，尽情的玩吧！

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

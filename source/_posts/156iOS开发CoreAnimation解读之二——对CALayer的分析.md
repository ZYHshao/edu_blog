---
title: iOS开发CoreAnimation解读之二——对CALayer的分析
date: 2015-11-26
categories: iOS逻辑初窥
tags: []
---
## iOS开发CoreAnimation解读之二——对CALayer的分析

### 一、UIView中的CALayer属性

#### 1.Layer专门负责view的视图渲染

        每一个UIView的对象中都有一个layer这样的属性，并且layer会负责view中有关图形绘制的相关操作，例如我们设置view的背景颜色和设置layer的背景颜色都是有效的，并且，设置view的背景色依然是通过layer来展示的，我们可以写如下的测试代码：

```
    UIView * view = [[UIView alloc]initWithFrame:CGRectMake(100, 100, 100, 100)];
    view.backgroundColor = [UIColor redColor];
    
    UIView * view2 = [[UIView alloc]initWithFrame:CGRectMake(100, 300, 100, 100)];
    view2.layer.backgroundColor = view.layer.backgroundColor;
    [self.view addSubview:view];
    [self.view addSubview:view2];
```

![](http://static.oschina.net/uploads/space/2015/1125/103123_6Jl7_2340880.png)

可以看出，我们设置view的backgroundColor属性其实起作用的也是layer的backgroundColor。

#### 2.自定义view默认layer属性的类

        UIView是很多视图类的父类，根据功能不同，会分出UIImageView，UIScrollerView，UITableView等，CALayer也相似，其也可以根据功能分出许多子类，还可以根据我们的需求自定义一个Layer类。UIView其中的layer默认是CALyer类，我们也可以通过重写View中的如下方法来使其创建我们需要的layer类：

```
+(Class)layerClass{
}
```

例如我们自定义一个View类，在自定义一个Layer类，是的自定义的View默认创建的layer是自定义的layer：

![](http://static.oschina.net/uploads/space/2015/1125/114958_xmKv_2340880.png)

在MyView中重写上述方法：

```
+(Class)layerClass{
    return [MyLayer  class];
}
```

在MyLayer中进行一些自定义：

```
- (instancetype)init
{
    self = [super init];
    if (self) {
        self.backgroundColor = [UIColor redColor].CGColor;
    }
    return self;
}
```

之后我们使用这个MyView的对象时，layer层的背景色就是红色的了。

### 二、几种系统的Layer类

        前边说过，和UIView相似，CALayer也很据功能衍生出许多子类，系统系统给我们可以使用的有如下几种：

#### 1.CAEmitterLayer

CoreAnimation框架中的CAEmitterLayer是一个粒子发射器系统，负责粒子的创建和发射源属性。通过它，我们可以轻松创建出炫酷的粒子效果。

#### 2.CAGradientLayer

CAGradientLayer可以创建出色彩渐变的图层效果，如下：

![](http://static.oschina.net/uploads/space/2015/1125/141858_Mnu0_2340880.png)

#### 3.CAEAGLLayer

CAEAGLLayer可以通过OpenGL ES来进行界面的绘制。

#### 4.CAReplicatorLayer

CAReplicatorLayer是一个layer容器，会对其中的subLayer进行复制和属性偏移，通过它，可以创建出类似倒影的效果，也可以进行变换复制，如下：

![](http://static.oschina.net/uploads/space/2015/1125/151642_Awcc_2340880.png)

#### 5.CAScrollLayer

CAScrollLayer可以支持其上管理的多个子层进行滑动，但是只能通过代码进行管理，不能进行用户点按触发。

#### 6.CAShapeLayer

CAShapeLayer可以让我们在layer层是直接绘制出自定义的形状。

#### 7.CATextLayer

CATextLayer可以通过字符串进行文字的绘制。

#### 8.CATiledLayer

CATiledLayer类似瓦片视图，可以将绘制分区域进行，常用于一张大的图片的分不分绘制。

#### 9.CATransformLayer

CATransformLayer用于构建一些3D效果的图层。

### 三、设置与调整Layer层的内容

设置层的内容有下面三种方式：

1.可以通过设置CGImage为layer的内容。

2.可以通过代理方法来动态修改或者绘制层的内容。

3.通过自定义CALayer对象来创建层的内容。

当你设置了Layer的内容后，例如设置了一张图片，内容的尺寸不一定会刚好和layer的尺寸合适，我们可以对其位置的调整，使其达到我们想要的效果，contentsGravity属性决定了内容对齐与填充方式，它可以分为两个方面：

1.不改变内容的原始大小

这种模式中不会改变内容的原始大小，如果层的尺寸小于内容的尺寸，则内容会被切割，如果层的尺寸大于内容的尺寸，多出的部分将会显示层的背景颜色。下面的这些设置方式为这种模式：

```
CA_EXTERN NSString * const kCAGravityCenter
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);
CA_EXTERN NSString * const kCAGravityTop
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);
CA_EXTERN NSString * const kCAGravityBottom
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);
CA_EXTERN NSString * const kCAGravityLeft
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);
CA_EXTERN NSString * const kCAGravityRight
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);
CA_EXTERN NSString * const kCAGravityTopLeft
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);
CA_EXTERN NSString * const kCAGravityTopRight
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);
CA_EXTERN NSString * const kCAGravityBottomLeft
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);
CA_EXTERN NSString * const kCAGravityBottomRight
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);
```

每个参数对应的对其模式如下图：

![](http://static.oschina.net/uploads/space/2015/1125/174950_GFuW_2340880.png)

2.改变内容的尺寸大小

这种模式设置的实际上是一种填充方式，参数如下：

```
CA_EXTERN NSString * const kCAGravityResize
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);
CA_EXTERN NSString * const kCAGravityResizeAspect
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);
CA_EXTERN NSString * const kCAGravityResizeAspectFill
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);
```

![](http://static.oschina.net/uploads/space/2015/1125/175202_z9ZM_2340880.png)

### 四、CALayer的接口应用总结

#### 1、创建与初始化layer相关

```
//通过类方法创建并初始化一个layer
+ (instancetype)layer;
//初始化方法
- (instancetype)init;
//通过一个layer创建一个副本
- (instancetype)initWithLayer:(id)layer;
```

#### 2、渲染层layer与模型层layer

    在CALayer中，有如下两个属性，他们都返回一个CALayer的对象：

```
//渲染层layer
- (nullable id)presentationLayer;
//模型层layer
- (id)modelLayer;
```

对于presentationLayer，这个属性不一定总会返回一个实体对象，只有当进行动画或者其他渲染的操作时，这个属性会返回一个在当前屏幕上的layer，不且每一次执行，这个对象都会不同，它是原layer的一个副本presentationLayer的modelLayer就是其实体layer层。

对于modelLayer，它会返回当前的存储信息的Layer，也是当前的layer对象，始终唯一。

#### 3.一些属性与方法

```
+ (nullable id)defaultValueForKey:(NSString *)key;
```

上面这个属性用于设置layer中默认属性的值，我们可以在子类中重写这个方法来改变默认创建的layer的一些属性，例如如下代码，我们创建出来的layer就默认有红色的背景颜色：

```
+(id)defaultValueForKey:(NSString *)key{
    if ([key isEqualToString:@"backgroundColor"]) {
        return (id)[UIColor redColor].CGColor;
    }
    return [super defaultValueForKey:key];
}
```

```
//这个方法也只使用在子类中重写，用于设置在某些属性改变时是否进行layer重绘
+ (BOOL)needsDisplayForKey:(NSString *)key;
//子类重写这个方法设置属性是否可以被归档
- (BOOL)shouldArchiveValueForKey:(NSString *)key;
/*********************************************/
//设置layer尺寸
@property CGRect bounds;
//设置layer位置
@property CGPoint position;
//设置其在父layer中的层次，默认为0，这个值越大，层次越靠上
@property CGFloat zPosition;
//锚点
@property CGPoint anchorPoint;
//在Z轴上的锚点位置 3D变换时会有很大影响
@property CGFloat anchorPointZ;
//进行3D变换
@property CATransform3D transform;
//获取和设置CGAffineTransform变换
- (CGAffineTransform)affineTransform;
- (void)setAffineTransform:(CGAffineTransform)m;
//设置layer的frame
@property CGRect frame;
//设置是否隐藏
@property(getter=isHidden) BOOL hidden;
//每个layer层有两面，这个属性确定是否两面都显示
@property(getter=isDoubleSided) BOOL doubleSided;
//是否进行y轴的方向翻转
@property(getter=isGeometryFlipped) BOOL geometryFlipped;
//获取当前layer内容y轴方向是否被翻转了
- (BOOL)contentsAreFlipped;
//父layer视图
@property(nullable, readonly) CALayer *superlayer;
//从其父layer层上移除
- (void)removeFromSuperlayer;
//所有子layer数组
@property(nullable, copy) NSArray<CALayer *> *sublayers;
//添加一个字layer
- (void)addSublayer:(CALayer *)layer;
//插入一个子layer
- (void)insertSublayer:(CALayer *)layer atIndex:(unsigned)idx;
//将一个子layer插入到最下面
- (void)insertSublayer:(CALayer *)layer below:(nullable CALayer *)sibling;
//将一个子layer插入到最上面
- (void)insertSublayer:(CALayer *)layer above:(nullable CALayer *)sibling;
//替换一个子layer
- (void)replaceSublayer:(CALayer *)layer with:(CALayer *)layer2;
//对其子layer进行3D变换
@property CATransform3D sublayerTransform;
//遮罩层layer
@property(nullable, strong) CALayer *mask;
//舍否进行bounds的切割，在设置圆角属性时会设置为YES
@property BOOL masksToBounds;
//下面这些方法用于坐标转换
- (CGPoint)convertPoint:(CGPoint)p fromLayer:(nullable CALayer *)l;
- (CGPoint)convertPoint:(CGPoint)p toLayer:(nullable CALayer *)l;
- (CGRect)convertRect:(CGRect)r fromLayer:(nullable CALayer *)l;
- (CGRect)convertRect:(CGRect)r toLayer:(nullable CALayer *)l;
//返回包含某一点的最上层的子layer
- (nullable CALayer *)hitTest:(CGPoint)p;
//返回layer的bounds内是否包含某一点
- (BOOL)containsPoint:(CGPoint)p;
//设置layer的内容，一般会设置为CGImage的对象
@property(nullable, strong) id contents;
//获取内容的rect尺寸
@property CGRect contentsRect;
//设置内容的填充和对其方式，具体上面有说
@property(copy) NSString *contentsGravity;
//设置内容的缩放
@property CGFloat contentsScale;
```

下面这个属性和内容拉伸相关：

```
@property CGRect contentsCenter;
```

这个属性确定一个矩形区域，当内容进行拉伸或者缩放的时候，这一部分的区域是会被形变的，例如默认设置为(0,0,1,1)，则整个内容区域都会参与形变。如果我们设置为(0.25,0.25,0.5,0.5),那么只有中间0.5*0.5比例宽高的区域会被拉伸，四周都不会。

下面这两个属性用来设置缩放或拉伸的模式：

```
//设置缩小的模式
@property(copy) NSString *minificationFilter;
//设置放大的模式
@property(copy) NSString *magnificationFilter;
//缩放因子
@property float minificationFilterBias;
//模式参数如下
//临近插值
NSString * const kCAFilterNearest;
//线性拉伸
NSString * const kCAFilterLinear;
//瓦片复制拉伸
NSString * const kCAFilterTrilinear;
```

```
//设置内容是否完全不透明
@property(getter=isOpaque) BOOL opaque;
//重新加载绘制内容
- (void)display;
//设置内容为需要重新绘制
- (void)setNeedsDisplay;
//设置某一区域内容需要重新绘制
- (void)setNeedsDisplayInRect:(CGRect)r;
//获取是否需要重新绘制
- (BOOL)needsDisplay;
//如果需要，进行内容重绘
- (void)displayIfNeeded;
//这个属性设置为YES，当内容改变时会自动调用- (void)setNeedsDisplay函数
@property BOOL needsDisplayOnBoundsChange;
//绘制与读取内容
- (void)drawInContext:(CGContextRef)ctx;
- (void)renderInContext:(CGContextRef)ctx;
```

```
//设置背景颜色
@property(nullable) CGColorRef backgroundColor;
//设置圆角半径
@property CGFloat cornerRadius;
//设置边框宽度
@property CGFloat borderWidth;
//设置边框颜色
@property(nullable) CGColorRef borderColor;
//设置透明度
@property float opacity;
//设置阴影颜色
@property(nullable) CGColorRef shadowColor;
//设置阴影透明度
@property float shadowOpacity;
//设置阴影偏移量
@property CGSize shadowOffset;
//设置阴影圆角半径
@property CGFloat shadowRadius;
//设置阴影路径
@property(nullable) CGPathRef shadowPath;
```

```
//添加一个动画对象 key值起到id的作用，通过key值，可以取到这个动画对象
- (void)addAnimation:(CAAnimation *)anim forKey:(nullable NSString *)key;
//移除所有动画对象
- (void)removeAllAnimations;
//移除某个动画对象
- (void)removeAnimationForKey:(NSString *)key;
//获取所有动画对象的key值
- (nullable NSArray<NSString *> *)animationKeys;
//通过key值获取动画对象
- (nullable CAAnimation *)animationForKey:(NSString *)key;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

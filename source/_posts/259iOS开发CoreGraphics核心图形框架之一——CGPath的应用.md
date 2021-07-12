---
title: iOS开发CoreGraphics核心图形框架之一——CGPath的应用
date: 2016-10-11
categories: iOS逻辑初窥
tags: []
---
## iOS开发CoreGraphics核心图形框架之一——CGPath的应用

### 一、引言

    CoreGraphics核心图形框架相较于UIKit框架更加偏于底层。在Objective-C工程中，CoreGraphics其中方法都是采用C语言风格进行编写的，同时其并不支持Objective-C的自动引用计数，在使用这个框架进行编程时，开发者要手动对内存进行管理。在Swift工程中，Apple使用Swift语言对CoreGraphics矿建进行了重构，将CGPath，CGMutablePaht等都重新定义为了类。CGPath可以理解为图形的路径，在Objective-C工程中，其实系统定义的一个内部结构体，开发者不可以直接使用，开发者CGPathRef和CGMutablePathRef别名作为CGPath的引用，实际上，CGPathRef和CGMutablePathRef都是CGPath结构体类型的指针，不同的是一个是const类型不可修改的，一个是可以修改的，系统定义如下：

```objectivec
typedef struct CGPath *CGMutablePathRef;
typedef const struct CGPath *CGPathRef;
```

### 二、CGPath创建与内存管理的相关方法

    关于CGPath的创建与内存管理的相关方法，列举如下：

```objectivec
//这个方法获取CGPath类在CoreGraphics框架中的唯一标识
//CFTypeID 实际上是无符号整型的别名 其为CoreGraphics框架中每个类都定义了一个标识 CGPath为280
CFTypeID CGPathGetTypeID(void);
//这个方法创建一个srtuct CGPath * 指针 可以理解为可变的CGPath类
CGMutablePathRef  CGPathCreateMutable(void);
//这个方法通过一个CGPathRef来创建CGPathRef 
CGPathRef  CGPathCreateCopy(CGPathRef  path);
//这个方法在通过CGPathRef创建CGPathRef时会将得路径进行transform变换后返回
CGPathRef  CGPathCreateCopyByTransformingPath(CGPathRef path, const CGAffineTransform * transform);
//这个方法通过CGPathRef创建可变的CGMutablePathRef
CGMutablePathRef CGPathCreateMutableCopy(CGPathRef path);
//意义同上，在创建的CGMutablePathRef基础上进行一次transform变换在返回
CGMutablePathRef CGPathCreateMutableCopyByTransformingPath(CGPathRef path, const CGAffineTransform * transform)
//这个方法将创建矩形路径 第一个参数为要绘制的矩形区域 第2个参数为要进行的transform变换
CGPathRef  CGPathCreateWithRect(CGRect rect,const CGAffineTransform * transform);
//这个方法将创建椭圆形路径
CGPathRef CGPathCreateWithEllipseInRect(CGRect rect, const CGAffineTransform *  transform);
//这个方法用于创建圆角矩形路径
/*
rect :绘制的矩形区域
cornerWidth: 横向圆角尺寸
cornerHeight:纵向圆角尺寸
*/
CGPathRef CGPathCreateWithRoundedRect(CGRect rect, CGFloat cornerWidth, CGFloat cornerHeight,const CGAffineTransform * transform);
//这个方法用于创建虚线路径
/*
这个方法略微有些复杂 其中参数意义如下：
path：要进行虚线化的路径
phase：从lengths数组的第几部分开始绘制虚线
lengths:C风格的数组 其中为CGFloat值 表示每段虚线的绘制长度 例如传入数组为{10,5}，则虚线的先绘制长度为10的实线 在绘制长度为5的空白 在进行循环
count：这个参数需要设置为lengths数组的长度
*/
CGPathRef CGPathCreateCopyByDashingPath(CGPathRef  path, const CGAffineTransform *  transform,CGFloat phase, const CGFloat *lengths, size_t count);
//通过CGPathRef来创建斜线
/*
lineWidth:设置线宽
lineCap:设置线帽风格 可选参数如下：
typedef CF_ENUM(int32_t, CGLineCap) {
    kCGLineCapButt,  默认的风格 线的端点精确到点
    kCGLineCapRound, 圆滑的端点 线的端点为半径为线宽一半的圆弧
    kCGLineCapSquare 尖锐的过渡
};
lineJoin：设置连接线处的风格 可选参数如下：
typedef CF_ENUM(int32_t, CGLineJoin) {
    kCGLineJoinMiter, //以锋利的角作为连接线的转折
    kCGLineJoinRound, //以圆角作为连接线的转折
    kCGLineJoinBevel  //贝塞尔风格的转折
};
miterLimit：这个值将决定线连接处角的锋利程度
*/
CGPathRef CGPathCreateCopyByStrokingPath(CGPathRef cg_nullable path, const CGAffineTransform * __nullable transform,CGFloat lineWidth, CGLineCap lineCap,CGLineJoin lineJoin, CGFloat miterLimit);
//手动使CGPathRef引用计数+1
CGPathRef CGPathRetain(CGPathRef cg_nullable path);
//手动使CGPathRef引用计数-1
void CGPathRelease(CGPathRef cg_nullable path);
```

自定义一个View视图，在其drawRect方法中进行界面的绘制，示例代码如下：

```objectivec
- (void)drawRect:(CGRect)rect {
    //获取当前绘图上下文
    CGContextRef contextRef = UIGraphicsGetCurrentContext();
    CGPoint center = CGPointMake(rect.size.width/2, rect.size.height/2);
    //创建圆角矩形路径
    CGPathRef pathRef = CGPathCreateWithRoundedRect(CGRectMake(center.x-50, center.y-50, 100, 100), 30, 10, nil);
    //将路径虚线化
    CGFloat floats[] = {10,5};
    pathRef = CGPathCreateCopyByDashingPath(pathRef, nil, 0, floats, 2);
    //设置绘制颜色
    [[UIColor redColor] setStroke];
    //将路径添加到绘图上下文中
    CGContextAddPath(contextRef, pathRef);
    //进行绘制
    CGContextDrawPath(contextRef, kCGPathStroke);
    //内存释放
    CGPathRelease(pathRef);
}
```

运行后效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1011/165512_mRDm_2340880.png)

### 三、CGPath的路径绘制相关方法

```objectivec
//将路径移动到一个点作为起点
void CGPathMoveToPoint(CGMutablePathRef  path,const CGAffineTransform * m, CGFloat x, CGFloat y);
//将路径移动到某个点画出一条线
void CGPathAddLineToPoint(CGMutablePathRef  path,const CGAffineTransform *  m, CGFloat x, CGFloat y);
//向路径中添加一段二次贝塞尔曲线
/*
cpx:控制点的x坐标
cpy:控制点的y坐标
*/
void CGPathAddQuadCurveToPoint(CGMutablePathRef path,const CGAffineTransform * m, CGFloat cpx, CGFloat cpy,CGFloat x, CGFloat y);
//添加一段三次贝塞尔曲线
void CGPathAddCurveToPoint(CGMutablePathRef path,const CGAffineTransform * m, CGFloat cp1x, CGFloat cp1y,CGFloat cp2x, CGFloat cp2y, CGFloat x, CGFloat y);
//这个方法用于闭合路径 调用这个方法后 路径最后的端点将和起点闭合
void CGPathCloseSubpath(CGMutablePathRef path);
//向路径中追加一个矩形
void CGPathAddRect(CGMutablePathRef  path, const CGAffineTransform * m, CGRect rect);
//向路径中追加一组矩形
void CGPathAddRects(CGMutablePathRef path, const CGAffineTransform *  m, const CGRect *  rects,size_t count);
//向路径中追加一组线条
void CGPathAddLines(CGMutablePathRef path, const CGAffineTransform *  m, const CGPoint * __nullable points, size_t count);
//添加一组椭圆
void CGPathAddEllipseInRect(CGMutablePathRef cg_nullable path,const CGAffineTransform * m, CGRect rect);
//向路径中追加一组圆弧
/*
x:圆心x坐标
y:圆心y坐标
radius:弧线半径
startAngle:起始角度
endAngle:终止角度
clockwise:是否顺时针绘制
*/
void CGPathAddArc(CGMutablePathRef  path, const CGAffineTransform * m, CGFloat x, CGFloat y, CGFloat radius, CGFloat startAngle, CGFloat endAngle, bool clockwise);
//向路径中追加一组圆弧
/*
x:圆心x坐标
y:圆心y坐标
radius:弧线半径
startAngle:起始角度
delta:圆弧绘制的长度 为弧度制 2π为整个圆
*/
void CGPathAddRelativeArc(CGMutablePathRef  path, const CGAffineTransform * __nullable matrix, CGFloat x, CGFloat y, CGFloat radius, CGFloat startAngle, CGFloat delta);
//向路径中追加一段圆弧 弧线是以(x1,y1)到(x2,y2)为切线的弧线
void CGPathAddArcToPoint(CGMutablePathRef path,const CGAffineTransform * m, CGFloat x1, CGFloat y1, CGFloat x2, CGFloat y2, CGFloat radius);
//向路径中追加一段路径
void CGPathAddPath(CGMutablePathRef  path1,const CGAffineTransform *  m, CGPathRef path2);
```

示例代码如下：

```objectivec
- (void)drawRect:(CGRect)rect {
    CGContextRef contextRef = UIGraphicsGetCurrentContext();
    CGPoint center = CGPointMake(rect.size.width/2, rect.size.height/2);
    CGMutablePathRef pathRef = CGPathCreateMutable();
    CGPathMoveToPoint(pathRef, nil, center.x, center.y-50);
    CGPathAddLineToPoint(pathRef, nil, center.x+100, center.y);
    CGPathAddQuadCurveToPoint(pathRef, nil, 0, 0, center.x+100, center.y-100);
    CGPathAddRelativeArc(pathRef, nil, 100, 100, 50, 0, M_PI);
    CGPathCloseSubpath(pathRef);
    [[UIColor redColor] setStroke];
    CGContextAddPath(contextRef, pathRef);
    CGContextDrawPath(contextRef, kCGPathStroke);
    CGPathRelease(pathRef);
    CGContextRelease(contextRef);
}
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1011/175323_dBOM_2340880.png)

### 四、CGPath中的其他方法汇总

```objectivec
//判断某个路径是否为空
bool CGPathIsEmpty(CGPathRef path);
//判断某个路径是否为某个矩形
bool CGPathIsRect(CGPathRef cg_nullable path, CGRect *  rect);
//获取某个路径当前绘制所在的点
CGPoint CGPathGetCurrentPoint(CGPathRef path);
//获取某个路径包含所有点的尺寸
CGPathGetBoundingBox(CGPathRef cg_nullable path);
//获取某个路径的尺寸
CGRect CGPathGetPathBoundingBox(CGPathRef path);
//判断路径是否包含某个点
bool CGPathContainsPoint(CGPathRef  path, const CGAffineTransform *  m, CGPoint point, bool eoFill);
```

### 五、关于CGPathElement结构体

        当每次向CGPath路径做操作时，操作的过程实际上都会被记录下来，每个操作行为节点都被封装为了CGPathElement结构体，开发者可以通过如下方法来获取所有操作行为：

```objectivec
CGPathApply(pathRef, nil, func);
```

CGPathApply()方法中的第3个参数为一个函数指针，示例C函数实现如下：

```objectivec
void func(void * __nullable info,
      const CGPathElement *  element){
    printf("%d",(*element).type);
}
```

CGPathElement结构体的定义如下：

```objectivec
struct CGPathElement {
  //操作节点的类型
  CGPathElementType type;
  //对应的点集
  CGPoint *  points;
};
//CGPathElementType枚举定义如下
typedef CF_ENUM(int32_t, CGPathElementType) {
  //移动到点的操作行为
  kCGPathElementMoveToPoint,
  //添加线的操作行为
  kCGPathElementAddLineToPoint,
  //添加二次贝塞尔曲线的操作行为
  kCGPathElementAddQuadCurveToPoint,
  //添加三次贝塞尔曲线的操作行为
  kCGPathElementAddCurveToPoint,
  //闭合路径的操作行为
  kCGPathElementCloseSubpath
};
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: iOS开发CoreGraphics核心图形框架之四——变换函数
date: 2016-11-02
categories: iOS逻辑初窥
tags: []
---
## iOS开发CoreGraphics核心图形框架之五——Patterns模型的应用

### 一、引言

    Patterns称为模型可能并不直观，说一个场景我们或许就可以更加容易的理解Patterns。在开发中，开发者经常会遇到这样的需求，将某个图片或者某个图形进行平铺作为界面的背景，当然iOS中有现成的方法来将图片转换为背景色进行背景的渲染，但是这种方式并不太灵活，例如背景花纹的着色，背景图片的平铺间距设置等需求都无法满足。Patterns就是用来处理这样的需求。

    Patterns可以理解为一个模型单元，即花纹背景中的一个花纹单元，开发者可以自定义这个单元的绘制内容，一旦创建了CGPatternRef引用，开发者就可以将它向普通颜色一样进行使用，可以进行填充，可以进行路径绘制等。

### 二、创建CGPatternRef模型引用

    在UIView子类的drawRect:方法中来做如下的测试：

```objectivec
- (void)drawRect:(CGRect)rect {
    // Drawing code
    //创建回调结构体 后面会介绍
    CGPatternCallbacks callback = {0,&drawPatternCallback,&releaseInfoCallback};
    //创建模型引用
    CGPatternRef pattren = CGPatternCreate(NULL, CGRectMake(0, 0, 30,30), CGAffineTransformIdentity, 35, 35, kCGPatternTilingConstantSpacing, false, &callback);
    //创建颜色数组 代表RGBA的值
    static const CGFloat color[4] = { 0, 1, 0, 1 };
    //创建颜色空间
    CGColorSpaceRef baseSpace = CGColorSpaceCreateWithName (kCGColorSpaceGenericRGB);
    CGColorSpaceRef patternSpace = CGColorSpaceCreatePattern (baseSpace);
    //设置填充颜色空间
    CGContextSetFillColorSpace (UIGraphicsGetCurrentContext(), patternSpace);
    //设置填充模型
    CGContextSetFillPattern(UIGraphicsGetCurrentContext(), pattren, color);
    //进行填充
    CGContextFillRect(UIGraphicsGetCurrentContext(), CGRectMake(0, 0, 200, 200));
    
}
```

上面的示例代码中，有几个地方需要进行介绍：

CGPatternCallBacks是CoreGraphics框架的CGPattern.h文件中定义的一个结构体，这个结构体组合了模型Pattern的版本，创建回调和释放回调。创建回调和释放回调需要传入两个方法块的地址，即block。这两个block的格式定义如下：

```objectivec
//创建模型回调的格式定义
//info参数为需要传递给回调函数的数据
//content参数为所绘制的图形上下文
typedef void (*CGPatternDrawPatternCallback)(void * __nullable info,
                                             CGContextRef cg_nullable context);
//释放回调 开发者可以在其中进行内存的释放
typedef void (*CGPatternReleaseInfoCallback)(void * __nullable info);
```

我们所实现的drawPatternCallback(),releaseInfoCallback()方法示例如下：

```objectivec
// 绘制 回调
#define PSIZE 16
void drawPatternCallback(void *info,CGContextRef myContext){
//这里我借用了官方文档中的代码 如下的代码将绘制出五角星
    int k;
    double r, theta;
    
    r = 0.8 * PSIZE / 2;
    theta = 2 * M_PI * (2.0 / 5.0); // 144 degrees
    
    CGContextTranslateCTM (myContext, PSIZE/2, PSIZE/2);
    
    CGContextMoveToPoint(myContext, 0, r);
    for (k = 1; k < 5; k++) {
        CGContextAddLineToPoint (myContext,
                                 r * sin(k * theta),
                                 r * cos(k * theta));
    }
    CGContextClosePath(myContext);
    CGContextFillPath(myContext);
}


// 移除 回调
void releaseInfoCallback(void *info) {
   
}
```

在回过来看创建CGPatternRef的方法：

```objectivec
/*
这个方法
第1个参数为要传递进创建模型方法的信息
第2个参数为设置每个模型单元的尺寸
第3个参数设置模型的几何变换
第4个参数设置模型的整体宽度  通过这个参数可以设置边距
第5个参数设置模型的整体高度  通过这个参数可以设置边距
第6个参数设置模型的渲染方式
第7个参数设置为有色渲染还是无色渲染 
第8个参数设置相关回调结构体
*/
CGPatternRef pattren = CGPatternCreate(NULL, CGRectMake(0, 0, 30,30), CGAffineTransformIdentity, 35, 35, kCGPatternTilingConstantSpacing, false, &callback);
```

关于模型的渲染方式，需要设置为CGPatternTiling类型的枚举，如下：

```objectivec
typedef CF_ENUM (int32_t, CGPatternTiling) {
    //无失真的平铺 将调整单元之间的间距
    kCGPatternTilingNoDistortion,
    //细微调整单元大小
    kCGPatternTilingConstantSpacingMinimalDistortion,
    //恒定间距，通过调整单元大小实现 会失真
    kCGPatternTilingConstantSpacing
};
```

CGContextSetFillPattern()方法用于将模型设置为要渲染界面的颜料，之后调用CGContextStrokePath(), CGContextFillPath(), CGContextFillRect()等相关方法都可以实现将模型铺平渲染到指定容器。需要注意，CGContextSetFillPattern()方法中第1个参数为绘图上下文，第2个参数为模型CGPatternRef引用，第3个参数为一个色值数组，这里如果模式是无色渲染方式创建的，需要传入4个元素的RGBA数组，如果是有色模式创建的，需要传入一个透明度值，可以是float类型的指针。

运行工程，效果如下图所示:

![](http://static.oschina.net/uploads/space/2016/1102/170429_FK7T_2340880.png)

将代码简单修改如下，就可以实现以五角星围成的矩形：

```objectivec
- (void)drawRect:(CGRect)rect {
    // Drawing code
    CGPatternCallbacks callback = {0,&drawPatternCallback,&releaseInfoCallback};
    CGPatternRef pattren = CGPatternCreate(NULL, CGRectMake(0, 0, 30,30), CGAffineTransformIdentity, 30, 30, kCGPatternTilingConstantSpacing, false, &callback);
    static const CGFloat color[4] = { 1, 0, 0, 1 };
    CGColorSpaceRef baseSpace;
    CGColorSpaceRef patternSpace;
    baseSpace = CGColorSpaceCreateWithName (kCGColorSpaceGenericRGB);
    patternSpace = CGColorSpaceCreatePattern (baseSpace);
    CGContextSetStrokeColorSpace (UIGraphicsGetCurrentContext(), patternSpace);
    CGContextSetStrokePattern(UIGraphicsGetCurrentContext(), pattren, color);
    CGContextSetLineWidth(UIGraphicsGetCurrentContext(), 40);
    CGContextStrokeRect(UIGraphicsGetCurrentContext(), CGRectMake(0, 0, 200, 200));
}
```

效果如下:

![](http://static.oschina.net/uploads/space/2016/1102/171253_SwsH_2340880.png)

### 三、CGPattern中其他方法

```objectivec
//获取CGPattern在CoreGraphics框架中的id
CFTypeID CGPatternGetTypeID(void);
//进行引用计数加1
CGPatternRef cg_nullable CGPatternRetain(CGPatternRef cg_nullable pattern);
//进行引用计数减1
void CGPatternRelease(CGPatternRef cg_nullable pattern);
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

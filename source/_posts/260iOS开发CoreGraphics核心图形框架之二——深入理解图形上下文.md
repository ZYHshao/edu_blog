---
title: iOS开发CoreGraphics核心图形框架之二——深入理解图形上下文
date: 2016-10-16
categories: iOS逻辑初窥
tags: []
---
## iOS开发CoreGraphics核心图形框架之二——深入理解图形上下文

### 一、引言

      在上一篇博客中，介绍了有关CGPath绘制路径的相关方法，其中在View视图的drawRect方法中，已经使用过上下文将Path路径绘制到当前视图上，上一篇博客只是抛砖引玉，本片博客将更深入的介绍下有关上下文的更多内容。关于接胡搜啊CGPath应用的博客地址如下：

iOS开发CoreGraphics核心图形框架之一——CGPath的应用：[https://my.oschina.net/u/2340880/blog/757072](https://my.oschina.net/u/2340880/blog/757072)。

### 二、关于图形上下文Graphics Context

      GraphicsContext对于开发者来说是完全透明的，开发者不需要关心其实现，也不需要关心其绘制方式，开发者只需要将要绘制的内容传递给图形上下文，由图形上下文来将内容绘制到对应的目标上。这个目标可以是视图，窗口，打印机，PDF文档或者位图对象。需要注意，绘制的顺序在CoreGraphics框架中十分重要，如果后绘制的内容和先绘制的内容有位置冲突，后绘制的内容将覆盖先绘制的内容。

    特定的上下文用于将内容绘制到特定的输出源上，CoreGraphics中提供如下几种图形上下文：

1.位图图形上下文：位图图形上下文用于将RGB图像，GMYK图像或者黑白图像绘制到一个位图(bitmap)对象中。

2.PDF图形上下文：PDF图形上下文可以帮助开发者创建PDF文件，将内容绘制进PDF文件中，其与位图上下文最大的区别在于PDF数据可以保存多页图像。

3.窗口上下文：用于OS系统中的窗口绘制。

4.图层上下文：用于将内容绘制在Layer图层上。

5.打印上下文：使用Mac打印功能时，此上下文用于将内容绘制在打印输出源上。

### 三、在UIKit框架中操作图形上下文

    在UIKit框架中有一个UIGraphics头文件，其中封装了许多对当前图形上下文进行操作的方法。首先任何UIView和其子类的视图控件都有一个drawRect方法，当视图将要被绘制时会调用这个方法，在drawRect方法中开发者可以获取到当前视图的图形上下文，通过这个图形上下文可以对视图进行自定义的绘制。UIGraphics头文件中定义的如下方法可以对当前的图形上下文进行操作：

```objectivec
//这个方法用于获取当前的图形上下文
UIKIT_EXTERN CGContextRef __nullable UIGraphicsGetCurrentContext(void) CF_RETURNS_NOT_RETAINED;
//这个方法用于将某个图形上下文对象压入栈中 使其变为当前的图形上下文
UIKIT_EXTERN void UIGraphicsPushContext(CGContextRef context);
//这个方法用于将当前的图形上下文出栈 当前的图形上下文始终是栈顶的图形上下文
UIKIT_EXTERN void UIGraphicsPopContext(void);
```

需要注意，上面的UIGraphicsPushContext()与UIGraphicsPopContext()方法常用于切换当前的图形上下文。

```objectivec
//下面这两个方法用于向当前的图形上下文中填充矩形 
UIKIT_EXTERN void UIRectFillUsingBlendMode(CGRect rect, CGBlendMode blendMode);
UIKIT_EXTERN void UIRectFill(CGRect rect);
//下面这两个方法用于向当前的图形上下文中绘制矩形边框
UIKIT_EXTERN void UIRectFrameUsingBlendMode(CGRect rect, CGBlendMode blendMode);
UIKIT_EXTERN void UIRectFrame(CGRect rect);
//这个方法用于裁剪当前的图形上下文的绘制区域
UIKIT_EXTERN void UIRectClip(CGRect rect);
```

上面方法中的CGBlendMode参数用于设置图像的混合模式，意义列举如下：

```objectivec
typedef CF_ENUM (int32_t, CGBlendMode) {
    //在背景图像之上绘制原图像
    kCGBlendModeNormal,
    //将背景与原图像进行混合
    kCGBlendModeMultiply,
    //将背景与原图像进行逆向混合
    kCGBlendModeScreen,
    //覆盖原图像 同时保持背景阴影
    kCGBlendModeOverlay,
    //进行灰度复合
    kCGBlendModeDarken,
    //进行亮度复合
    kCGBlendModeLighten,
    //复合时 黑色不进行复合
    kCGBlendModeColorDodge,
    //复合时 白色不进行复合
    kCGBlendModeColorBurn,
    //复合时 根据黑白色值比例进行复合
    kCGBlendModeSoftLight,
    kCGBlendModeHardLight,
    //复合时 将原图像中有关背景图像的色值去除
    kCGBlendModeDifference,
    //与kCGBlendModeDifference类似 对比度更低
    kCGBlendModeExclusion,
    //使用原图像的色调与饱和度
    kCGBlendModeHue,
    //同kCGBlendModeHue 纯灰度的区域不产生变化
    kCGBlendModeSaturation,
    //同kCGBlendModeHue 保留灰度等级
    kCGBlendModeColor,
    //与kCGBlendModeHue效果相反
    kCGBlendModeLuminosity,
    
    //下面这些枚举定义了MacOS中图像复合的计算方式 
    //R 结果  
    //S 原图像
    //D 背景图像
    //Ra Sa Da为带透明alpha通道

    kCGBlendModeClear,                  /* R = 0 */
    kCGBlendModeCopy,                   /* R = S */
    kCGBlendModeSourceIn,               /* R = S*Da */
    kCGBlendModeSourceOut,              /* R = S*(1 - Da) */
    kCGBlendModeSourceAtop,             /* R = S*Da + D*(1 - Sa) */
    kCGBlendModeDestinationOver,        /* R = S*(1 - Da) + D */
    kCGBlendModeDestinationIn,          /* R = D*Sa */
    kCGBlendModeDestinationOut,         /* R = D*(1 - Sa) */
    kCGBlendModeDestinationAtop,        /* R = S*(1 - Da) + D*Sa */
    kCGBlendModeXOR,                    /* R = S*(1 - Da) + D*(1 - Sa) */
    kCGBlendModePlusDarker,             /* R = MAX(0, (1 - D) + (1 - S)) */
    kCGBlendModePlusLighter             /* R = MIN(1, S + D) */
};

```

下面这些方法用于操作位图图形上下文：

```objectivec
//这个方法会创建一个位图图形上下文 并将其push进图形上下文栈中 size参数设置图像的大小
UIKIT_EXTERN void     UIGraphicsBeginImageContext(CGSize size);
//方法同上，其中opaque参数设置是否为不透明的 scale设置缩放因子
UIKIT_EXTERN void     UIGraphicsBeginImageContextWithOptions(CGSize size, BOOL opaque, CGFloat scale) NS_AVAILABLE_IOS(4_0);
//这个方法用于将当前的位图图形上下文内容画成UIImage对象
UIKIT_EXTERN UIImage* __nullable UIGraphicsGetImageFromCurrentImageContext(void);
//结束位图图形上下文的编辑 会POP出栈
UIKIT_EXTERN void     UIGraphicsEndImageContext(void); 
```

我们可以通过代码来画一个简单的UIImage图像，示例如下：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    //创建位图图形上下文 设置大小为200*200
    UIGraphicsBeginImageContext(CGSizeMake(200, 200));
    //获取到当前图形上下文
    CGContextRef ref = UIGraphicsGetCurrentContext();
    //裁剪其进行绘制的尺寸为100*100
    UIRectClip(CGRectMake(0, 0, 100, 100));
    //设置线条颜色
    [[UIColor redColor] setStroke];
    //设置填充颜色
    [[UIColor grayColor] setFill];
    //设置边框宽度
    CGContextSetLineWidth(ref, 10);
    //进行填充
    UIRectFill(CGRectMake(0, 0, 100, 100));
    //进行边框绘制
    UIRectFrame(CGRectMake(0, 0, 200, 200));
    //拿到UIImage实例
    UIImage * image = UIGraphicsGetImageFromCurrentImageContext();
    //结束位图上下文编辑
    UIGraphicsEndImageContext();
    //将UIImage展示到界面上
    UIImageView * imageView = [[UIImageView alloc]initWithImage:image];
    imageView.contentMode = UIViewContentModeCenter;
    imageView.frame = CGRectMake(100, 100, 200, 200);
    [self.view addSubview:imageView];
}
```

效果如下图所示：

![](http://static.oschina.net/uploads/space/2016/1015/190118_LISd_2340880.png)

与操作PDF图形上下文的相关方法如下：

```objectivec
//这个方法用于创建一个PDF图形上下文 将其入栈 作为当前的图形上下文  
/*
其中path为PDF文件写入的路径
bounds为PDF文档的尺寸
decumentInfo地点为设置PDF文档信息 后面会介绍
*/
UIKIT_EXTERN BOOL UIGraphicsBeginPDFContextToFile(NSString *path, CGRect bounds, NSDictionary * __nullable documentInfo) NS_AVAILABLE_IOS(3_2);
//这个方法用于穿件一个PDF图形上下文 但是将PDF内容写成Data数据 参数意义同上
UIKIT_EXTERN void UIGraphicsBeginPDFContextToData(NSMutableData *data, CGRect bounds, NSDictionary * __nullable documentInfo) NS_AVAILABLE_IOS(3_2);
//结束PDF图形上下文的编辑 将其出栈
UIKIT_EXTERN void UIGraphicsEndPDFContext(void) NS_AVAILABLE_IOS(3_2);
//这个方法用于将当前的PDF图形上下文新开一页内容
UIKIT_EXTERN void UIGraphicsBeginPDFPage(void) NS_AVAILABLE_IOS(3_2);
//同上
UIKIT_EXTERN void UIGraphicsBeginPDFPageWithInfo(CGRect bounds, NSDictionary * __nullable pageInfo) NS_AVAILABLE_IOS(3_2);
//返回当前PDF图形上下文所在页的尺寸
UIKIT_EXTERN CGRect UIGraphicsGetPDFContextBounds(void) NS_AVAILABLE_IOS(3_2);
//向PDF文档中的某个区域添加链接
UIKIT_EXTERN void UIGraphicsSetPDFContextURLForRect(NSURL *url, CGRect rect) NS_AVAILABLE_IOS(3_2);
//向PDF文档中的某个区域添加一个跳转目标 使其滚动到某点
UIKIT_EXTERN void UIGraphicsAddPDFContextDestinationAtPoint(NSString *name, CGPoint point) NS_AVAILABLE_IOS(3_2);
//向PDF文档中的某个区域添加一个跳转目标 使其滚动到某个区域
UIKIT_EXTERN void UIGraphicsSetPDFContextDestinationForRect(NSString *name, CGRect rect) NS_AVAILABLE_IOS(3_2);
```

上面有提到，在创建PDF图形上下文时，可以设置一个信息字典，这个字典中常用的可以进行配置的键值如下：

```objectivec
//这个键是可选的 对应需要设置为字符串类型的值 表明文档作者
kCGPDFContextAuthor
//这个键是可选的 对应需要设置为字符串类型的值 表示生成文档的命名名称
kCGPDFContextCreator
//这个键是可选的 对应需要设置为字符串类型的值 表示文档名称
kCGPDFContextTitle
//这个键设置所有者密码 需要设置为CFString的值
kCGPDFContextOwnerPassword
//这个键设置用户密码 需要设置为CFString的值
kCGPDFContextUserPassword
//这个键设置是否允许在未解锁状态下进行打印 需要设置为CFBollean的值 默认为允许
kCGPDFContextAllowsPrinting
//这个键设置是否允许在未解锁状态下进行复制 需要设置为CFBollean的值 默认为允许
kCGPDFContextAllowsCopying
//设置输出规范
kCGPDFContextOutputIntent
kCGPDFContextOutputIntents
//设置文档的主题 需要设置为CFString的值
kCGPDFContextSubject
//设置文档的关键字
kCGPDFContextKeywords
//设置密钥长度
kCGPDFContextEncryptionKeyLength
```

### 四、CGContext功能解析

    前边介绍了如何拿到对应的图形上下文，拿到图形上下文后，开发者便可以随心所欲的通过图形上下文向目标上绘制内容。CoreGraphics框架中提供的CGContext绘制相关方法解析如下：

```objectivec
//获取CGContext类在CoreGraphics框架中的id值
CFTypeID CGContextGetTypeID(void);
//将当前图形上下文进行保存 会执行push入栈
void CGContextSaveGState(CGContextRef cg_nullable c);
//将图形上下文恢复到保存时的状态 
void CGContextRestoreGState(CGContextRef cg_nullable c);
//对context内容进行缩放操作
void CGContextScaleCTM(CGContextRef cg_nullable c, CGFloat sx, CGFloat sy);
//对context内容进行平移操作
void CGContextTranslateCTM(CGContextRef cg_nullable c, CGFloat tx, CGFloat ty);
//对context内容进行旋转操作
void CGContextRotateCTM(CGContextRef cg_nullable c, CGFloat angle);
//对context内容进行transform变换操作
void CGContextConcatCTM(CGContextRef cg_nullable c,CGAffineTransform transform);
//获取某个context的transform变换对象
CGAffineTransform CGContextGetCTM(CGContextRef cg_nullable c);
//设置绘制的线宽
void CGContextSetLineWidth(CGContextRef cg_nullable c, CGFloat width);
//设置绘制的线帽风格
void CGContextSetLineCap(CGContextRef cg_nullable c, CGLineCap cap);
//设置绘制的线连接处风格
void CGContextSetLineJoin(CGContextRef cg_nullable c, CGLineJoin join);
//设置绘制的先转折处风格
void CGContextSetMiterLimit(CGContextRef cg_nullable c, CGFloat limit);
//设置虚线配置参数
void CGContextSetLineDash(CGContextRef cg_nullable c, CGFloat phase, const CGFloat * __nullable lengths, size_t count);
//设置平滑度
void CGContextSetFlatness(CGContextRef cg_nullable c, CGFloat flatness);
//设置透明度
void CGContextSetAlpha(CGContextRef cg_nullable c, CGFloat alpha);
//设置图像复合模式
void CGContextSetBlendMode(CGContextRef cg_nullable c, CGBlendMode mode);
//开始新的路径 旧的路径将被抛弃
void CGContextBeginPath(CGContextRef cg_nullable c);
//将路径起点移动到某个点
void CGContextMoveToPoint(CGContextRef cg_nullable c, CGFloat x, CGFloat y);
//向路径中添加一条线
void CGContextAddLineToPoint(CGContextRef cg_nullable c,CGFloat x, CGFloat y);
//向路径中添加三次贝塞尔曲线
void CGContextAddCurveToPoint(CGContextRef cg_nullable c, CGFloat cp1x, CGFloat cp1y, CGFloat cp2x, CGFloat cp2y, CGFloat x, CGFloat y);
//向路径中添加二次贝塞尔曲线
void CGContextAddQuadCurveToPoint(CGContextRef cg_nullable c, CGFloat cpx, CGFloat cpy, CGFloat x, CGFloat y);
//闭合路径
void CGContextClosePath(CGContextRef cg_nullable c);
//向路径中添加一个矩形
void CGContextAddRect(CGContextRef cg_nullable c, CGRect rect);
//向路径中添加一组矩形
void CGContextAddRects(CGContextRef cg_nullable c, const CGRect * __nullable rects, size_t count);
//向路径中添加一组线条
void CGContextAddLines(CGContextRef cg_nullable c, const CGPoint * __nullable points, size_t count);
//向路径中添加椭圆
CGContextAddEllipseInRect(CGContextRef cg_nullable c, CGRect rect);
//向路径中添加圆弧
void CGContextAddArc(CGContextRef cg_nullable c, CGFloat x, CGFloat y, CGFloat radius, CGFloat startAngle, CGFloat endAngle, int clockwise);
void CGContextAddArcToPoint(CGContextRef cg_nullable c, CGFloat x1, CGFloat y1, CGFloat x2, CGFloat y2, CGFloat radius);
//直接向上下文中添加一个路径对象
void CGContextAddPath(CGContextRef cg_nullable c, CGPathRef cg_nullable path);
//将上下文中的路径内容替换掉 只留下边框
void CGContextReplacePathWithStrokedPath(CGContextRef cg_nullable c);
//判断某个Context的路径是否为空
bool CGContextIsPathEmpty(CGContextRef cg_nullable c);
//获取一个Context路径当前端点的位置
CGPoint CGContextGetPathCurrentPoint(CGContextRef cg_nullable c);
//获取路径的尺寸
CGRect CGContextGetPathBoundingBox(CGContextRef cg_nullable c);
//进行图形上下文的拷贝
CGPathRef __nullable CGContextCopyPath(CGContextRef cg_nullable c);
//获取context的路径中是否包含某个点
bool CGContextPathContainsPoint(CGContextRef cg_nullable c, CGPoint point, CGPathDrawingMode mode);
//进行路径的绘制
/*
mode枚举意义如下：
  kCGPathFill,    //进行填充
  kCGPathEOFill,   //补集进行填充绘制
  kCGPathStroke,  //边框绘制
  kCGPathFillStroke, //边框绘制并填充
  kCGPathEOFillStroke //补集进行边框和填充绘制
*/
void CGContextDrawPath(CGContextRef cg_nullable c, CGPathDrawingMode mode)；
//进行路径的填充
void CGContextFillPath(CGContextRef cg_nullable c);
//进行路径所围成区域的补集区域填充
void CGContextEOFillPath(CGContextRef cg_nullable c);
//进行边框绘制
void CGContextStrokePath(CGContextRef cg_nullable c);
//填充某个矩形区域
void CGContextFillRect(CGContextRef cg_nullable c, CGRect rect);
//填充一组矩形区域
void CGContextFillRects(CGContextRef cg_nullable c, const CGRect * __nullable rects, size_t count);
//进行矩形区域的边框绘制
void CGContextStrokeRect(CGContextRef cg_nullable c, CGRect rect);
//进行矩形区域的边框绘制 可以设置边框宽度
void CGContextStrokeRectWithWidth(CGContextRef cg_nullable c, CGRect rect, CGFloat width);
//清除某个矩形区域
void CGContextClearRect(CGContextRef cg_nullable c, CGRect rect);
//进行虚线区域的填充
void CGContextFillEllipseInRect(CGContextRef cg_nullable c, CGRect rect);
//进行虚线区域边框的绘制
void CGContextStrokeEllipseInRect(CGContextRef cg_nullable c,CGRect rect);
//绘制一组线
void CGContextStrokeLineSegments(CGContextRef cg_nullable c, const CGPoint * __nullable points, size_t count);
//依据Context当前路径进行裁剪
void CGContextClip(CGContextRef cg_nullable c);
//进行路径区域的补集区域裁剪
void CGContextEOClip(CGContextRef cg_nullable c);
//这个方法十分重要 其可以将图片裁剪成图形上下文定义的形状
void CGContextClipToMask(CGContextRef cg_nullable c, CGRect rect, CGImageRef cg_nullable mask);
//获取裁剪的区域尺寸
CGRect CGContextGetClipBoundingBox(CGContextRef cg_nullable c);
//进行区域裁剪
void CGContextClipToRect(CGContextRef cg_nullable c, CGRect rect);
//进行一组区域的裁剪
void CGContextClipToRects(CGContextRef cg_nullable c, const CGRect *  rects, size_t count);
//设置图形上下文的填充颜色
void CGContextSetFillColorWithColor(CGContextRef cg_nullable c, CGColorRef cg_nullable color);
//设置图形上下文的边框颜色
void CGContextSetStrokeColorWithColor(CGContextRef cg_nullable c, CGColorRef cg_nullable color);
//设置图形上下文填充颜色的色彩空间
void CGContextSetFillColorSpace(CGContextRef cg_nullable c,CGColorSpaceRef cg_nullable space);
//设置图形上下文边框颜色的色彩空间
void CGContextSetStrokeColorSpace(CGContextRef cg_nullable c, CGColorSpaceRef cg_nullable space);
//下面这些函数与设置颜色和组件模块属性相关
void CGContextSetFillColor(CGContextRef cg_nullable c, const CGFloat * cg_nullable components);
void CGContextSetStrokeColor(CGContextRef cg_nullable c, const CGFloat * cg_nullable components);
void CGContextSetFillPattern(CGContextRef cg_nullable c, CGPatternRef cg_nullable pattern, const CGFloat * cg_nullable components);
void CGContextSetStrokePattern(CGContextRef cg_nullable c, CGPatternRef cg_nullable pattern, const CGFloat * cg_nullable components);
void CGContextSetPatternPhase(CGContextRef cg_nullable c, CGSize phase);
void CGContextSetGrayFillColor(CGContextRef cg_nullable c, CGFloat gray, CGFloat alpha);
void CGContextSetGrayStrokeColor(CGContextRef cg_nullable c, CGFloat gray, CGFloat alpha);
void CGContextSetRGBFillColor(CGContextRef cg_nullable c, CGFloat red, CGFloat green, CGFloat blue, CGFloat alpha);
void CGContextSetRGBStrokeColor(CGContextRef cg_nullable c, CGFloat red, CGFloat green, CGFloat blue, CGFloat alpha);
void CGContextSetCMYKFillColor(CGContextRef cg_nullable c, CGFloat cyan, CGFloat magenta, CGFloat yellow, CGFloat black, CGFloat alpha);
void CGContextSetCMYKStrokeColor(CGContextRef cg_nullable c, CGFloat cyan, CGFloat magenta, CGFloat yellow, CGFloat black, CGFloat alpha);
//将当前上下文内容渲染进颜色intent中
void CGContextSetRenderingIntent(CGContextRef cg_nullable c, CGColorRenderingIntent intent);
//在指定区域内渲染图片
void CGContextDrawImage(CGContextRef cg_nullable c, CGRect rect, CGImageRef cg_nullable image);
//在区域内进行瓦片方式的图片渲染
void CGContextDrawTiledImage(CGContextRef cg_nullable c, CGRect rect, CGImageRef cg_nullable image);
//获取上下文渲染的图像质量
CGInterpolationQuality CGContextGetInterpolationQuality(CGContextRef cg_nullable c);
//设置上下文渲染时的图像质量
void CGContextSetInterpolationQuality(CGContextRef cg_nullable c, CGInterpolationQuality quality);
//设置进行阴影的渲染
void CGContextSetShadowWithColor(CGContextRef cg_nullable c,  CGSize offset, CGFloat blur, CGColorRef __nullable color);
void CGContextSetShadow(CGContextRef cg_nullable c, CGSize offset, CGFloat blur);
//绘制线性渐变效果
void CGContextDrawLinearGradient(CGContextRef cg_nullable c,  CGGradientRef cg_nullable gradient, CGPoint startPoint, CGPoint endPoint, CGGradientDrawingOptions options);
//绘制半径渐变效果
void CGContextDrawRadialGradient(CGContextRef cg_nullable c, CGGradientRef cg_nullable gradient, CGPoint startCenter, CGFloat startRadius, CGPoint endCenter, CGFloat endRadius, CGGradientDrawingOptions options);
//用渐变填充上下文的裁剪区域
void CGContextDrawShading(CGContextRef cg_nullable c,  cg_nullable CGShadingRef shading);
//设置绘制的文字间距
void CGContextSetCharacterSpacing(CGContextRef cg_nullable c, CGFloat spacing);
//设置绘制的文字位置
void CGContextSetTextPosition(CGContextRef cg_nullable c, CGFloat x, CGFloat y);
//获取绘制的文字位置
CGPoint CGContextGetTextPosition(CGContextRef cg_nullable c);
//设置文字transform变换
void CGContextSetTextMatrix(CGContextRef cg_nullable c, CGAffineTransform t);
//获取文字的transform变换
CGAffineTransform CGContextGetTextMatrix(CGContextRef cg_nullable c);
//设置文字的绘制模式
/*
  kCGTextFill,           //填充
  kCGTextStroke,         //空心 
  kCGTextFillStroke,     //填充加边框
  kCGTextInvisible,      //在可是区域内
  kCGTextFillClip,       //裁剪填充
  kCGTextStrokeClip,     //裁剪绘制边框
  kCGTextFillStrokeClip,//进行裁剪
  kCGTextClip
*/
void CGContextSetTextDrawingMode(CGContextRef cg_nullable c, CGTextDrawingMode mode);
//设置绘制文字的字体
void CGContextSetFont(CGContextRef cg_nullable c, CGFontRef cg_nullable font);
//设置绘制文字的字号
void CGContextSetFontSize(CGContextRef cg_nullable c, CGFloat size);
void CGContextShowGlyphsAtPositions(CGContextRef cg_nullable c, const CGGlyph * cg_nullable glyphs, const CGPoint * cg_nullable Lpositions,  size_t count);
//设置绘制的字符风格
void CGContextShowGlyphsAtPositions(CGContextRef cg_nullable c, const CGGlyph * cg_nullable glyphs, const CGPoint * cg_nullable Lpositions, size_t count);
//进行PDF页的绘制
void CGContextDrawPDFPage(CGContextRef cg_nullable c, CGPDFPageRef cg_nullable page);
//开启一个新的PDF页
void CGContextBeginPage(CGContextRef cg_nullable c,  const CGRect * __nullable mediaBox);
//结束当前的PDF页
void CGContextEndPage(CGContextRef cg_nullable c);
//内存引用加1
CGContextRef cg_nullable CGContextRetain(CGContextRef cg_nullable c);
//内存引用计数减1
void CGContextRelease(CGContextRef cg_nullable c);
//将上下文中的内容立即渲染到目标
void CGContextFlush(CGContextRef cg_nullable c);
//将上下文中的内容进行同步
void CGContextSynchronize(CGContextRef cg_nullable c);
//是否开启抗锯齿效果
void CGContextSetShouldAntialias(CGContextRef cg_nullable c,  bool shouldAntialias);
//是否允许抗锯齿效果
void CGContextSetAllowsAntialiasing(CGContextRef cg_nullable c,  bool allowsAntialiasing);
//是否开启字体平滑
void CGContextSetShouldSmoothFonts(CGContextRef cg_nullable c, bool shouldSmoothFonts);
//是否允许字体平滑
void CGContextSetAllowsFontSmoothing(CGContextRef cg_nullable c, bool allowsFontSmoothing);
//设置是否开启subpixel状态渲染符号
void CGContextSetShouldSubpixelPositionFonts(CGContextRef cg_nullable c, bool shouldSubpixelPositionFonts);
//是否允许subpixel状态渲染符号
void CGContextSetAllowsFontSubpixelPositioning(CGContextRef cg_nullable c, bool allowsFontSubpixelPositioning);
//这个方法会在当前Context中开启一个透明的层 之后的绘制会绘制到这个透明的层上
void CGContextBeginTransparencyLayer(CGContextRef cg_nullable c, CFDictionaryRef __nullable auxiliaryInfo);
//在Context中开启一个透明的层
void CGContextBeginTransparencyLayerWithRect(CGContextRef cg_nullable c, CGRect rect, CFDictionaryRef __nullable auxInfo);
//完成透明层的渲染
void CGContextEndTransparencyLayer(CGContextRef cg_nullable c);
//返回用户控件的transform变换
CGAffineTransform CGContextGetUserSpaceToDeviceSpaceTransform(CGContextRef cg_nullable c);
//将用户控件点的坐标转换为设备控件坐标
CGPoint CGContextConvertPointToDeviceSpace(CGContextRef cg_nullable c, CGPoint point);
//将设备空间的点坐标转换为用户空间的点坐标
CGPoint CGContextConvertPointToUserSpace(CGContextRef cg_nullable c, CGPoint point);
//将用于空间的尺寸转换为设备空间的尺寸
CGSize CGContextConvertSizeToDeviceSpace(CGContextRef cg_nullable c, CGSize size);
//将设备空间的尺寸转换为用户空间的尺寸
CGSize CGContextConvertSizeToUserSpace(CGContextRef cg_nullable c, CGSize size);
//将用户空间的rect转换为设备空间的rect
CGRect CGContextConvertRectToDeviceSpace(CGContextRef cg_nullable c, CGRect rect);
//将设备空间的rect转换为用户空间的rect
CGRect CGContextConvertRectToUserSpace(CGContextRef cg_nullable c, CGRect rect);

//下面这些方法已经被弃用 
//设置字体 使用CoreText相关的API代替
void CGContextSelectFont(CGContextRef cg_nullable c, const char * cg_nullable name, CGFloat size, CGTextEncoding textEncoding);
//绘制文本 使用CoreText相关API代替
void CGContextShowText(CGContextRef cg_nullable c, const char * cg_nullable string, size_t length);
//在相应位置绘制文本 使用CoreText相关API代替
void CGContextShowTextAtPoint(CGContextRef cg_nullable c, CGFloat x, CGFloat y, const char * cg_nullable string, size_t length);
//进行符号的绘制 使用CoreText相关API代替
void CGContextShowGlyphs(CGContextRef cg_nullable c, const CGGlyph * __nullable g, size_t count);
//在相应位置绘制符号 使用CoreText相关API代替
void CGContextShowGlyphsAtPoint(CGContextRef cg_nullable c, CGFloat x, CGFloat y, const CGGlyph * __nullable glyphs, size_t count);
//绘制符号 使用一个固定的缩进值 使用CoreText相关API代替
void CGContextShowGlyphsWithAdvances(CGContextRef cg_nullable c, const CGGlyph * __nullable glyphs, const CGSize * __nullable advances, size_t count);
//进行PDF文档绘制 CGPDFPage相关API代替
void CGContextDrawPDFDocument(CGContextRef cg_nullable c, CGRect rect, CGPDFDocumentRef cg_nullable document, int page);
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

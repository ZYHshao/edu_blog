---
title: iOS开发之CoreImage框架使用
date: 2018-12-22
categories: iOS逻辑初窥
tags: []
---
## iOS开发之CoreImage框架使用

      CoreImage框架是一个专门用来对图片进行处理的框架，其中提供了许多高级功能，可以帮助开发者完成UIKit或者CoreGraphics框架无法完成的任务，并且使用CoreImage框架可以十分轻松的实现滤镜以及图像识别等流行技术。本篇博客主要介绍和总结CoreImage框架的使用，并提供范例代码。

### 一、图像过滤器

#### 1.几组内置的过滤器

    CIFilter是CoreImage中提供的图像过滤器，也可以将其理解为滤镜。许多美颜应用，图像处理应用等都是为原图添加了滤镜效果。本节我们着重看下与这个类相关的应用。首先，CoreImaghe默认提供了非常多的滤镜效果，但是并没有详细的文档介绍，有关滤镜效果可以分为下面几个类别：

```objectivec
//通过改变图像的几何形状来创建3D效果，类似隆起 过滤器组
NSString * const kCICategoryDistortionEffect;
//旋转扭曲相关过滤器组
NSString * const kCICategoryGeometryAdjustment;
//混合过滤器组 对两个图像进行混合操作
NSString * const kCICategoryCompositeOperation;
//一种色调过滤器组 类似报纸风格
NSString * const kCICategoryHalftoneEffect;
//颜色过滤器组 调整对比度 亮度等
NSString * const kCICategoryColorEffect;
//多个图像源的过滤器
NSString * const kCICategoryTransition;
//平铺图像过滤器
NSString * const kCICategoryTileEffect;
//滤光类过滤器 通常作为其他过滤器的输入
NSString * const kCICategoryGenerator;
//减弱图像数据的过滤器 通常用来进行图像分析
NSString * const kCICategoryReduction;
//渐变过滤器 
NSString * const kCICategoryGradient;
//画像过滤器
NSString * const kCICategoryStylize;
//锐化过滤器
NSString * const kCICategorySharpen;
//模糊过滤器
NSString * const kCICategoryBlur;
//视频图片相关过滤器
NSString * const kCICategoryVideo;
//静态图片相关过滤器
NSString * const kCICategoryStillImage;
//交叉图像过滤器
NSString * const kCICategoryInterlaced;
//非矩形图像上的过滤器
NSString * const kCICategoryNonSquarePixels;
//高动态图像的过滤器
NSString * const kCICategoryHighDynamicRange;
//CoreImage内置的过滤器
NSString * const kCICategoryBuiltIn;
//复合的过滤器
NSString * const kCICategoryFilterGenerator;
```

上面列出了非常多的类别，其实上面只是按照不同的场景将过滤器进行了分类，每个分类中都定义了许多内置的过滤器，使用下面的方法可以获取每个分类下提供的过滤器：

```objectivec
//获取某个分类的所有过滤器名
+ (NSArray<NSString *> *)filterNamesInCategory:(nullable NSString *)category;
//获取一组分类下的所有过滤器名
+ (NSArray<NSString *> *)filterNamesInCategories:(nullable NSArray<NSString *> *)categories;
```

#### 2.过滤器的一个简单示例

下面示例代码演示过滤器的简单应用：

```objectivec
UIImage * img = [UIImage imageNamed:@"1.png"];
CIImage * image = [[CIImage alloc]initWithImage:img];
CIFilter * filter = [CIFilter filterWithName:@"CIBoxBlur" keysAndValues:kCIInputImageKey,image, nil];
[filter setDefaults];
CIContext * context = [[CIContext alloc]initWithOptions:nil];
CIImage * output = [filter outputImage];
CGImageRef ref = [context createCGImage:output fromRect:[output extent]];
UIImage * newImage = [UIImage imageWithCGImage:ref];
CGImageRelease(ref);
UIImageView * imageView = [[UIImageView alloc]initWithFrame:CGRectMake(170, 30, 150, 400)];
imageView.image = newImage;
[self.view addSubview:imageView]; 
UIImageView * imageView2 = [[UIImageView alloc]initWithFrame:CGRectMake(0, 30, 150, 400)];
imageView2.image = img;
[self.view addSubview:imageView2];
```

效果如下图：

![](https://oscimg.oschina.net/oscnet/24a8db1b3c214669877769cdabb603b027a.jpg)

上面演示了简单的模糊过滤效果。

#### 3.对CIFilter类进行解析

CIFilter类的解析如下：

```objectivec
//过滤后输出的图像
@property (readonly, nonatomic, nullable) CIImage *outputImage;
//过滤器名称
@property (nonatomic, copy) NSString *name;
//是否开启CoreAnimation动画效果
@property (getter=isEnabled) BOOL enabled;
//返回当前过滤器所有支持的输入键
@property (nonatomic, readonly) NSArray<NSString *> *inputKeys;
//返回当前过滤器所有支持的输出键
@property (nonatomic, readonly) NSArray<NSString *> *outputKeys;
//将过滤器的所有输入值设置为默认值
- (void)setDefaults;
//返回当前过滤器的属性字段
/*
需要注意这个字段对于学习此过滤器非常有用
其中会声明此过滤器的输入和输出 即如果使用
*/
@property (nonatomic, readonly) NSDictionary<NSString *,id> *attributes;
//用来进行过滤器的自定义 后面会介绍
- (nullable CIImage *)apply:(CIKernel *)k
                  arguments:(nullable NSArray *)args
                    options:(nullable NSDictionary<NSString *,id> *)dict;
//同上
- (nullable CIImage *)apply:(CIKernel *)k, ...;
//根据过滤器的名称创建过滤器
+ (nullable CIFilter *) filterWithName:(NSString *) name;
//创建过滤器 同时进行配置
+ (nullable CIFilter *)filterWithName:(NSString *)name
                        keysAndValues:key0, ...;
+ (nullable CIFilter *)filterWithName:(NSString *)name
                  withInputParameters:(nullable NSDictionary<NSString *,id> *)params;
//注册过滤器
+ (void)registerFilterName:(NSString *)name
               constructor:(id<CIFilterConstructor>)anObject
           classAttributes:(NSDictionary<NSString *,id> *)attributes;
//将一组过滤器进行编码
+ (nullable NSData*)serializedXMPFromFilters:(NSArray<CIFilter *> *)filters
                            inputImageExtent:(CGRect)extent;
//进行反编码
+ (NSArray<CIFilter *> *)filterArrayFromSerializedXMP:(NSData *)xmpData
                                     inputImageExtent:(CGRect)extent
                                                error:(NSError **)outError;
```

#### 4.常用过滤器详解

-   区域凸起过滤器

这个过滤器的作用是在图片的某个区域创建一块凸起。示例代码如下：

```objectivec
/*
kCIInputCenterKey键用来设置滤镜中心
kCIInputScaleKey 设置为0则没有影响 1则会凸起效果 -1则会凹入效果
kCIInputRadiusKey 设置滤镜的影响范围
*/
CIFilter * filter = [CIFilter filterWithName:@"CIBumpDistortion" keysAndValues:kCIInputImageKey,image,kCIInputCenterKey,[[CIVector alloc] initWithX:100 Y:200],kCIInputScaleKey,@-1,kCIInputRadiusKey,@150, nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/430f348114806e1695db9eab491edaa6c39.jpg)

-   线性凹凸过滤器

这个过滤器创建类似波纹效果，示例如下：

```objectivec
/*
与上一个过滤器相比 可以设置
kCIInputAngleKey 角度 0-2π
*/
CIFilter * filter = [CIFilter filterWithName:@"CIBumpDistortionLinear" keysAndValues:kCIInputImageKey,image,kCIInputCenterKey,[[CIVector alloc] initWithX:100 Y:200],kCIInputScaleKey,@-1,kCIInputRadiusKey,@150,kCIInputAngleKey,@(M_PI_2), nil];
```

效果如下：

![](https://oscimg.oschina.net/oscnet/11c0b1faa1460b7fa23f7bd38671c94f68f.jpg)

-   圆形飞溅过滤器

这个过滤器的作用是选取图像的某个区域，对其四周进行飞溅拉伸，例如：

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CICircleSplashDistortion" keysAndValues:kCIInputImageKey,image,kCIInputCenterKey,[[CIVector alloc] initWithX:100 Y:200],kCIInputRadiusKey,@50, nil];
```

效果如下：

![](https://oscimg.oschina.net/oscnet/723a5f106f3076d972c413426b7bd46d58a.jpg)

-   圆形缠绕过滤器

这个过滤器选取某个区域，进行缠绕效果，例如：

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CICircularWrap" keysAndValues:kCIInputImageKey,image,kCIInputCenterKey,[[CIVector alloc] initWithX:100 Y:200],kCIInputRadiusKey,@20, kCIInputAngleKey,@3,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/6469711af268c7b0cc0332822eb581ef328.jpg)

-   灰度混合过滤器

这个过滤器将提供混合图像的灰度值应用于目标图像，例如：

```objectivec
/*
inputDisplacementImage设置要混合的灰度图片
*/
CIFilter * filter = [CIFilter filterWithName:@"CIDroste" keysAndValues:kCIInputImageKey,image,kCIInputScaleKey,@200,@"inputDisplacementImage",image2,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/15e862dc1e9dd634ece51d8927aa9034904.jpg)

-   递归绘制图像区域

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIDroste" keysAndValues:kCIInputImageKey,image,@"inputInsetPoint0",[[CIVector alloc] initWithX:100 Y:100],@"inputInsetPoint1",[[CIVector alloc] initWithX:200 Y:200],@"inputPeriodicity",@1,@"inputRotation",@0,@"inputStrands",@1,@"inputZoom",@1,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/0e7d06ce9442acb6a8054fc6a9824f51109.jpg)

-   玻璃纹理过滤器

这个过滤器用提供图片作为目标图片的纹理，进行混合，例如：

```objectivec
/*
inputTexture设置纹理图像
*/
CIFilter * filter = [CIFilter filterWithName:@"CIGlassDistortion" keysAndValues:kCIInputImageKey,image,kCIInputCenterKey,[[CIVector alloc] initWithX:100 Y:200],kCIInputScaleKey,@100,@"inputTexture",image2,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/52640114163381a7a52674f5f56c852ac5e.jpg)

-   菱形透镜过滤器

```objectivec
/*
inputPoint0设置第一个圆的圆心
inputPoint1设置第二个圆的圆心
inputRadius设置半径
inputRefraction设置折射率 0-5之间
*/
CIFilter * filter = [CIFilter filterWithName:@"CIGlassLozenge" keysAndValues:kCIInputImageKey,image,@"inputPoint0",[[CIVector alloc] initWithX:100 Y:200],@"inputPoint1",[[CIVector alloc] initWithX:200 Y:200],@"inputRadius",@100,@"inputRefraction",@2,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/035b1ce74a68064dcafc10ac0fe54f70822.jpg)

-   圆孔形变过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIHoleDistortion" keysAndValues:kCIInputImageKey,image,@"inputRadius",@50,kCIInputCenterKey,[[CIVector alloc] initWithX:100 Y:200],nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/e06fad2bc55bdae87c2fe83faef6f47e477.jpg)

-   九宫格拉伸过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CINinePartStretched" keysAndValues:kCIInputImageKey,image2,@"inputBreakpoint0",[[CIVector alloc] initWithX:50 Y:50],@"inputBreakpoint1",[[CIVector alloc] initWithX:100 Y:100],@"inputGrowAmount",[[CIVector alloc] initWithX:50 Y:50],nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/33f7ef97e689a9edb72d8f2c4407ef01c8d.jpg)

-   九宫格复制过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CINinePartTiled" keysAndValues:kCIInputImageKey,image2,@"inputBreakpoint0",[[CIVector alloc] initWithX:50 Y:50],@"inputBreakpoint1",[[CIVector alloc] initWithX:100 Y:100],@"inputGrowAmount",[[CIVector alloc] initWithX:50 Y:50],@"inputFlipYTiles",@1,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/663837a7cf141a3575571b8a7a12cf258ad.jpg)

-   紧缩过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIPinchDistortion" keysAndValues:kCIInputImageKey,image2,@"inputCenter",[[CIVector alloc] initWithX:150 Y:150],@"inputRadius",@500,@"inputScale",@1,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/cd6ce1d87b78902a52c45eed6c660299c12.jpg)

-   拉伸裁剪过滤器

```objectivec
/*
inputSize 设置拉伸裁剪尺寸
*/
CIFilter * filter = [CIFilter filterWithName:@"CIStretchCrop" keysAndValues:kCIInputImageKey,image2,@"inputCenterStretchAmount",@1,@"inputCropAmount",@0.5,@"inputSize",[[CIVector alloc] initWithX:300 Y:150],nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/2f656fd5b7e832f50632fbaeeb4f3529f98.jpg)

-   环状透镜过滤器

这个过滤器创建一个环状透镜，对图像进行扭曲。

```objectivec
/*
inputCenter设置环中心
inputRadius 设置半径
inputRefraction 设置折射率
inputWidth 设置环宽度
*/
CIFilter * filter = [CIFilter filterWithName:@"CITorusLensDistortion" keysAndValues:kCIInputImageKey,image2,@"inputCenter",[[CIVector alloc] initWithX:150 Y:150],@"inputRadius",@150,@"inputRefraction",@1.6,@"inputWidth",@40,nil];
```

效果如下：

![](https://oscimg.oschina.net/oscnet/3feab550ad80ff64dbece2dca4631628aad.jpg)

-   旋转过滤器 

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CITwirlDistortion" keysAndValues:kCIInputImageKey,image2,@"inputAngle",@3.14,@"inputCenter",[[CIVector alloc] initWithX:150 Y:150],@"inputRadius",@150,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/30dccc1198526d164c14530aca78136d6c8.jpg)

-   涡流过滤器

```objectivec
//inputAngle 设置涡流角度
CIFilter * filter = [CIFilter filterWithName:@"CIVortexDistortion" keysAndValues:kCIInputImageKey,image2,@"inputAngle",@(M_PI*10),@"inputCenter",[[CIVector alloc] initWithX:150 Y:150],@"inputRadius",@150,nil];
```

效果如下：

![](https://oscimg.oschina.net/oscnet/10efe45dd980fd18ea549b51e76244e7c4e.jpg)

-   形变过滤器 

这个过滤器对图像进行简单的形变处理，如缩放，旋转，平移等。

```objectivec
CGAffineTransform tr =  CGAffineTransformMakeRotation(M_PI_2);
CIFilter * filter = [CIFilter filterWithName:@"CIAffineTransform" keysAndValues:kCIInputImageKey,image2,@"inputTransform",[NSValue valueWithCGAffineTransform:tr],nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/b4491f96dd5c880f263ffccc2f7a3ab8f3c.jpg)

-   矩形裁剪过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CICrop" keysAndValues:kCIInputImageKey,image2,@"inputRectangle",[[CIVector alloc] initWithCGRect:CGRectMake(0, 0, 150, 150)],nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/f48c0ceea4840853c795bda25d745a22bcc.jpg)

-   边缘采样过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIEdgePreserveUpsampleFilter" keysAndValues:kCIInputImageKey,image,@"inputLumaSigma",@0.15,@"inputSpatialSigma",@3,@"inputSmallImage",image2,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/bbe9f1b404c8511dea4c9c95cdce4425878.jpg)

-   矩形矫正过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIPerspectiveCorrection" keysAndValues:kCIInputImageKey,image2,@"inputBottomLeft",[[CIVector alloc] initWithX:0 Y:0],@"inputBottomRight",[[CIVector alloc] initWithX:150 Y:0],@"inputTopLeft",[[CIVector alloc] initWithX:0 Y:150],@"inputTopRight",[[CIVector alloc] initWithX:150 Y:150],nil];
```

效果如图：

![](https://oscimg.oschina.net/oscnet/a2e7e71847b98fa79e48dcfdcc2781a4d04.jpg)

-   旋转矫正过滤器 

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIStraightenFilter" keysAndValues:kCIInputImageKey,image2,@"inputAngle",@3.14,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/bab76e7ff5cb4041de2fb7d1abb456e088a.jpg)

-   背景混合过滤器

通过提供一个图像作为背景与目标图像进行混合。

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIAdditionCompositing" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/5ba2059c7b7e350150632a9952cfc4ab4e7.jpg)

-   色彩混合过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIColorBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/990da84f712eb45272a0f7a716c100b6d56.jpg)

-   暗混合过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIColorBurnBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

效果如下：

![](https://oscimg.oschina.net/oscnet/a08c926020e9f29bed13530e76922fc607a.jpg)

-   亮混合过滤器 

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIColorDodgeBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 ![](https://oscimg.oschina.net/oscnet/d62aa2e9a12cbaa86fbe4823ad7564605e3.jpg)

-   暗选择混合模式过滤器

 这个过滤器将选择较暗的图像作为混合背景，例如：

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIDarkenBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

效果如下：

![](https://oscimg.oschina.net/oscnet/8ddb81444e325a7cc184bd5be7b88f11e80.jpg)

-   亮选择混合模式过滤器

这个过滤器将选择较亮的图像作为混合背景，例如：

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIDifferenceBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/0fc3b4d03c9081955dfb734c6cbb228e317.jpg)

-   分开混合模式过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIDivideBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

效果如下：

![](https://oscimg.oschina.net/oscnet/479f8631b6736a34a34120bda57266f5571.jpg)

-   排除混合模式过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIExclusionBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/fd82734a2f8197acfcf86521f61d6b1e6c2.jpg)

-   强光混合模式过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIHardLightBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/58fbef955a3b45d144d2f2e0c1ecc88c2cb.jpg)

-   色调混合模式过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIHueBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/aad818d99fff9ddb7bcb5f67e6560e5d93c.jpg)

-   减轻混合模式过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CILightenBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/511fa605c2413382ec0cbaa886821115943.jpg)

-   线性燃烧混合模式过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CILinearBurnBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

效果如下：

![](https://oscimg.oschina.net/oscnet/9700b3f3526fde848397c9ca1e727420955.jpg)

-   线性高亮混合过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CILinearDodgeBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/6b6d5a7144e09a95a1ddc76eee5b1d1a98d.jpg)

-   亮度混合过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CILuminosityBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/f97a80d79799e15c793aaa51090e65989d5.jpg)

-   最大值混合过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIMaximumCompositing" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/865ccf4a853e652d85286ccfee4d16579e0.jpg)

-   最小值混合过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIMinimumCompositing" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/d6d671fa23af7a40e5f560c29a94beaa80e.jpg)

-   多重混合过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIMultiplyBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 效果如下：

![](https://oscimg.oschina.net/oscnet/6887bbf2da33242cf1cd4b60ff2ec130d63.jpg)

-   多重合成过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIMultiplyCompositing" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 ![](https://oscimg.oschina.net/oscnet/71f27618d99a4d2aeca0fb182c2d56419e2.jpg)

-   重叠混合模式

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIOverlayBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 ![](https://oscimg.oschina.net/oscnet/aef3662c175fafd7d95b8955644799de6e7.jpg)

-   亮混合模式

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIPinLightBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 ![](https://oscimg.oschina.net/oscnet/3c9b3ffaf3915cf28b80b0e8130a05ba3ff.jpg)

-   饱和混合模式

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CISaturationBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 ![](https://oscimg.oschina.net/oscnet/14345282e936d9a08dc18accdc0aecec53b.jpg)

-   屏幕混合模式

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIScreenBlendMode" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 ![](https://oscimg.oschina.net/oscnet/c4dc4ea019ba061a1ccfd652ed53aa90c9b.jpg)

-   源图像上层混合

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CISourceAtopCompositing" keysAndValues:kCIInputImageKey,image2,@"inputBackgroundImage",image,nil];
```

 ![](https://oscimg.oschina.net/oscnet/f08b0417a4fdefb1e9b070f9d3b0135c90c.jpg)

-   圆屏过滤器

```objectivec
/*
inputSharpness 设置圆圈锐度
inputWidth 设置间距
*/
CIFilter * filter = [CIFilter filterWithName:@"CICircularScreen" keysAndValues:kCIInputImageKey,image2,kCIInputCenterKey,[[CIVector alloc] initWithX:150 Y:150],@"inputSharpness",@0.7,@"inputWidth",@6,nil];
```

![](https://oscimg.oschina.net/oscnet/ccfe5a4f9c96b2eb0981f4012ef08b9b968.jpg)

-   半色调过滤器 

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CICMYKHalftone" keysAndValues:kCIInputImageKey,image2,@"inputAngle",@0,kCIInputCenterKey,[[CIVector alloc] initWithX:150 Y:150],@"inputGCR",@1,@"inputSharpness",@0.7,@"inputUCR",@0.5,@"inputWidth",@6,nil];
```

![](https://oscimg.oschina.net/oscnet/8d9bc057e91eb41edcca47fce574840d965.jpg)

-   点屏过滤器 

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIDotScreen" keysAndValues:kCIInputImageKey,image2,@"inputAngle",@0,kCIInputCenterKey,[[CIVector alloc] initWithX:150 Y:150],@"inputSharpness",@0.7,@"inputWidth",@6,nil];
```

![](https://oscimg.oschina.net/oscnet/baac52760fdd6329512ea25a54444f14ead.jpg)

-   阴影屏过滤器 

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIHatchedScreen" keysAndValues:kCIInputImageKey,image2,@"inputAngle",@0,kCIInputCenterKey,[[CIVector alloc] initWithX:150 Y:150],@"inputSharpness",@0.7,@"inputWidth",@6,nil];
```

 ![](https://oscimg.oschina.net/oscnet/4635728f7fcd859306b919216c13b646507.jpg)

-   线性sRGB过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CILinearToSRGBToneCurve" keysAndValues:kCIInputImageKey,image2,nil];
```

 ![](https://oscimg.oschina.net/oscnet/f197e5e4486824b4a37ab3fb4c5b8c80594.jpg)

-   色彩翻转过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIColorInvert" keysAndValues:kCIInputImageKey,image2,nil];
```

 ![](https://oscimg.oschina.net/oscnet/ffb2c4665e8563e14378712d9246eda6a4b.jpg)

-   色图过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIColorMap" keysAndValues:kCIInputImageKey,image2,@"inputGradientImage",image,nil];
```

 ![](https://oscimg.oschina.net/oscnet/9d898d8531b0b819b1e3e08ba33ca7fcd6d.jpg)

-   单色过滤器

```objectivec
/*
inputColor 设置输入颜色
inputIntensity 设置影响程度
*/
CIFilter * filter = [CIFilter filterWithName:@"CIColorMonochrome" keysAndValues:kCIInputImageKey,image2,@"inputColor",[CIColor colorWithRed:0.5 green:0.5 blue:0.5],@"inputIntensity",@1,nil];
```

![](https://oscimg.oschina.net/oscnet/fc83e286588421a12868a80408f69fb1edc.jpg)

-   分色镜过滤器 

```objectivec
/*
inputLevels设置亮度级别
*/
CIFilter * filter = [CIFilter filterWithName:@"CIColorPosterize" keysAndValues:kCIInputImageKey,image2,@"inputLevels",@6,nil];
```

![](https://oscimg.oschina.net/oscnet/08ea763020d5f17475365d83b512fb23630.jpg)

-   反色过滤器 

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIFalseColor" keysAndValues:kCIInputImageKey,image2,@"inputColor0",[CIColor colorWithRed:0 green:0 blue:0],@"inputColor1",[CIColor colorWithRed:1 green:1 blue:0],nil];
```

 ![](https://oscimg.oschina.net/oscnet/850817a2d73d438bc2dca44a7236d74bb7e.jpg)

-   光效褪色过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIPhotoEffectFade" keysAndValues:kCIInputImageKey,image2,nil];
```

 ![](https://oscimg.oschina.net/oscnet/c25d2053059349e9a0a80e98295ad024948.jpg)

-   光效瞬时过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIPhotoEffectInstant" keysAndValues:kCIInputImageKey,image2,nil];
```

![](https://oscimg.oschina.net/oscnet/fe8a2f4b5a5c3b8d638641505bea297af86.jpg)

-   光效单光过滤器 

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIPhotoEffectMono" keysAndValues:kCIInputImageKey,image2,nil];
```

![](https://oscimg.oschina.net/oscnet/4194d887876c06b6eb807a9bef375c0c819.jpg)

-   黑色光效应过滤器 

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIPhotoEffectNoir" keysAndValues:kCIInputImageKey,image2,nil];
```

![](https://oscimg.oschina.net/oscnet/d7dfde1de08815d2f7752dae916128bd3e5.jpg) 

-   光渐进过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIPhotoEffectProcess" keysAndValues:kCIInputImageKey,image2,nil];
```

 ![](https://oscimg.oschina.net/oscnet/91181325364bceb0d195791abe37219d443.jpg)

-   光转移过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIPhotoEffectTransfer" keysAndValues:kCIInputImageKey,image2,nil];
```

 ![](https://oscimg.oschina.net/oscnet/025552bf6f9e6c752974621c707caf02969.jpg)

-   棕褐色过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CISepiaTone" keysAndValues:kCIInputImageKey,image2,nil];
```

![](https://oscimg.oschina.net/oscnet/dce03a2cb1b4486590bbe247ceab00176d3.jpg)

-   热图过滤器 

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIThermal" keysAndValues:kCIInputImageKey,image2,nil];
```

 ![](https://oscimg.oschina.net/oscnet/1cea9c31cbf44b07c99fb78ca9c8754db28.jpg)

-   X射线过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIXRay" keysAndValues:kCIInputImageKey,image2,nil];
```

 ![](https://oscimg.oschina.net/oscnet/83d19c0b012fa8850a4342f99c9b61d8ba0.jpg)

-   模糊过滤器

```objectivec
//参数进行模糊效果的设置
CIFilter * filter = [CIFilter filterWithName:@"CIBokehBlur" keysAndValues:kCIInputImageKey,image2,@"inputSoftness",@0.5,@"inputRingSize",@0.1,@"inputRingAmount",@0,@"inputRadius",@10,nil];
```

![](https://oscimg.oschina.net/oscnet/4b635c860ed84f339c3ec81443ae818d5f8.jpg)

-   盒模糊过滤器 

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIBoxBlur" keysAndValues:kCIInputImageKey,image2,@"inputRadius",@10,nil];
```

 ![](https://oscimg.oschina.net/oscnet/222129fbe8cdc5f8fe608bddf7323b992e3.jpg)

-   阀瓣模糊过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIDiscBlur" keysAndValues:kCIInputImageKey,image2,@"inputRadius",@25,nil];
```

 ![](https://oscimg.oschina.net/oscnet/8a07c22385ba4bb9285a2e152436ad58e1b.jpg)

-   高斯模糊过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIGaussianBlur" keysAndValues:kCIInputImageKey,image2,@"inputRadius",@10,nil];
```

 ![](https://oscimg.oschina.net/oscnet/1ec99843fe70181a3bf4e29e8264d7b74bc.jpg)

-   梯度模糊过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIMorphologyGradient" keysAndValues:kCIInputImageKey,image2,@"inputRadius",@5,nil];
```

 ![](https://oscimg.oschina.net/oscnet/8e844ccec3c5c6d2c4e3f3cf2f250a9ab25.jpg)

-   运动模糊过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIMotionBlur" keysAndValues:kCIInputImageKey,image2,@"inputRadius",@5,nil];
```

 ![](https://oscimg.oschina.net/oscnet/52d5736d529ee61992cd8c9b452ed661c07.jpg)

-   缩放模糊过滤器

```objectivec
CIFilter * filter = [CIFilter filterWithName:@"CIZoomBlur" keysAndValues:kCIInputImageKey,image2,nil];
```

 ![](https://oscimg.oschina.net/oscnet/667e5456ffdd5bc7b8ed15e26ebdf006ba9.jpg)

#### 5.自定义过滤器

    上面演示了非常多的常用内置过滤器，我们也可以通过继承CIFilter来自定义过滤器。

    自定义过滤器之前，首先需要了解CIKernel这个类，CIKernel是Core Image Kernel Language 的抽象对象。CIKL是CoreImage中专门用来编写像素处理函数的语言。

CIKernel相关类解析如下：

```objectivec
//基类 用于通用的过滤函数
@interface CIKernel : NSObject
//从字符串加载一组过滤函数
+ (nullable NSArray<CIKernel *> *)kernelsWithString:(NSString *)string;
//从字符串加载一个过滤函数
+ (nullable instancetype)kernelWithString:(NSString *)string ;
//名称
@property (atomic, readonly) NSString *name ;
//进行图片生成
- (nullable CIImage *)applyWithExtent:(CGRect)extent
                          roiCallback:(CIKernelROICallback)callback
                            arguments:(nullable NSArray<id> *)args;
@end
//用于颜色修正的过滤函数
@interface CIColorKernel : CIKernel
+ (nullable instancetype)kernelWithString:(NSString *)string;
- (nullable CIImage *)applyWithExtent:(CGRect)extent
                            arguments:(nullable NSArray<id> *)args;
@end
//用于形状修正的过滤函数
@interface CIWarpKernel : CIKernel
+ (nullable instancetype)kernelWithString:(NSString *)string;
@end
//用于色彩混合的过滤函数
@interface CIBlendKernel : CIColorKernel
+ (nullable instancetype)kernelWithString:(NSString *)string;
- (nullable CIImage *)applyWithForeground:(CIImage*)foreground
                               background:(CIImage*)background;
@end
```

下面是一个简单的翻转图像的自定义过滤器示意，首先新建一个新的cikernel文件，命名为a.cikernel，如下：

```
kernel vec2 mirrorX ( float imageWidth )
{
// 获取待处理点的位置
vec2 currentVec = destCoord();
// 返回最终显示位置
return vec2 ( imageWidth - currentVec.x , currentVec.y );
}
```

新建一个过滤器类，命名为MyFilter，如下：

```objectivec
#import <CoreImage/CoreImage.h>
@interface MyFilter : CIFilter
@property(nonatomic,strong)CIImage * inputImage;
@end
#import "MyFilter.h"

@interface MyFilter()

@property(nonatomic,strong)CIWarpKernel * kernel;

@end

@implementation MyFilter



- (instancetype)init {
    
    self = [super init];
    if (self) {
            //从文件读取过滤函数
            NSBundle *bundle = [NSBundle bundleForClass: [self class]];
            NSURL *kernelURL = [bundle URLForResource:@"a" withExtension:@"cikernel"];
            NSError *error;
            NSString *kernelCode = [NSString stringWithContentsOfURL:kernelURL
                                                            encoding:NSUTF8StringEncoding error:&error];
            
            NSArray *kernels = [CIKernel kernelsWithString:kernelCode];
            self.kernel = [kernels objectAtIndex:0];
    }
    return self;
}

- (CIImage *)outputImage
{
    CGFloat inputWidth = self.inputImage.extent.size.width;
    CIImage *result = [self.kernel applyWithExtent:self.inputImage.extent roiCallback:^CGRect(int index, CGRect destRect) {
        return destRect;
    } inputImage:self.inputImage arguments:@[@(inputWidth)]];
    return result;
}
//设置说明字典
-(NSDictionary<NSString *,id> *)attributes{
    return @{
             @"inputImage" :  @{
                 @"CIAttributeClass" : @"CIImage",
                 @"CIAttributeDisplayName" : @"Image--",
                 @"CIAttributeType" : @"CIAttributeTypeImage"
                 }};
}
@end
```

如下进行使用即可：

```objectivec
MyFilter * filter = [[MyFilter alloc]init];
filter.inputImage = image2;
CIContext * context = [[CIContext alloc]initWithOptions:nil];
CIImage * output = [filter outputImage];
CGImageRef ref = [context createCGImage:output fromRect:output.extent];
UIImage * newImage = [UIImage imageWithCGImage:ref];
```

### 二、使用CoreImage实现人脸识别

    人脸识别是目前非常热门的一种图像处理技术，CoreImage内置了对人脸进行识别的相关功能接口，并且可以对人脸面部特征进行抓取，下面我们来实现一个简单的实时识别人脸特征的Demo。

    首先创建一个视图作为图像扫描视图，如下：

.h文件

```objectivec
//.h 文件
@interface FaceView : UIView
@end
```

 .m文件

```objectivec
//
//  FaceView.m
//  CoreImageDemo
//
//  Created by jaki on 2018/12/22.
//  Copyright © 2018年 jaki. All rights reserved.
//

#import "FaceView.h"
#import <AVFoundation/AVFoundation.h>
#import "FaceHandle.h"
//定义线程
#define FACE_SCAN_QUEUE "FACE_SCAN_QUEUE"

@interface FaceView()<AVCaptureVideoDataOutputSampleBufferDelegate>

@property(nonatomic,strong)AVCaptureSession *captureSession;

@property(nonatomic,strong)AVCaptureDeviceInput * captureInput;

@property(nonatomic,strong)AVCaptureVideoDataOutput * captureOutput;

@property(nonnull,strong)AVCaptureVideoPreviewLayer * videoLayer;

@property(nonatomic,strong)dispatch_queue_t queue;

@property(nonatomic,assign)BOOL hasHandle;

@property(nonatomic,strong)UIView * faceView;

@end

@implementation FaceView
#pragma mark - Override
-(instancetype)init{
    self = [super init];
    if (self) {
        [self install];
    }
    return self;
}

-(instancetype)initWithFrame:(CGRect)frame{
    self = [super initWithFrame:frame];
    if (self) {
        [self install];
    }
    return self;
}
-(void)layoutSubviews{
    [super layoutSubviews];
    self.videoLayer.frame = self.bounds;
}

#pragma mark - InnerFunc
-(void)install{
    if (![UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeCamera]) {
        NSLog(@"不支持");
        return;
    }
    self.queue = dispatch_queue_create(FACE_SCAN_QUEUE, NULL);
    [self.captureSession startRunning];
    AVAuthorizationStatus status = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeVideo];
    if (status!=AVAuthorizationStatusAuthorized) {
        NSLog(@"需要权限");
        return;
    }
    self.videoLayer = [AVCaptureVideoPreviewLayer layerWithSession:self.captureSession];
    self.videoLayer.frame = CGRectZero;
    self.videoLayer.videoGravity = AVLayerVideoGravityResizeAspectFill;
    [self.layer addSublayer:self.videoLayer];
    [self addSubview:self.faceView];
    self.faceView.frame = CGRectMake(0, 0, self.frame.size.width, self.frame.size.height);
    
}
//将人脸特征点标记出来
-(void)renderReactWithInfo:(NSDictionary *)info{
    for (UIView * v in self.faceView.subviews) {
        [v removeFromSuperview];
    }
    NSArray * faceArray = info[FACE_HANDLE_INFO_FACE_ARRAY];
    for (int i = 0;i < faceArray.count; i++) {
        NSDictionary * face = faceArray[i];
        NSValue * faceValue = face[FACE_HANDLE_INFO_FACE_FRAME];
        if (faceValue) {
            CGRect faceR = [faceValue CGRectValue];
            UIView * faceView = [[UIView alloc]initWithFrame:faceR];
            faceView.backgroundColor = [UIColor clearColor];
            faceView.layer.borderColor = [UIColor redColor].CGColor;
            faceView.layer.borderWidth = 2;
            [self.faceView addSubview:faceView];
        }
        NSValue * leftEye = face[FACE_HANDLE_INFO_FACE_LEFT_EYE_FRAME];
        if (leftEye) {
            CGRect leftEyeR = [leftEye CGRectValue];
            UIView * eye = [[UIView alloc]initWithFrame:leftEyeR];
            eye.backgroundColor = [UIColor clearColor];
            eye.layer.borderColor = [UIColor greenColor].CGColor;
            eye.layer.borderWidth = 2;
            [self.faceView addSubview:eye];
        }
        NSValue * rightEye = face[FACE_HANDLE_INFO_FACE_RIGHT_EYE_FRAME];
        if (rightEye) {
            CGRect rightEyeR = [rightEye CGRectValue];
            UIView * eye = [[UIView alloc]initWithFrame:rightEyeR];
            eye.backgroundColor = [UIColor clearColor];
            eye.layer.borderColor = [UIColor greenColor].CGColor;
            eye.layer.borderWidth = 2;
            [self.faceView addSubview:eye];
        }
        NSValue * mouth = face[FACE_HANDLE_INFO_FACE_MOUTH_FRAME];
        if (mouth) {
            CGRect mouthR = [mouth CGRectValue];
            UIView * mouth = [[UIView alloc]initWithFrame:mouthR];
            mouth.backgroundColor = [UIColor clearColor];
            mouth.layer.borderColor = [UIColor orangeColor].CGColor;
            mouth.layer.borderWidth = 2;
            [self.faceView addSubview:mouth];
        }
    }
}


#pragma AVDelegate
//进行画面的捕获
-(void)captureOutput:(AVCaptureOutput *)output didOutputSampleBuffer:(CMSampleBufferRef)sampleBuffer fromConnection:(AVCaptureConnection *)connection{
    if (self.hasHandle) {
        return;
    }
    self.hasHandle = YES;
    CVImageBufferRef imageBuffer = CMSampleBufferGetImageBuffer(sampleBuffer);
    CVPixelBufferLockBaseAddress(imageBuffer,0);
    uint8_t *baseAddress = (uint8_t *)CVPixelBufferGetBaseAddress(imageBuffer);
    size_t bytesPerRow = CVPixelBufferGetBytesPerRow(imageBuffer);
    size_t width = CVPixelBufferGetWidth(imageBuffer);
    size_t height = CVPixelBufferGetHeight(imageBuffer);
    
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
    CGContextRef newContext = CGBitmapContextCreate(baseAddress,width, height, 8, bytesPerRow, colorSpace,kCGBitmapByteOrder32Little | kCGImageAlphaPremultipliedFirst);
    CGImageRef newImage = CGBitmapContextCreateImage(newContext);
    CGContextRelease(newContext);
    CGColorSpaceRelease(colorSpace);
    UIImage *image= [UIImage imageWithCGImage:newImage scale:1.0 orientation:UIImageOrientationRight];
    CGImageRelease(newImage);
    //image
     //进行人脸识别的核心工具类
    [[FaceHandle sharedInstance] handleImage:image viewSize:self.frame.size completed:^(BOOL success, NSDictionary *info) {
        self.hasHandle  = NO;
        [self renderReactWithInfo:info];
    }];
    
    CVPixelBufferUnlockBaseAddress(imageBuffer,0);
}



#pragma mark - setter and getter

-(AVCaptureSession *)captureSession{
    if (!_captureSession) {
        _captureSession = [[AVCaptureSession alloc]init];
        [_captureSession addInput:self.captureInput];
        [_captureSession addOutput:self.captureOutput];
    }
    return _captureSession;
}

-(AVCaptureDeviceInput *)captureInput{
    if (!_captureInput) {
        _captureInput = [AVCaptureDeviceInput deviceInputWithDevice:[AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo] error:nil];
    }
    return _captureInput;
}

-(AVCaptureVideoDataOutput *)captureOutput{
    if (!_captureOutput) {
        _captureOutput = [[AVCaptureVideoDataOutput alloc]init];
        _captureOutput.alwaysDiscardsLateVideoFrames = YES;
        [_captureOutput setSampleBufferDelegate:self queue:self.queue];
        _captureOutput.videoSettings = @{(__bridge NSString *)kCVPixelBufferPixelFormatTypeKey:@(kCVPixelFormatType_32BGRA)};
    }
    return _captureOutput;
}


-(UIView *)faceView{
    if (!_faceView) {
        _faceView = [[UIView alloc]init];
        _faceView.backgroundColor = [UIColor clearColor];
    }
    return _faceView;
}



@end


```

在真机上运行工程，通过摄像头可以将实时的画面捕获到屏幕上，下面实现核心的人脸识别代码：

创建继承于NSObject的FaceHandle类，如下：

.h文件

```objectivec
extern const NSString * FACE_HANDLE_INFO_FACE_ARRAY;

extern const NSString * FACE_HANDLE_INFO_FACE_FRAME;

extern const NSString * FACE_HANDLE_INFO_FACE_LEFT_EYE_FRAME;

extern const NSString * FACE_HANDLE_INFO_FACE_RIGHT_EYE_FRAME;

extern const NSString * FACE_HANDLE_INFO_FACE_MOUTH_FRAME;

extern const NSString * FACE_HANDLE_INFO_ERROR;
@interface FaceHandle : NSObject
+(instancetype)sharedInstance;


-(void)handleImage:(UIImage *)image viewSize:(CGSize )viewSize completed:(void(^)(BOOL  success,NSDictionary * info))completion;
@end

```

.m文件

```objectivec
#import "FaceHandle.h"
#define FACE_HANDLE_DISPATCH_QUEUE "FACE_HANDLE_DISPATCH_QUEUE"
const NSString * FACE_HANDLE_INFO_FACE_FRAME = @"FACE_HANDLE_INFO_FACE_FRAME";

const NSString * FACE_HANDLE_INFO_FACE_LEFT_EYE_FRAME = @"FACE_HANDLE_INFO_FACE_LEFT_EYE_FRAME";

const NSString * FACE_HANDLE_INFO_FACE_RIGHT_EYE_FRAME = @"FACE_HANDLE_INFO_FACE_RIGHT_EYE_FRAME";

const NSString * FACE_HANDLE_INFO_FACE_MOUTH_FRAME = @"FACE_HANDLE_INFO_FACE_MOUTH_FRAME";

const NSString * FACE_HANDLE_INFO_ERROR = @"FACE_HANDLE_INFO_ERROR";

const NSString * FACE_HANDLE_INFO_FACE_ARRAY = @"FACE_HANDLE_INFO_FACE_ARRAY";
@interface FaceHandle()

@property(nonatomic,strong)dispatch_queue_t workingQueue;

@end

@implementation FaceHandle

+(instancetype)sharedInstance{
    static dispatch_once_t onceToken;
    static FaceHandle * sharedInstance = nil;
    if (!sharedInstance) {
        dispatch_once(&onceToken, ^{
            sharedInstance = [[FaceHandle alloc] init];
        });
    }
    return sharedInstance;
}

#pragma mark - Override
-(instancetype)init{
    self = [super init];
    if (self) {
        self.workingQueue = dispatch_queue_create(FACE_HANDLE_DISPATCH_QUEUE, NULL);
    }
    return self;
}


#pragma mark - InnerFunc
-(void)handleImage:(UIImage *)image viewSize:(CGSize )viewSize completed:(void (^)(BOOL , NSDictionary *))completion{
    if (!image) {
        if (completion) {
            completion(NO,@{FACE_HANDLE_INFO_ERROR:@"图片捕获出错"});
        }
        return;
    }
    dispatch_async(self.workingQueue, ^{
        UIImage * newImage = [self strectImage:image withSize:viewSize];
        if (newImage) {
            NSArray * faceArray = [self analyseFaceImage:newImage];
            if (completion) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    completion(YES,@{FACE_HANDLE_INFO_FACE_ARRAY:faceArray});
                });
            }
        }else{
            if (completion) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    completion(NO,@{FACE_HANDLE_INFO_ERROR:@"图片识别出错"});
                });
            }
        }
    });
}

//图片放大处理
-(UIImage *)strectImage:(UIImage *)img withSize:(CGSize)size{
    UIGraphicsBeginImageContext(size);
    CGRect thumbnailRect = CGRectZero;
    thumbnailRect.origin = CGPointMake(0, 0);
    thumbnailRect.size.width  = size.width;
    thumbnailRect.size.height = size.height;
    [img drawInRect:thumbnailRect];
    UIImage * newImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    if (newImage) {
        return  newImage;
    }
    return nil;
}

-(NSArray *)analyseFaceImage:(UIImage *)image{
    NSMutableArray * dataArray = [NSMutableArray array];
    CIImage * cImage = [CIImage imageWithCGImage:image.CGImage];
    NSDictionary* opts = [NSDictionary dictionaryWithObject:
                          CIDetectorAccuracyHigh forKey:CIDetectorAccuracy];
    //进行分析
    CIDetector* detector = [CIDetector detectorOfType:CIDetectorTypeFace
                                              context:nil options:opts];
    //获取特征数组
    NSArray* features = [detector featuresInImage:cImage];
    CGSize inputImageSize = [cImage extent].size;
    CGAffineTransform  transform = CGAffineTransformIdentity;
    transform = CGAffineTransformScale(transform, 1, -1);
    transform = CGAffineTransformTranslate(transform, 0, -inputImageSize.height);
    
    for (CIFaceFeature *faceFeature in features){
        NSMutableDictionary * faceDic = [NSMutableDictionary dictionary];
        CGRect faceViewBounds = CGRectApplyAffineTransform(faceFeature.bounds, transform);
        [faceDic setValue:[NSValue valueWithCGRect:faceViewBounds] forKey:(NSString *)FACE_HANDLE_INFO_FACE_FRAME];
        CGFloat faceWidth = faceFeature.bounds.size.width;
        if(faceFeature.hasLeftEyePosition){
            CGPoint faceViewLeftPoint = CGPointApplyAffineTransform(faceFeature.leftEyePosition, transform);
            CGRect leftEyeBounds = CGRectMake(faceViewLeftPoint.x-faceWidth*0.1, faceViewLeftPoint.y-faceWidth*0.1, faceWidth*0.2, faceWidth*0.2);
            [faceDic setValue:[NSValue valueWithCGRect:leftEyeBounds] forKey:(NSString *)FACE_HANDLE_INFO_FACE_LEFT_EYE_FRAME];
        }
        
        if(faceFeature.hasRightEyePosition){
            //获取人右眼对应的point
            CGPoint faceViewRightPoint = CGPointApplyAffineTransform(faceFeature.rightEyePosition, transform);
            CGRect rightEyeBounds = CGRectMake(faceViewRightPoint.x-faceWidth*0.1, faceViewRightPoint.y-faceWidth*0.1, faceWidth*0.2, faceWidth*0.2);
            [faceDic setValue:[NSValue valueWithCGRect:rightEyeBounds] forKey:(NSString *)FACE_HANDLE_INFO_FACE_RIGHT_EYE_FRAME];
        }
        
        if(faceFeature.hasMouthPosition){
            //获取人嘴巴对应的point
            CGPoint faceViewMouthPoint = CGPointApplyAffineTransform(faceFeature.mouthPosition, transform);
            CGRect mouthBounds = CGRectMake(faceViewMouthPoint.x-faceWidth*0.2, faceViewMouthPoint.y-faceWidth*0.2, faceWidth*0.4, faceWidth*0.4);
            [faceDic setValue:[NSValue valueWithCGRect:mouthBounds] forKey:(NSString *)FACE_HANDLE_INFO_FACE_MOUTH_FRAME];
        }
        [dataArray addObject:faceDic];
    }
    return [dataArray copy];
}
@end
```

打开百度，随便搜索一些人脸图片进行识别，可以看到识别率还是很高，如下图：

![](https://oscimg.oschina.net/oscnet/888a36f9fe83db22a5aef849152a8ec7952.jpg)

### 三、CIImage中提供了其他图像识别功能

        CIDetector除了可以用来进行人脸识别外，还支持进行二维码、矩形、文字等检测。

矩形区域识别，用来检测图像中的矩形边界，核心代码如下：

```objectivec
-(NSArray *)analyseRectImage:(UIImage *)image{
    NSMutableArray * dataArray = [NSMutableArray array];
    CIImage * cImage = [CIImage imageWithCGImage:image.CGImage];
    NSDictionary* opts = [NSDictionary dictionaryWithObject:
                          CIDetectorAccuracyHigh forKey:CIDetectorAccuracy];
    CIDetector* detector = [CIDetector detectorOfType:CIDetectorTypeRectangle
                                              context:nil options:opts];
    NSArray* features = [detector featuresInImage:cImage];
    CGSize inputImageSize = [cImage extent].size;
    CGAffineTransform  transform = CGAffineTransformIdentity;
    transform = CGAffineTransformScale(transform, 1, -1);
    transform = CGAffineTransformTranslate(transform, 0, -inputImageSize.height);
    
    for (CIRectangleFeature *feature in features){
        NSLog(@"%lu",features.count);
        NSMutableDictionary * dic = [NSMutableDictionary dictionary];
        CGRect viewBounds = CGRectApplyAffineTransform(feature.bounds, transform);
        [dic setValue:[NSValue valueWithCGRect:viewBounds] forKey:@"rectBounds"];
        CGPoint topLeft = CGPointApplyAffineTransform(feature.topLeft, transform);
        [dic setValue:[NSValue valueWithCGPoint:topLeft] forKey:@"topLeft"];
        CGPoint topRight = CGPointApplyAffineTransform(feature.topRight, transform);
        [dic setValue:[NSValue valueWithCGPoint:topRight] forKey:@"topRight"];
        CGPoint bottomLeft = CGPointApplyAffineTransform(feature.bottomLeft, transform);
        [dic setValue:[NSValue valueWithCGPoint:bottomLeft] forKey:@"bottomLeft"];
        CGPoint bottomRight = CGPointApplyAffineTransform(feature.bottomRight, transform);
        [dic setValue:[NSValue valueWithCGPoint:bottomRight] forKey:@"bottomRight"];
        [dataArray addObject:dic];
    }
    return [dataArray copy];
}
```

效果如下图所示：

![](https://oscimg.oschina.net/oscnet/335f852c44be9223f6cf37c87b21815de6a.jpg)

二维码扫描不仅可以分析出图片中的二维码位置，还可以解析出二维码数据，核心代码如下：

```objectivec
-(NSArray *)analyseQRImage:(UIImage *)image{
    NSMutableArray * dataArray = [NSMutableArray array];
    CIImage * cImage = [CIImage imageWithCGImage:image.CGImage];
    NSDictionary* opts = [NSDictionary dictionaryWithObject:
                          CIDetectorAccuracyHigh forKey:CIDetectorAccuracy];
    CIDetector* detector = [CIDetector detectorOfType:CIDetectorTypeQRCode
                                              context:nil options:opts];
    NSArray* features = [detector featuresInImage:cImage];
    CGSize inputImageSize = [cImage extent].size;
    CGAffineTransform  transform = CGAffineTransformIdentity;
    transform = CGAffineTransformScale(transform, 1, -1);
    transform = CGAffineTransformTranslate(transform, 0, -inputImageSize.height);
    
    for (CIQRCodeFeature *feature in features){
        NSMutableDictionary * dic = [NSMutableDictionary dictionary];
        CGRect viewBounds = CGRectApplyAffineTransform(feature.bounds, transform);
        [dic setValue:[NSValue valueWithCGRect:viewBounds] forKey:@"rectBounds"];
        CGPoint topLeft = CGPointApplyAffineTransform(feature.topLeft, transform);
        [dic setValue:[NSValue valueWithCGPoint:topLeft] forKey:@"topLeft"];
        CGPoint topRight = CGPointApplyAffineTransform(feature.topRight, transform);
        [dic setValue:[NSValue valueWithCGPoint:topRight] forKey:@"topRight"];
        CGPoint bottomLeft = CGPointApplyAffineTransform(feature.bottomLeft, transform);
        [dic setValue:[NSValue valueWithCGPoint:bottomLeft] forKey:@"bottomLeft"];
        CGPoint bottomRight = CGPointApplyAffineTransform(feature.bottomRight, transform);
        [dic setValue:[NSValue valueWithCGPoint:bottomRight] forKey:@"bottomRight"];
        [dic setValue:feature.messageString forKey:@"content"];
        [dataArray addObject:dic];
    }
    return [dataArray copy];
}

```

CIImage框架中还支持对文本区域进行分析，核心代码如下：

```objectivec
-(NSArray *)analyseTextImage:(UIImage *)image{
    NSMutableArray * dataArray = [NSMutableArray array];
    CIImage * cImage = [CIImage imageWithCGImage:image.CGImage];
    NSDictionary* opts = [NSDictionary dictionaryWithObject:
                          CIDetectorAccuracyHigh forKey:CIDetectorAccuracy];
    CIDetector* detector = [CIDetector detectorOfType:CIDetectorTypeText
                                              context:nil options:nil];
    NSArray* features = [detector featuresInImage:cImage options:@{CIDetectorReturnSubFeatures:@YES}];
    CGSize inputImageSize = [cImage extent].size;
    CGAffineTransform  transform = CGAffineTransformIdentity;
    transform = CGAffineTransformScale(transform, 1, -1);
    transform = CGAffineTransformTranslate(transform, 0, -inputImageSize.height);
    
    for (CITextFeature *feature in features){
        NSLog(@"%@",feature.subFeatures);
        NSMutableDictionary * dic = [NSMutableDictionary dictionary];
        CGRect viewBounds = CGRectApplyAffineTransform(feature.bounds, transform);
        [dic setValue:[NSValue valueWithCGRect:viewBounds] forKey:@"rectBounds"];
        CGPoint topLeft = CGPointApplyAffineTransform(feature.topLeft, transform);
        [dic setValue:[NSValue valueWithCGPoint:topLeft] forKey:@"topLeft"];
        CGPoint topRight = CGPointApplyAffineTransform(feature.topRight, transform);
        [dic setValue:[NSValue valueWithCGPoint:topRight] forKey:@"topRight"];
        CGPoint bottomLeft = CGPointApplyAffineTransform(feature.bottomLeft, transform);
        [dic setValue:[NSValue valueWithCGPoint:bottomLeft] forKey:@"bottomLeft"];
        CGPoint bottomRight = CGPointApplyAffineTransform(feature.bottomRight, transform);
        [dic setValue:[NSValue valueWithCGPoint:bottomRight] forKey:@"bottomRight"];
        
        [dataArray addObject:dic];
    }
    return [dataArray copy];
}
```

效果如下图所示：

![](https://oscimg.oschina.net/oscnet/aebc89407adb95c05bee296587332e41f87.jpg)

### 四、CoreImage中的相关核心类

####  1.CIColor类

CIColor类是CoreImage中描述色彩的类。

```objectivec
//通过CGColor创建CIColor
+ (instancetype)colorWithCGColor:(CGColorRef)c;
//构造方法
+ (instancetype)colorWithRed:(CGFloat)r green:(CGFloat)g blue:(CGFloat)b alpha:(CGFloat)a;
+ (instancetype)colorWithRed:(CGFloat)r green:(CGFloat)g blue:(CGFloat)b;
+ (nullable instancetype)colorWithRed:(CGFloat)r green:(CGFloat)g blue:(CGFloat)b alpha:(CGFloat)a colorSpace:(CGColorSpaceRef)colorSpace;
+ (nullable instancetype)colorWithRed:(CGFloat)r green:(CGFloat)g blue:(CGFloat)b colorSpace:(CGColorSpaceRef)colorSpace;
- (instancetype)initWithCGColor:(CGColorRef)c;
//通过字符串创建CIColor对象
+ (instancetype)colorWithString:(NSString *)representation;
- (instancetype)initWithRed:(CGFloat)r green:(CGFloat)g blue:(CGFloat)b alpha:(CGFloat)a;
- (instancetype)initWithRed:(CGFloat)r green:(CGFloat)g blue:(CGFloat)b;
- (nullable instancetype)initWithRed:(CGFloat)r green:(CGFloat)g blue:(CGFloat)b alpha:(CGFloat)a colorSpace:(CGColorSpaceRef)colorSpace;
- (nullable instancetype)initWithRed:(CGFloat)r green:(CGFloat)g blue:(CGFloat)b colorSpace:(CGColorSpaceRef)colorSpace;
//获取颜色分量个数
@property (readonly) size_t numberOfComponents;
//颜色分量
@property (readonly) const CGFloat *components;
//颜色透明度
@property (readonly) CGFloat alpha;
//色彩空间
@property (readonly) CGColorSpaceRef colorSpace;
//红绿蓝分量
@property (readonly) CGFloat red;
@property (readonly) CGFloat green;
@property (readonly) CGFloat blue;
//下面是定义的一些便捷的颜色变量
@property (class, strong, readonly) CIColor *blackColor  ;
@property (class, strong, readonly) CIColor *whiteColor  ;
@property (class, strong, readonly) CIColor *grayColor   ;
@property (class, strong, readonly) CIColor *redColor    ;
@property (class, strong, readonly) CIColor *greenColor  ;
@property (class, strong, readonly) CIColor *blueColor   ;
@property (class, strong, readonly) CIColor *cyanColor   ;
@property (class, strong, readonly) CIColor *magentaColor ;
@property (class, strong, readonly) CIColor *yellowColor  ;
@property (class, strong, readonly) CIColor *clearColor 
```

#### 2.CIImage类

CIImage是CoreImage中最核心的类，它描述了图像对象。

```objectivec
//创建一个新的CIImage实例
+ (CIImage *)imageWithCGImage:(CGImageRef)image;
//通过字典创建一个新的CIImage实例
/*
字典中的键
kCIImageColorSpace  设置颜色空间 为CGColorSpaceRef对象
kCIImageNearestSampling 是否临近采样  布尔值
kCIImageProperties    设置图片属性字典
kCIImageApplyOrientationProperty 布尔值 是否根据方向进行转换
kCIImageTextureTarget  NSNumber值 设置OpebGL目标纹理常数
kCIImageTextureFormat NSNumber值 设置OpebGL format
kCIImageAuxiliaryDepth 布尔值 是否返回深度图像
kCIImageAuxiliaryDisparity  布尔值 是否返回辅助时差图像
kCIImageAuxiliaryPortraitEffectsMatte  布尔值 是否返回肖像模板
*/
+ (CIImage *)imageWithCGImage:(CGImageRef)image
                      options:(nullable NSDictionary<CIImageOption, id> *)options;
//通过CALayer进行CIImage的创建
+ (CIImage *)imageWithCGLayer:(CGLayerRef)layer NS_DEPRECATED_MAC(10_4,10_11);
+ (CIImage *)imageWithCGLayer:(CGLayerRef)layer
                      options:(nullable NSDictionary<CIImageOption, id> *)options;
//使用bitmap数据创建CIImage
+ (CIImage *)imageWithBitmapData:(NSData *)data
                     bytesPerRow:(size_t)bytesPerRow
                            size:(CGSize)size
                          format:(CIFormat)format
                      colorSpace:(nullable CGColorSpaceRef)colorSpace;
//通过纹理创建CIImage
+ (CIImage *)imageWithTexture:(unsigned int)name
                         size:(CGSize)size
                      flipped:(BOOL)flipped
                   colorSpace:(nullable CGColorSpaceRef)colorSpace;
+ (CIImage *)imageWithTexture:(unsigned int)name
                         size:(CGSize)size
                      flipped:(BOOL)flipped
                      options:(nullable NSDictionary<CIImageOption, id> *)options;
+ (nullable CIImage *)imageWithMTLTexture:(id<MTLTexture>)texture
                                  options:(nullable NSDictionary<CIImageOption, id> *)options;
//通过url创建CIImage
+ (nullable CIImage *)imageWithContentsOfURL:(NSURL *)url;
+ (nullable CIImage *)imageWithContentsOfURL:(NSURL *)url
                                     options:(nullable NSDictionary<CIImageOption, id> *)options;
//通过NSDate创建CIImage
+ (nullable CIImage *)imageWithData:(NSData *)data;
+ (nullable CIImage *)imageWithData:(NSData *)data
                            options:(nullable NSDictionary<CIImageOption, id> *)options;
//通过CVImageBufferRef创建CIImage
+ (CIImage *)imageWithCVImageBuffer:(CVImageBufferRef)imageBuffer;
+ (CIImage *)imageWithCVImageBuffer:(CVImageBufferRef)imageBuffer
                            options:(nullable NSDictionary<CIImageOption, id> *)options;
//通过CVPixelBufferRef创建CIImage
+ (CIImage *)imageWithCVPixelBuffer:(CVPixelBufferRef)pixelBuffer;
+ (CIImage *)imageWithCVPixelBuffer:(CVPixelBufferRef)pixelBuffer
                            options:(nullable NSDictionary<CIImageOption, id> *)options;
//通过颜色创建CIImage
+ (CIImage *)imageWithColor:(CIColor *)color;
//创建空CIImage
+ (CIImage *)emptyImage;
//初始化方法
- (instancetype)initWithCGImage:(CGImageRef)image;
- (instancetype)initWithCGImage:(CGImageRef)image
                        options:(nullable NSDictionary<CIImageOption, id> *)options;
- (instancetype)initWithCGLayer:(CGLayerRef)layer);
- (instancetype)initWithCGLayer:(CGLayerRef)layer;
- (instancetype)initWithBitmapData:(NSData *)data
                       bytesPerRow:(size_t)bytesPerRow
                              size:(CGSize)size
                            format:(CIFormat)format
                        colorSpace:(nullable CGColorSpaceRef)colorSpace;
- (instancetype)initWithTexture:(unsigned int)name
                           size:(CGSize)size
                        flipped:(BOOL)flipped
                     colorSpace:(nullable CGColorSpaceRef)colorSpace;
- (instancetype)initWithTexture:(unsigned int)name
                           size:(CGSize)size
                        flipped:(BOOL)flipped
                        options:(nullable NSDictionary<CIImageOption, id> *)options;
- (nullable instancetype)initWithMTLTexture:(id<MTLTexture>)texture
                                    options:(nullable NSDictionary<CIImageOption, id> *)options；
- (nullable instancetype)initWithContentsOfURL:(NSURL *)url;
- (nullable instancetype)initWithContentsOfURL:(NSURL *)url
                                       options:(nullable NSDictionary<CIImageOption, id> *)options;
- (instancetype)initWithCVImageBuffer:(CVImageBufferRef)imageBuffer;
- (instancetype)initWithCVImageBuffer:(CVImageBufferRef)imageBuffer
                              options:(nullable NSDictionary<CIImageOption, id> *)options;
- (instancetype)initWithCVPixelBuffer:(CVPixelBufferRef)pixelBuffer;
- (instancetype)initWithCVPixelBuffer:(CVPixelBufferRef)pixelBuffer
                              options:(nullable NSDictionary<CIImageOption, id> *)options;
- (instancetype)initWithColor:(CIColor *)color;
//追加变换 返回结果CIImage对象
- (CIImage *)imageByApplyingTransform:(CGAffineTransform)matrix;
- (CIImage *)imageByApplyingOrientation:(int)orientation;
- (CIImage *)imageByApplyingCGOrientation:(CGImagePropertyOrientation)orientation;
//根据方向获取变换
- (CGAffineTransform)imageTransformForOrientation:(int)orientation;
- (CGAffineTransform)imageTransformForCGOrientation:(CGImagePropertyOrientation)orientation;
//进行混合
- (CIImage *)imageByCompositingOverImage:(CIImage *)dest;
//区域裁剪
- (CIImage *)imageByCroppingToRect:(CGRect)rect;
//返回图像边缘
- (CIImage *)imageByClampingToExtent;
//设置边缘 返回新图像对象
- (CIImage *)imageByClampingToRect:(CGRect)rect;
//用过滤器进行过滤
- (CIImage *)imageByApplyingFilter:(NSString *)filterName
               withInputParameters:(nullable NSDictionary<NSString *,id> *)params;
- (CIImage *)imageByApplyingFilter:(NSString *)filterName;
//图像边缘
@property (NS_NONATOMIC_IOSONLY, readonly) CGRect extent;
//属性字典
@property (atomic, readonly) NSDictionary<NSString *,id> *properties;
//通过URL创建的图像的URL
@property (atomic, readonly, nullable) NSURL *url;
//颜色空间
@property (atomic, readonly, nullable) CGColorSpaceRef colorSpace;
//通过CGImage创建的CGImage对象
@property (nonatomic, readonly, nullable) CGImageRef CGImage;
```

#### 3.CIContext类

    CIContext是CoreImage中的上下文对象，用来进行图片的渲染，已经转换为其他框架的图像对象。

```objectivec
//通过CGContextRef上下文创建CIContext上下文
/*
配置字典中可以进行配置的：
kCIContextOutputColorSpace   设置输出的颜色空间
kCIContextWorkingColorSpace  设置工作的颜色空间
kCIContextWorkingFormat      设置缓冲区数据格式
kCIContextHighQualityDownsample 布尔值
kCIContextOutputPremultiplied  设置输出是否带alpha通道
kCIContextCacheIntermediates  布尔值
kCIContextUseSoftwareRenderer 设置是否使用软件渲染
kCIContextPriorityRequestLow  是否低质量

*/
+ (CIContext *)contextWithCGContext:(CGContextRef)cgctx
                            options:(nullable NSDictionary<CIContextOption, id> *)options;
//创建上下文对象
+ (CIContext *)contextWithOptions:(nullable NSDictionary<CIContextOption, id> *)options;
+ (CIContext *)context;
- (instancetype)initWithOptions:(nullable NSDictionary<CIContextOption, id> *)options;
//使用指定的处理器创建CIContext
+ (CIContext *)contextWithMTLDevice:(id<MTLDevice>)device;
+ (CIContext *)contextWithMTLDevice:(id<MTLDevice>)device
                            options:(nullable NSDictionary<CIContextOption, id> *)options;
//工作的颜色空间
@property (nullable, nonatomic, readonly) CGColorSpaceRef workingColorSpace;
//缓冲区格式
@property (nonatomic, readonly) CIFormat workingFormat;
//进行CIImage图像的绘制
- (void)drawImage:(CIImage *)image
          atPoint:(CGPoint)atPoint
         fromRect:(CGRect)fromRect;
- (void)drawImage:(CIImage *)image
           inRect:(CGRect)inRect
         fromRect:(CGRect)fromRect;
//使用CIImage创建CGImageRef
- (nullable CGImageRef)createCGImage:(CIImage *)image;
                            fromRect:(CGRect)fromRect;
- (nullable CGImageRef)createCGImage:(CIImage *)image
                            fromRect:(CGRect)fromRect
                              format:(CIFormat)format
                          colorSpace:(nullable CGColorSpaceRef)colorSpace;
//创建CALayer
- (nullable CGLayerRef)createCGLayerWithSize:(CGSize)size
                                        info:(nullable CFDictionaryRef)info;
//将图片写入bitMap数据
- (void)render:(CIImage *)image
      toBitmap:(void *)data
      rowBytes:(ptrdiff_t)rowBytes
        bounds:(CGRect)bounds
        format:(CIFormat)format;
//将图片写入缓存
- (void)render:(CIImage *)image 
toCVPixelBuffer:(CVPixelBufferRef)buffer
    colorSpace:(nullable CGColorSpaceRef)colorSpace;
- (void)render:(CIImage *)image
toCVPixelBuffer:(CVPixelBufferRef)buffer
        bounds:(CGRect)bounds
    colorSpace:(nullable CGColorSpaceRef)colorSpace；
//将图片写入纹理
- (void)render:(CIImage *)image
  toMTLTexture:(id<MTLTexture>)texture
 commandBuffer:(nullable id<MTLCommandBuffer>)commandBuffer
        bounds:(CGRect)bounds
    colorSpace:(CGColorSpaceRef)colorSpace;
//清除缓存
- (void)clearCaches;
//输入图像的最大尺寸
- (CGSize)inputImageMaximumSize;
//输出图像的最大尺寸
- (CGSize)outputImageMaximumSize;
//将CIImage写成TIFF数据
- (nullable NSData*) TIFFRepresentationOfImage:(CIImage*)image
                                        format:(CIFormat)format
                                    colorSpace:(CGColorSpaceRef)colorSpace
                                       options:(NSDictionary<CIImageRepresentationOption, id>*)options;
//将CIImage写成JPEG数据
- (nullable NSData*) JPEGRepresentationOfImage:(CIImage*)image
                                    colorSpace:(CGColorSpaceRef)colorSpace
                                       options:(NSDictionary<CIImageRepresentationOption, id>*)options;
//将CIImage写成HEIF数据
- (nullable NSData*) HEIFRepresentationOfImage:(CIImage*)image
                                        format:(CIFormat)format
                                    colorSpace:(CGColorSpaceRef)colorSpace
                                       options:(NSDictionary<CIImageRepresentationOption, id>*)options;
//将CIImage写成PNG数据
- (nullable NSData*) PNGRepresentationOfImage:(CIImage*)image
                                       format:(CIFormat)format
                                   colorSpace:(CGColorSpaceRef)colorSpace
                                      options:(NSDictionary<CIImageRepresentationOption, id>*)options;
//将CIImage写入TIFF文件
- (BOOL) writeTIFFRepresentationOfImage:(CIImage*)image
                                  toURL:(NSURL*)url
                                 format:(CIFormat)format
                             colorSpace:(CGColorSpaceRef)colorSpace 
                                options:(NSDictionary<CIImageRepresentationOption, id>*)options
                                  error:(NSError **)errorPtr;
//将CIImage写入PNG文件
- (BOOL) writePNGRepresentationOfImage:(CIImage*)image
                                 toURL:(NSURL*)url
                                format:(CIFormat)format
                            colorSpace:(CGColorSpaceRef)colorSpace
                               options:(NSDictionary<CIImageRepresentationOption, id>*)options
                                 error:(NSError **)errorPtr;
//将CIImage写入JPEG文件
- (BOOL) writeJPEGRepresentationOfImage:(CIImage*)image
                                  toURL:(NSURL*)url
                             colorSpace:(CGColorSpaceRef)colorSpace
                                options:(NSDictionary<CIImageRepresentationOption, id>*)options
                                  error:(NSError **)errorPtr;
//将CIImage写HEIF文件
- (BOOL) writeHEIFRepresentationOfImage:(CIImage*)image
                                  toURL:(NSURL*)url
                                 format:(CIFormat)format
                             colorSpace:(CGColorSpaceRef)colorSpace
                                options:(NSDictionary<CIImageRepresentationOption, id>*)options
                                  error:(NSError **)errorPtr;
```

#### 4.CIDetector类

    前面有过CIDetector类的功能演示，这是CIImage框架中非常强大的一个类，使用它可以进行复杂的图片识别技术，解析如下：

```objectivec
//创建CIDetector实例 
/*
type用来指定识别的类型
CIDetectorTypeFace  人脸识别模式
CIDetectorTypeRectangle 矩形检测模式
CIDetectorTypeText   文本区域检测模式
CIDetectorTypeQRCode 二维码扫描模式


option可以指定配置字典 可配置的键如下
CIDetectorAccuracy 设置检测精度 CIDetectorAccuracyLow 低 CIDetectorAccuracyHigh 高
CIDetectorTracking 设置是否跟踪特征
CIDetectorMinFeatureSize  设置特征最小尺寸 0-1之间 相对图片
CIDetectorMaxFeatureCount 设置最大特征数
CIDetectorImageOrientation 设置方向
CIDetectorEyeBlink  设置布尔值 是否提取面部表情 眨眼
CIDetectorSmile    设置布尔值 是否提取面部表情  微笑
CIDetectorFocalLength  设置焦距
CIDetectorAspectRatio 设置检测到矩形的宽高比
CIDetectorReturnSubFeatures 设置是否提取子特征

*/
+ (nullable CIDetector *)detectorOfType:(NSString*)type
                                context:(nullable CIContext *)context
                                options:(nullable NSDictionary<NSString *,id> *)options;
//进行图片分析 提取特征数组
- (NSArray<CIFeature *> *)featuresInImage:(CIImage *)image;
- (NSArray<CIFeature *> *)featuresInImage:(CIImage *)image
                                  options:(nullable NSDictionary<NSString *,id> *)options;
```

#### 5.CIFeature相关类

CIFeature与其相关子类定义了特征数据模型。

```objectivec
@interface CIFeature : NSObject {}
//特征类型
/*
CIFeatureTypeFace
CIFeatureTypeRectangle
CIFeatureTypeQRCode
CIFeatureTypeText
*/
@property (readonly, retain) NSString *type;
//特征在图片中的bounds
@property (readonly, assign) CGRect bounds;
@end

//人脸特征对象
@interface CIFaceFeature : CIFeature
//位置尺寸
@property (readonly, assign) CGRect bounds;
//左眼位置
@property (readonly, assign) BOOL hasLeftEyePosition;
@property (readonly, assign) CGPoint leftEyePosition;
//是否有左眼特征
@property (readonly, assign) BOOL hasRightEyePosition;
//右眼位置
@property (readonly, assign) CGPoint rightEyePosition;
//是否有右眼特征
@property (readonly, assign) BOOL hasMouthPosition;
//口部特征
@property (readonly, assign) CGPoint mouthPosition;
//是否有跟踪特征ID
@property (readonly, assign) BOOL hasTrackingID;
//跟踪特征ID
@property (readonly, assign) int trackingID;
@property (readonly, assign) BOOL hasTrackingFrameCount;
@property (readonly, assign) int trackingFrameCount;
@property (readonly, assign) BOOL hasFaceAngle;
@property (readonly, assign) float faceAngle;
//是否微笑
@property (readonly, assign) BOOL hasSmile;
//左眼是否闭眼
@property (readonly, assign) BOOL leftEyeClosed;
//右眼是否闭眼
@property (readonly, assign) BOOL rightEyeClosed;

@end

//矩形特征对象
@interface CIRectangleFeature : CIFeature
//位置尺寸
@property (readonly) CGRect bounds;
@property (readonly) CGPoint topLeft;
@property (readonly) CGPoint topRight;
@property (readonly) CGPoint bottomLeft;
@property (readonly) CGPoint bottomRight;

@end

//二维码特征对象
@interface CIQRCodeFeature : CIFeature
//位置尺寸信息
@property (readonly) CGRect bounds;
@property (readonly) CGPoint topLeft;
@property (readonly) CGPoint topRight;
@property (readonly) CGPoint bottomLeft;
@property (readonly) CGPoint bottomRight;
//二维码内容
@property (nullable, readonly) NSString* messageString;
//二维码描述数据
@property (nullable, readonly) CIQRCodeDescriptor *symbolDescriptor NS_AVAILABLE(10_13, 11_0);

@end

//文本特征对象
@interface CITextFeature : CIFeature
//位置信息
@property (readonly) CGRect bounds;
@property (readonly) CGPoint topLeft;
@property (readonly) CGPoint topRight;
@property (readonly) CGPoint bottomLeft;
@property (readonly) CGPoint bottomRight;
//子特征
@property (nullable, readonly) NSArray *subFeatures;


@end
```

> 热爱技术，热爱生活，写代码，交朋友 珲少  QQ：316045346

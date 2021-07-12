---
title: iOS开发CoreGraphics核心图形框架之七——图像处理
date: 2016-11-28
categories: iOS逻辑初窥
tags: []
---
## iOS开发CoreGraphics核心图形框架之七——图像处理

### 一、引言

    位图图像数据实际上一个像素阵列，其中每个像素代表了图像中的一个点。位图实际上只支持矩形区域的渲染，但是使用透明技术可以实现任意形状图像的渲染。开发者也可以对要进行渲染的图像进行旋转、切割等操作。

### 二、通过图像裁剪创建图像

    CoreGraphics框架中提供了许多方法来创建位图数据引用CGImageRef对象，其中封装在CGImage.h文件中。在UIKit框架中也提供了方便的接口供开发者进行CGImageRef与UIImage对象的相互转换。

    通过CoreGraphics框架中提供的图像裁剪方法，开发者可以截取一张大图片中的一部分作为新的图像进行渲染。在Web开发中，为了减少请求次数，常常会将许多小图片合成一张大图片返回给前端，同时还会给前端返回一个json文件，文件中存放着每个独立小图的坐标位置，前端在使用时进行截取即可，这种图片常常被称作雪碧图。在iOS开发中游戏开发中，很多游戏引擎也提供了类似的方法，方便开发者对游戏素材进行管理。实际上，通过CoreGraphics框架，开发者也可以自己实现一套这样的图片加载逻辑，如果在自己的应用中，同时需要异步加载的小图片很多，也可以设计成下载一张大图后从中截取需要的图片。进行图像截取的示例代码如下：

```objectivec
-(void)drawRect:(CGRect)rect{
    CGContextRef contextRef = UIGraphicsGetCurrentContext();
    //进行坐标系的翻转
    CGContextTranslateCTM(contextRef, 0, rect.size.height);
    CGContextScaleCTM(contextRef, 1.0, -1.0);
    CGImageRef orignImage = [UIImage imageNamed:@"image"].CGImage;
    CGContextDrawImage(contextRef, CGRectMake(0, 0, 320, 200), orignImage);
    CGImageRef rectImage =  CGImageCreateWithImageInRect(orignImage, CGRectMake(300, 400, 800, 400));
    CGContextDrawImage(contextRef, CGRectMake(0, 220, 320, 200), rectImage);
    CGImageRelease(orignImage);
    CGImageRelease(rectImage);
}
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1128/161011_RH56_2340880.png)

需要注意，CGContextDrawImage()方法渲染的图像是上下翻转的，可以通过调整坐标系来将图片翻转回来。

### 三、通过膜层来实现图像的自定义裁剪

    通过Mask膜层可以实现炫酷的图像裁剪与风格重绘。膜层可以简单的理解为将一个图层追加到原图层上，但需要注意，图层中颜色为纯黑的部分，会按照原图绘制，纯白的部分会被完全遮挡，这中间的颜色会以特定的算法进行alpha值的更改。例如将如下图片作为膜层绘制到原图像上：

![](https://static.oschina.net/uploads/space/2016/1128/170652_uCL0_2340880.png)

代码如下：

```objectivec
-(void)drawRect:(CGRect)rect{
    CGContextRef contextRef = UIGraphicsGetCurrentContext();
    //进行坐标系的翻转
    CGContextTranslateCTM(contextRef, 0, rect.size.height);
    CGContextScaleCTM(contextRef, 1.0, -1.0);

    CGImageRef orignImage = [UIImage imageNamed:@"image"].CGImage;
    CGImageRef maskRef = [UIImage imageNamed:@"maskImage"].CGImage;
    //通过图片数据创建膜层
    CGImageRef mask = CGImageMaskCreate(CGImageGetWidth(maskRef),
                      CGImageGetHeight(maskRef),
                      CGImageGetBitsPerComponent(maskRef),
                      CGImageGetBitsPerPixel(maskRef),
                      CGImageGetBytesPerRow(maskRef),
                      CGImageGetDataProvider(maskRef), nil, YES);
    CGImageRef resultImage =  CGImageCreateWithMask(orignImage, mask);
    CGContextDrawImage(contextRef, CGRectMake(0, 0, 320, 200), resultImage);
    CGImageRelease(orignImage);
    CGImageRelease(maskRef);
    CGImageRelease(mask);
}
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1128/170854_5y1T_2340880.png)

    除了使用图片膜层来对原图像数据进行裁剪处理外，还可以通过颜色数据定义膜层来进行裁剪。这个方法就能加强大了，其可以将图像中某个范围的颜色所对应的所有区域裁剪出来。示例代码如下：

```objectivec
-(void)drawRect:(CGRect)rect{
    CGContextRef contextRef = UIGraphicsGetCurrentContext();
    //进行坐标系的翻转
    CGContextTranslateCTM(contextRef, 0, rect.size.height);
    CGContextScaleCTM(contextRef, 1.0, -1.0);

    CGImageRef orignImage = [UIImage imageNamed:@"image2"].CGImage;
    const CGFloat myMaskingColors[6] = {35, 154,  23, 194, 103, 214};
    CGImageRef mask2 = CGImageCreateWithMaskingColors(orignImage, myMaskingColors);
    CGContextDrawImage(contextRef, CGRectMake(0, 0, 320, 200), mask2);
    
    CGImageRelease(orignImage);
    CGImageRelease(mask2);
}

```

CGImageCreateWithMaskingColors()这个方法需要两个参数，第一个参数是要进行裁剪的图像，那二个参数需要设置为一个表示色彩的数组，需要注意，这个数组中元素的个数需要是当前色彩空间颜色原色数的两倍，例如RGB色彩空间对应这个数组需要有6个元素{min1,max1,min2,max2,min3,max3}。之后会对图像数据中的每一个像素点进行遍历，假如此像素点的颜色值为{c1,c2,c3}。则当满足如下条件时，这个像素点会被裁剪：

min1<c1<max1,min2<c2<max2,min3<c3<max3

需要注意，使用这种方式进行膜层裁剪，原图像不可以有alpha通道，色值的取值范围为0-255之间。上面示例代码会将原图像裁剪成如下效果：

![](https://static.oschina.net/uploads/space/2016/1128/182207_IV0n_2340880.png)

对于被裁剪出来的部分，开发者可以使用其他颜色进行填充，示例代码如下：

```objectivec
-(void)drawRect:(CGRect)rect{
    CGContextRef contextRef = UIGraphicsGetCurrentContext();
    //进行坐标系的翻转
    CGContextTranslateCTM(contextRef, 0, rect.size.height);
    CGContextScaleCTM(contextRef, 1.0, -1.0);
    CGImageRef orignImage = [UIImage imageNamed:@"image2"].CGImage;
    //设置填充
    [[UIColor redColor] setFill];
    CGContextFillRect(contextRef, CGRectMake(0, 0, 320, 200));
    const CGFloat myMaskingColors[6] = {35, 154,  23, 194, 103, 214};
    CGImageRef mask2 = CGImageCreateWithMaskingColors(orignImage, myMaskingColors);
    CGContextDrawImage(contextRef, CGRectMake(0, 0, 320, 200), mask2);
    
    CGImageRelease(orignImage);
}
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/1128/182413_vzad_2340880.png)

    除了上面介绍了两种对图像进行裁剪的方法外，CoreGraphics框架中还提供了一种裁剪方式，示例代码如下：

```objectivec
-(void)drawRect:(CGRect)rect{
    CGContextRef contextRef = UIGraphicsGetCurrentContext();
    //进行坐标系的翻转
    CGContextTranslateCTM(contextRef, 0, rect.size.height);
    CGContextScaleCTM(contextRef, 1.0, -1.0);

    CGImageRef orignImage = [UIImage imageNamed:@"image2"].CGImage;
    CGImageRef maskRef = [UIImage imageNamed:@"maskImage"].CGImage;
    CGImageRef mask = CGImageMaskCreate(CGImageGetWidth(maskRef),
                      CGImageGetHeight(maskRef),
                      CGImageGetBitsPerComponent(maskRef),
                      CGImageGetBitsPerPixel(maskRef),
                      CGImageGetBytesPerRow(maskRef),
                      CGImageGetDataProvider(maskRef), nil, YES);
    //进行膜层的裁剪
    CGContextClipToMask(contextRef, CGRectMake(0, 0, 320, 200), mask);
    CGContextDrawImage(contextRef, CGRectMake(0, 0, 320, 200), orignImage);
    CGImageRelease(orignImage);
}
```

### 四、进行图像混合

    使用CoreGraphics框架也可以绘制复杂的图像混合效果，在进行图像混合时，需要先绘制背景图像，之后设置图像混合模式，在绘制前景图像，CoreGraphics会根据混合模式来进行最后图像的绘制。例如使用如下背景图像来与前景图像来进行混合：

背景图像：

![](https://static.oschina.net/uploads/space/2016/1128/184907_LUgN_2340880.png)

前景图像：

![](https://static.oschina.net/uploads/space/2016/1128/184946_J9NY_2340880.jpg)

示例代码如下：

```objectivec
-(void)drawRect:(CGRect)rect{
    CGContextRef contextRef = UIGraphicsGetCurrentContext();
    //进行坐标系的翻转
    CGContextTranslateCTM(contextRef, 0, rect.size.height);
    CGContextScaleCTM(contextRef, 1.0, -1.0);
    CGImageRef background = [UIImage imageNamed:@"background"].CGImage;
    CGContextDrawImage(contextRef, CGRectMake(60, 25, 200, 150), background);
    CGContextSetBlendMode(contextRef, kCGBlendModeNormal);
    CGImageRef orignImage = [UIImage imageNamed:@"image2"].CGImage;
    CGContextDrawImage(contextRef, CGRectMake(0, 0, 320, 200), orignImage);
    CGImageRelease(background);
    CGImageRelease(orignImage);
}
```

kCGBlendModeNormal模式的混合就是简单覆盖，前景图像会完全将背景图像覆盖，运行效果如下：

![](https://static.oschina.net/uploads/space/2016/1128/185444_x265_2340880.png)

kCGBlendModeMultiply模式是叠加混合模式，其会将前景图alpha化，效果如下：

![](https://static.oschina.net/uploads/space/2016/1128/185708_DYSL_2340880.png)

kCGBlendModeScreen模式会将前景图进行裁剪，最终的结果颜色将比原图轻，效果如下：

![](https://static.oschina.net/uploads/space/2016/1128/185853_CZGO_2340880.png)

kCGBlendModeOverlay模式也会将前景图进行裁剪，会保持原图色彩，效果如下：

![](https://static.oschina.net/uploads/space/2016/1128/190125_pVyy_2340880.png)

kCGBlendModeDarken混合模式会将原图色值加深，效果如下：

![](https://static.oschina.net/uploads/space/2016/1128/190336_K7HP_2340880.png)

kCGBlendModeLighten在混合时则会选择色值较轻的图像进行混合，效果如下：

![](https://static.oschina.net/uploads/space/2016/1128/190545_OAnH_2340880.png)

kCGBlendModeColorDodge混合模式效果如下：

![](https://static.oschina.net/uploads/space/2016/1128/190732_NrmO_2340880.png)

kCGBlendModeColorBurn混合模式效果如下：

![](https://static.oschina.net/uploads/space/2016/1128/190829_9B9i_2340880.png)

kCGBlendModeSoftLight为柔光混合模式，效果如下：

![](https://static.oschina.net/uploads/space/2016/1128/190955_zROq_2340880.png)

kCGBlendModeHardLight为重光混合模式，效果如下：

![](https://static.oschina.net/uploads/space/2016/1128/191127_YDho_2340880.png)

kCGBlendModeDifference差异混合模式会取颜色的逆向值，效果如下：

![](https://static.oschina.net/uploads/space/2016/1128/191255_I0iR_2340880.png)

kCGBlendModeExclusion混合模式效果如下：

![](https://static.oschina.net/uploads/space/2016/1128/191409_kgbM_2340880.png)

kCGBlendModeHue混合模式会改变色彩的饱和度，效果如下：

![](https://static.oschina.net/uploads/space/2016/1128/191512_hPjz_2340880.png)

kCGBlendModeSaturation混合模式效果如下：

![](https://static.oschina.net/uploads/space/2016/1128/191631_uMAk_2340880.png)

kCGBlendModeColor混合模式效果如下：

![](https://static.oschina.net/uploads/space/2016/1128/191749_cGkq_2340880.png)

kCGBlendModeLuminosity光影混合模式会将前景图进行黑白化，效果如下：

![](https://static.oschina.net/uploads/space/2016/1128/191908_R165_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: iOS中使用像素位图(CGImageRef)对图片进行处理
date: 2015-04-26
categories: iOS逻辑初窥
tags: []
---
## iOS中对图片进行重绘处理的方法总结

### 一、CGImageRef是什么

CGImageRef是定义在QuartzCore框架中的一个结构体指针，用C语言编写。在CGImage.h文件中，我们可以看到下面的定义：

```
typedef struct CGImage *CGImageRef;
```

CGImageRef 和 struct CGImage * 是完全等价的。这个结构用来创建像素位图，可以通过操作存储的像素位来编辑图片。

QuartzCore这个框架是可移植的。

### 二、CGImageRef相关的一些方法解析

CFTypeID CGImageGetTypeID(void)

这个方法返回的是一个编号，每个Core Foundation框架中得结构都会有一个这样的编号，CFTypeID定义如下：

```
#if __LLP64__
typedef unsigned long long CFTypeID;
typedef unsigned long long CFOptionFlags;
typedef unsigned long long CFHashCode;
typedef signed long long CFIndex;
#else
typedef unsigned long CFTypeID;
typedef unsigned long CFOptionFlags;
typedef unsigned long CFHashCode;
typedef signed long CFIndex;
#endif
```

这个方法没有特殊的意义，只是一个标识符。

CGImageRef CGImageCreate(size_t width, size_t height,

    size_t bitsPerComponent, size_t bitsPerPixel, size_t bytesPerRow,

 CGColorSpaceRef space, CGBitmapInfo bitmapInfo, CGDataProviderRef provider,

    const CGFloat decode\[\], bool shouldInterpolate,

 CGColorRenderingIntent intent);

通过这个方法，我们可以创建出一个CGImageRef类型的对象，下面分别对参数进行解释：

sizt_t是定义的一个可移植性的单位，在64位机器中为8字节，32位位4字节。

width：图片宽度像素

height：图片高度像素

bitsPerComponent：每个颜色的比特数，例如在rgba-32模式下为8

bitsPerPixel：每个像素的总比特数

bytesPerRow：每一行占用的字节数，注意这里的单位是字节

space：颜色空间模式，例如const CFStringRef kCGColorSpaceGenericRGB 这个函数可以返回一个颜色空间对象。

bitmapInfo：位图像素布局，枚举如下：

```
typedef CF_OPTIONS(uint32_t, CGBitmapInfo) {
  kCGBitmapAlphaInfoMask = 0x1F,
  kCGBitmapFloatComponents = (1 << 8),
    
  kCGBitmapByteOrderMask = 0x7000,
  kCGBitmapByteOrderDefault = (0 << 12),
  kCGBitmapByteOrder16Little = (1 << 12),
  kCGBitmapByteOrder32Little = (2 << 12),
  kCGBitmapByteOrder16Big = (3 << 12),
  kCGBitmapByteOrder32Big = (4 << 12)
}
```

provider：数据源提供者

decode\[\]：解码渲染数组

shouldInterpolate：是否抗锯齿

intent：图片相关参数

CGImageRef CGImageMaskCreate(size_t width, size_t height,

    size_t bitsPerComponent, size_t bitsPerPixel, size_t bytesPerRow,

    CGDataProviderRef provider, const CGFloat decode\[\], bool shouldInterpolate)

这个方法用于创建mask图片图层，可以设置其显示部分与不显示部分达到特殊的效果，参数意义同上。

CGImageRef CGImageCreateCopy(CGImageRef image)

这个方法可以复制一个CGImageRef对象

CGImageRef CGImageCreateWithJPEGDataProvider(CGDataProviderRef

    source, const CGFloat decode\[\], bool shouldInterpolate,

 CGColorRenderingIntent intent)

通过JPEG数据源获取图像

CGImageRef CGImageCreateWithPNGDataProvider(CGDataProviderRef source,

    const CGFloat decode\[\], bool shouldInterpolate,

 CGColorRenderingIntent intent)

通过PNG数据源获取图像

CGImageRef CGImageCreateWithImageInRect(CGImageRef image,

    CGRect rect)

  
截取图像的一个区域重绘图像

CGImageRef CGImageCreateWithMask(CGImageRef image, CGImageRef mask)

截取mask图像的某一区域重绘

CGImageRef CGImageCreateWithMaskingColors(CGImageRef image,

    const CGFloat components\[\])

通过颜色分量数组创建位图

CGImageRef CGImageCreateCopyWithColorSpace(CGImageRef image,

 CGColorSpaceRef space)

通过颜色空间模式复制位图

CGImageRef CGImageRetain(CGImageRef image)

引用+1

void CGImageRelease(CGImageRef image)

引用-1

bool CGImageIsMask(CGImageRef image)

返回是否为Mask图层

size_t CGImageGetWidth(CGImageRef image)

获取宽度像素

size_t CGImageGetHeight(CGImageRef image)

获取高度像素

下面这些方法分别获取相应属性

size_t CGImageGetBitsPerComponent(CGImageRef image)

size_t CGImageGetBitsPerPixel(CGImageRef image)

size_t CGImageGetBytesPerRow(CGImageRef image)

CGColorSpaceRef CGImageGetColorSpace(CGImageRef image)CG_EXTERN CGImageAlphaInfo CGImageGetAlphaInfo(CGImageRef image)

CGDataProviderRef CGImageGetDataProvider(CGImageRef image)

const CGFloat *CGImageGetDecode(CGImageRef image)

bool CGImageGetShouldInterpolate(CGImageRef image)

CGColorRenderingIntent CGImageGetRenderingIntent(CGImageRef image)

CGBitmapInfo CGImageGetBitmapInfo(CGImageRef image)

### 三、应用举例

使用CGImageRef进行图片截取

```
    //原图片
    UIImage * img = [UIImage imageNamed:@"11.11.52.png"];
    //转化为位图
    CGImageRef temImg = img.CGImage;
    //根据范围截图
    temImg=CGImageCreateWithImageInRect(temImg, CGRectMake(0, 0, 100, 100));
    //得到新的图片
    UIImage *new = [UIImage imageWithCGImage:temImg];
    //释放位图对象
    CGImageRelease(temImg);
```

注意：最后必须要调用这个函数，否则会造成内存泄露

```
 CGImageRelease(temImg)
```

疏漏之处 欢迎指正

学习使用 欢迎转载

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

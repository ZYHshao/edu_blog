---
title: iOS开发CoreGraphics核心图形框架之三——颜色与色彩空间
date: 2016-10-20
categories: iOS逻辑初窥
tags: []
---
## iOS开发CoreGraphics核心图形框架之三——颜色与色彩空间

### 一、引言

    颜色的实质是表示颜色的二进制数据，如果没有确定的规则，则这些二进制数据完全没有意义。所谓色彩空间，即是表示这些颜色数据信息是如何解释的。同样的一张图片，在不同的色彩空间下，其渲染的模样将有很大的不同。在CoreGraphics框架中，与色彩相关的功能主要有CGColor与CGColorSpace构成。

### 二、关于CGColor相关方法的解析

    CGColorRef是CoreGraphics框架中用于描述颜色的引用类型，其中常用方法解析如下：

```objectivec
//根据色彩空间创建一个CGColorRef实例
/*
需要注意，这个方法中的第2个参数需要传递一个float数据，其需要和第1个参数的色彩空间意义对应
例如在RGBA色彩空间中，float数组中需要传递4个值，分别表示红绿蓝和透明度
*/
CGColorRef __nullable CGColorCreate(CGColorSpaceRef cg_nullable space, const CGFloat * cg_nullable components);
//创建黑白色彩空间下的颜色
/*
参数 
gray为灰度
alpha 为透明度
*/
CGColorRef  CGColorCreateGenericGray(CGFloat gray, CGFloat alpha);
//创建RGB色彩空间下的颜色
CGColorRef  CGColorCreateGenericRGB(CGFloat red, CGFloat green, CGFloat blue, CGFloat alpha);
//创建CMYB印刷模式色彩空间下的颜色
CGColorCreateGenericCMYK(CGFloat cyan, CGFloat magenta, CGFloat yellow, CGFloat black, CGFloat alpha);
//获取颜色常量
/*
colorName定义如下：
 //标准白色
 CFStringRef  kCGColorWhite;
 //标准黑色
 CFStringRef  kCGColorBlack;
 //标准透明色
 CFStringRef  kCGColorClear;
*/
CGColorRef __nullable CGColorGetConstantColor(CFStringRef cg_nullable colorName);
//通过模式与色彩空间创建颜色
CGColorRef __nullable CGColorCreateWithPattern(CGColorSpaceRef cg_nullable space, CGPatternRef cg_nullable pattern, const CGFloat * cg_nullable components);
//复制一个CGColorRef
CGColorRef __nullable CGColorCreateCopy(CGColorRef cg_nullable color);
//复制颜色 并追加透明度
CGColorRef __nullable CGColorCreateCopyWithAlpha(CGColorRef cg_nullable color, CGFloat alpha);
//将原色彩空间与目标色彩空间相匹配 创建颜色实例
/*
CGColorRenderingInter设置颜色渲染模式
typedef CF_ENUM (int32_t, CGColorRenderingIntent) {
    kCGRenderingIntentDefault,   //默认的渲染模式
    kCGRenderingIntentAbsoluteColorimetric, //绝对比色模式
    kCGRenderingIntentRelativeColorimetric, //相对比色模式
    kCGRenderingIntentPerceptual,   //压缩色域模式
    kCGRenderingIntentSaturation   //转换色域模式
};
*/
CGColorRef __nullable CGColorCreateCopyByMatchingToColorSpace(cg_nullable CGColorSpaceRef, CGColorRenderingIntent intent, CGColorRef cg_nullable color, __nullable CFDictionaryRef options);
//内存引用+1
CGColorRef cg_nullable CGColorRetain(CGColorRef cg_nullable color);
//内存引用-1
void CGColorRelease(CGColorRef cg_nullable color);
//比较两个颜色引用是否相同
bool CGColorEqualToColor(CGColorRef cg_nullable color1, CGColorRef cg_nullable color2);
//获取颜色内容的色彩描述值个数 包括alpha通道
size_t CGColorGetNumberOfComponents(CGColorRef cg_nullable color);
//获取颜色色彩描述值数组
CGFloat * __nullable CGColorGetComponents(CGColorRef cg_nullable color);
//获取颜色的透明度
CGFloat CGColorGetAlpha(CGColorRef cg_nullable color);
//获取颜色的色彩空间
CGColorSpaceRef __nullable CGColorGetColorSpace(CGColorRef cg_nullable color);
//获取与此颜色相关的模型
CGPatternRef __nullable CGColorGetPattern(CGColorRef cg_nullable color);
//获取CGColorRef类在CoreGraphics框架中的id
CFTypeID CGColorGetTypeID(void);
```

### 三、关于CGColorSpace相关方法解析

    CGColorSpace用来描述色彩空间，其中方法解析如下：

```objectivec
//创建一个基于设备的黑白色彩空间
CGColorSpaceRef cg_nullable CGColorSpaceCreateDeviceGray(void);
//创建一个基于设备的RGB色彩空间
CGColorSpaceRef cg_nullable CGColorSpaceCreateDeviceRGB(void);
//创建一个基于设备的CMYK色彩空间
CGColorSpaceRef cg_nullable CGColorSpaceCreateDeviceCMYK(void);
//创建一个经过校准的黑白色彩空间
CGColorSpaceRef __nullable CGColorSpaceCreateCalibratedGray(const CGFloat whitePoint[3], const CGFloat blackPoint[3], CGFloat gamma);
//创建一个经过校准的RGB色彩空间
CGColorSpaceRef __nullable CGColorSpaceCreateCalibratedRGB(const CGFloat whitePoint[3], const CGFloat blackPoint[3], const CGFloat gamma[3], const CGFloat matrix[9]);
//创建一个经过校准的LAB色彩空间
CGColorSpaceRef __nullable CGColorSpaceCreateLab(const CGFloat whitePoint[3], const CGFloat blackPoint[3], const CGFloat range[4]);
//使用ICC文件创建ICC-based色彩空间
CGColorSpaceRef __nullable CGColorSpaceCreateWithICCProfile(CFDataRef cg_nullable data);
CGColorSpaceRef __nullable CGColorSpaceCreateICCBased(size_t nComponents, const CGFloat * __nullable range, CGDataProviderRef cg_nullable profile, CGColorSpaceRef __nullable alternate);
//使用索引创建色彩空间
CGColorSpaceRef __nullable CGColorSpaceCreateIndexed(CGColorSpaceRef cg_nullable baseSpace, size_t lastIndex, const unsigned char * cg_nullable colorTable);
//通过名称创建色彩空间
/*
标准的黑白色彩空间
CFStringRef kCGColorSpaceGenericGray;
标准的RGB色彩空间
const CFStringRef kCGColorSpaceGenericRGB;
标准的CMYK色彩空间
CFStringRef kCGColorSpaceGenericCMYK;
displatP3色彩空间
CFStringRef kCGColorSpaceDisplayP3;
LinearRGB色彩空间
CFStringRef kCGColorSpaceGenericRGBLinear;
Adobe RGB 1998 版本的色彩空间
CFStringRef kCGColorSpaceAdobeRGB1998;
黑白色彩空间 设置伽马值为2.2
CFStringRef kCGColorSpaceGenericGrayGamma2_2;
XYZ色彩空间
CFStringRef kCGColorSpaceGenericXYZ;
ACEScg色彩空间
kCGColorSpaceACESCGLinear;
ITU-R Recommendation BT.709 色彩空间
kCGColorSpaceITUR_709;
ITU-R Recommendation BT.2020色彩空间
kCGColorSpaceITUR_2020;
RGB色彩空间
kCGColorSpaceROMMRGB;
DCI P3 色彩空间
kCGColorSpaceDCIP3;
扩展的sRGB色彩空间
kCGColorSpaceExtendedSRGB;
线性sRGB色彩空间
kCGColorSpaceLinearSRGB;
扩展的线性sRGB扩展空间
CFStringRef kCGColorSpaceExtendedLinearSRGB;
扩展的黑白色彩空间
CFStringRef kCGColorSpaceExtendedGray;
扩展的线性黑白色彩空间
CFStringRef kCGColorSpaceLinearGray;
扩展的Generic Gray 2.2色彩空间
CFStringRef kCGColorSpaceExtendedLinearGray;
*/
CGColorSpaceRef __nullable CGColorSpaceCreateWithName(CFStringRef cg_nullable name);
//内存引用计数+1
CGColorSpaceRef cg_nullable CGColorSpaceRetain(CGColorSpaceRef cg_nullable space);
//内存引用计数-1
void CGColorSpaceRelease(CGColorSpaceRef cg_nullable space);
//进行色彩空间的复制
CFStringRef __nullable CGColorSpaceCopyName(CGColorSpaceRef cg_nullable space);
//获取CGColorRef类在CoreGraphics框架中的id
CFTypeID CGColorSpaceGetTypeID(void);
//获取色彩空间颜色值参数个数
CGColorSpaceGetNumberOfComponents(CGColorSpaceRef cg_nullable space);
//获取色彩空间模式
/*
typedef CF_ENUM (int32_t,  CGColorSpaceModel) {
    kCGColorSpaceModelUnknown = -1,  未知模式
    kCGColorSpaceModelMonochrome,    单色色彩空间模式
    kCGColorSpaceModelRGB,           RGB色彩空间模式
    kCGColorSpaceModelCMYK,          CMYK色彩空间模式
    kCGColorSpaceModelLab,           LAB色彩空间模式
    kCGColorSpaceModelDeviceN,       设备色彩空间模式
    kCGColorSpaceModelIndexed,       引用色彩空间模式
    kCGColorSpaceModelPattern        模型色彩空间模式
};

*/
CGColorSpaceModel CGColorSpaceGetModel(CGColorSpaceRef cg_nullable space);
```

相同的图像，使用不同的色彩空间进行渲染，得到的结果可能大不一样，例如如下代码修改图片的色彩空间：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    CGImageRef image = CGImageCreateCopyWithColorSpace([UIImage imageNamed:@"image"].CGImage, CGColorSpaceCreateWithName(kCGColorSpaceGenericRGB));
    CGImageRef image2 = CGImageCreateCopyWithColorSpace([UIImage imageNamed:@"image"].CGImage, CGColorSpaceCreateWithName(kCGColorSpaceROMMRGB));
    UIImageView * imageView = [[UIImageView alloc]initWithImage:[UIImage imageWithCGImage:image]];
    UIImageView * imageView2 = [[UIImageView alloc]initWithImage:[UIImage imageWithCGImage:image2]];
    imageView.frame = CGRectMake(100, 100, 200, 200);
    imageView2.frame = CGRectMake(100, 300, 200, 200);
    [self.view addSubview:imageView];
    [self.view addSubview:imageView2];
}
```

效果如下：

原图如下：

![](http://static.oschina.net/uploads/space/2016/1020/164246_R3qx_2340880.png)

模拟器运行如下：

![](http://static.oschina.net/uploads/space/2016/1020/164659_1mRS_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

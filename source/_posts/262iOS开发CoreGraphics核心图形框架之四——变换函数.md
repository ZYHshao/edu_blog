---
title: iOS开发CoreGraphics核心图形框架之四——变换函数
date: 2016-10-30
categories: iOS逻辑初窥
tags: []
---
## iOS开发CoreGraphics核心图形框架之四——变换函数

### 一、引言

    在上一篇博客中，介绍了有关CGContext相关操作方法，其中可以直接调用一些方法来进行所绘制图形的平移，缩放，翻转等变换。对于图形了几何变换，开发者也可以采用另一种方式实现，CoreGraphics框架中提供了CGAffineTransform结构体，这个结构体中定义了图形变换的相关信息。

关于CGContext的相关内如博地址客如下：[https://my.oschina.net/u/2340880/blog/759070](https://my.oschina.net/u/2340880/blog/759070)。

### 二、使用CGAffineTransform相关函数进行绘制图形的几何变换

    CGAffineTransform中定义的方法即意义列举如下：

```objectivec
//创建标准的变换矩阵
CGAffineTransform CGAffineTransformIdentity;
//手动创建变换矩阵
CGAffineTransform CGAffineTransformMake(CGFloat a, CGFloat b, CGFloat c, CGFloat d, CGFloat tx, CGFloat ty);
//创建平移变换
CGAffineTransform CGAffineTransformMakeTranslation(CGFloat tx, CGFloat ty);
//创建缩放变换
CGAffineTransform CGAffineTransformMakeScale(CGFloat sx, CGFloat sy);
//创建旋转变换
CGAffineTransform CGAffineTransformMakeRotation(CGFloat angle);
//判断某个变化是否是来自标准矩阵的变换
bool CGAffineTransformIsIdentity(CGAffineTransform t);
//对某个变换矩阵进行平移变换
CGAffineTransform CGAffineTransformTranslate(CGAffineTransform t, CGFloat tx, CGFloat ty);
//对某个变换矩阵进行缩放变换
CGAffineTransform CGAffineTransformScale(CGAffineTransform t, CGFloat sx, CGFloat sy);
//对某个变换矩阵进行旋转变换
CGAffineTransform CGAffineTransformRotate(CGAffineTransform t, CGFloat angle);
//对某个变换矩阵进行翻转变换
CGAffineTransform CGAffineTransformInvert(CGAffineTransform t);
//对两个变换矩阵进行计算
CGAffineTransform CGAffineTransformConcat(CGAffineTransform t1, CGAffineTransform t2);
//比较两个变换矩阵是否相同
bool CGAffineTransformEqualToTransform(CGAffineTransform t1, CGAffineTransform t2);
//获取应用变换后某点的坐标
CGPoint CGPointApplyAffineTransform(CGPoint point, CGAffineTransform t);
//获取应用变换后某个区域的尺寸
CGSize CGSizeApplyAffineTransform(CGSize size, CGAffineTransform t);
//获取应用变换后某个区域的位置和尺寸
CGRect CGRectApplyAffineTransform(CGRect rect, CGAffineTransform t);
```

上述变换方法可以直接作用于View，示例如下：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    UIImageView * view = [[UIImageView alloc]initWithFrame:CGRectMake(100, 100, 200, 200)];
    view.backgroundColor =[UIColor whiteColor];
    view.image = [UIImage imageNamed:@"image"];
    view.transform = CGAffineTransformRotate(CGAffineTransformIdentity, M_PI_4);
    [self.view addSubview:view];
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

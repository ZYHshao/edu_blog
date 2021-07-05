---
title: iOS开发CoreAnimation解读之六——CATransform3D变换的应用
date: 2015-12-06
categories: iOS逻辑初窥
tags: []
---
## iOS开发CoreAnimation解读之五——CATransform3D变换的应用

### 一、引言

        CATransform3D定义了一个变化矩阵，通过对矩阵参数的设置，我们可以改变layer的一些属性，这个属性的改变，可以产生动画的效果。首先，CATransform3D定义了一个4*4的矩阵，如下：

```
struct CATransform3D
{
  CGFloat m11, m12, m13, m14;
  CGFloat m21, m22, m23, m24;
  CGFloat m31, m32, m33, m34;
  CGFloat m41, m42, m43, m44;
};
```

从m11到m44定义的含义如下：

m11：x轴方向进行缩放

m12：和m21一起决定z轴的旋转

m13:和m31一起决定y轴的旋转

m14:

m21:和m12一起决定z轴的旋转

m22:y轴方向进行缩放

m23:和m32一起决定x轴的旋转

m24:

m31:和m13一起决定y轴的旋转

m32:和m23一起决定x轴的旋转

m33:z轴方向进行缩放

m34:透视效果m34= -1/D，D越小，透视效果越明显，必须在有旋转效果的前提下，才会看到透视效果

m41:x轴方向进行平移

m42:y轴方向进行平移

m43:z轴方向进行平移

m44:初始为1

### 二、CATransform3D中的属性和方法

```
//初始化一个transform3D对象，不做任何变换
const CATransform3D CATransform3DIdentity;
//判断一个transform3D对象是否是初始化的对象
bool CATransform3DIsIdentity (CATransform3D t);
//比较两个transform3D对象是否相同
bool CATransform3DEqualToTransform (CATransform3D a, CATransform3D b);
//将两个 transform3D对象变换属性进行叠加，返回一个新的transform3D对象
CATransform3D CATransform3DConcat (CATransform3D a, CATransform3D b);
```

#### 1、平移变换

```
//返回一个平移变换的transform3D对象 tx，ty，tz对应x，y，z轴的平移
CATransform3D CATransform3DMakeTranslation (CGFloat tx, CGFloat ty, CGFloat tz);
//在某个transform3D变换的基础上进行平移变换，t是上一个transform3D，其他参数同上
CATransform3D CATransform3DTranslate (CATransform3D t, CGFloat tx, CGFloat ty, CGFloat tz);
```

例如：

```
    UIImageView * imageView = [[UIImageView alloc]initWithFrame:CGRectMake(100, 100, 100, 100)];
    imageView.image = [UIImage imageNamed:@"屏幕快照 2015-12-06 下午3.27.15.png"];
    [self.view addSubview:imageView];
    
    UIImageView * newImageView = [[UIImageView alloc]initWithFrame:CGRectMake(100, 100, 100, 100)];
    newImageView.image=[UIImage imageNamed:@"屏幕快照 2015-12-06 下午3.27.15.png"];
    [self.view addSubview:newImageView];
    CATransform3D trans = CATransform3DMakeTranslation(10, 200, 0);
    newImageView.layer.transform =trans;
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/1206/152731_iQIU_2340880.png)

#### 2、缩放变换

```
//x，y，z分别对应x轴，y轴，z轴的缩放比例
CATransform3D CATransform3DMakeScale (CGFloat sx, CGFloat sy, CGFloat sz);
//在一个transform3D变换的基础上进行缩放变换，其他参数同上
CATransform3D CATransform3DScale (CATransform3D t, CGFloat sx, CGFloat sy, CGFloat sz);
```

例如：

```
UIImageView * imageView = [[UIImageView alloc]initWithFrame:CGRectMake(100, 100, 100, 100)];
    imageView.image = [UIImage imageNamed:@"屏幕快照 2015-12-06 下午3.27.15.png"];
    [self.view addSubview:imageView];
    
    UIImageView * newImageView = [[UIImageView alloc]initWithFrame:CGRectMake(100, 300, 100, 100)];
    newImageView.image=[UIImage imageNamed:@"屏幕快照 2015-12-06 下午3.27.15.png"];
    [self.view addSubview:newImageView];
    CATransform3D trans = CATransform3DMakeScale(2, 1, 1);
    newImageView.layer.transform =trans;
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/1206/153159_APG9_2340880.png)

#### 3、旋转变换

```
//angle参数是旋转的角度，为弧度制 0-2π
//x，y，z决定了旋转围绕的中轴，取值为-1——1之间，例如（1，0，0）,则是绕x轴旋转（0.5，0.5，0），则是绕x轴与y轴中
//间45度为轴旋转,依次进行计算
CATransform3D CATransform3DMakeRotation (CGFloat angle, CGFloat x, CGFloat y, CGFloat z);
//在一个transform3D的基础上进行旋转变换，其他参数如上
CATransform3D CATransform3DRotate (CATransform3D t, CGFloat angle, CGFloat x, CGFloat y, CGFloat z);
```

例如：

```
UIImageView * imageView = [[UIImageView alloc]initWithFrame:CGRectMake(100, 100, 100, 100)];
    imageView.image = [UIImage imageNamed:@"屏幕快照 2015-12-06 下午3.27.15.png"];
    [self.view addSubview:imageView];
    
    UIImageView * newImageView = [[UIImageView alloc]initWithFrame:CGRectMake(100, 300, 100, 100)];
    newImageView.image=[UIImage imageNamed:@"屏幕快照 2015-12-06 下午3.27.15.png"];
    [self.view addSubview:newImageView];
    CATransform3D trans = CATransform3DMakeRotation(M_PI/2, 0, 0, 1);
    newImageView.layer.transform =trans;
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/1206/153746_NIUC_2340880.png)

另外，当我们有垂直于z轴的旋转分量时，设置m34的值可以增加透视效果，也可以理解为景深效果，例如：

```
    UIImageView * imageView = [[UIImageView alloc]initWithFrame:CGRectMake(100, 100, 100, 100)];
    imageView.image = [UIImage imageNamed:@"屏幕快照 2015-12-06 下午3.27.15.png"];
    imageView.layer.transform = CATransform3DMakeRotation(M_PI/4, 0, 1, 0);
    [self.view addSubview:imageView];
    
    UIImageView * newImageView = [[UIImageView alloc]initWithFrame:CGRectMake(100, 300, 100, 100)];
    newImageView.image=[UIImage imageNamed:@"屏幕快照 2015-12-06 下午3.27.15.png"];
    [self.view addSubview:newImageView];
    CATransform3D trans = CATransform3DIdentity;
    trans.m34 = -1/100.0;
    trans = CATransform3DRotate(trans, M_PI/4, 0, 1, 0);  
    newImageView.layer.transform =trans;
```

两个imageView都进行了y轴的旋转变换，第二个有透视效果，第一个没有，运行如下：

![](http://static.oschina.net/uploads/space/2015/1206/154348_3kub_2340880.png)

#### 4、旋转翻转变换

```
//将一个旋转的效果进行翻转 
CATransform3D CATransform3DInvert (CATransform3D t);
```

例如：

```
    UIImageView * imageView = [[UIImageView alloc]initWithFrame:CGRectMake(100, 100, 100, 100)];
    imageView.image = [UIImage imageNamed:@"屏幕快照 2015-12-06 下午3.27.15.png"];
    imageView.layer.transform = CATransform3DMakeRotation(M_PI/4, 0, 0, 1);
    [self.view addSubview:imageView];
    
    UIImageView * newImageView = [[UIImageView alloc]initWithFrame:CGRectMake(100, 300, 100, 100)];
    newImageView.image=[UIImage imageNamed:@"屏幕快照 2015-12-06 下午3.27.15.png"];
    [self.view addSubview:newImageView];
    CATransform3D trans = CATransform3DMakeRotation(M_PI/4, 0, 0, 1);
    trans = CATransform3DInvert(trans);
    
    newImageView.layer.transform =trans;
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/1206/155148_RdVk_2340880.png)

#### 5、CATransform3D与CGAffineTransform的转换

        CGAffineTransform是UIKit框架中一个用于变换的矩阵，其作用与CATransform类似，只是其可以直接作用于View，而不用作用于layer，这两个矩阵也可以进行转换，方法如下：

```
//将一个CGAffinrTransform转化为CATransform3D
CATransform3D CATransform3DMakeAffineTransform (CGAffineTransform m);
//判断一个CATransform3D是否可以转换为CAAffineTransform
bool CATransform3DIsAffine (CATransform3D t);
//将CATransform3D转换为CGAffineTransform
CGAffineTransform CATransform3DGetAffineTransform (CATransform3D t);
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

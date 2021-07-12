---
title: iOS第三方星级视图控件——HCSStarRatingView
date: 2016-07-06
categories: iOS第三方库
tags: []
---
## iOS第三方星级视图控件——HCSStarRatingView

        HCStarRatingView是一款十分小巧的星级视图控件，其通过原生画图的方式来渲染星级视图页面，同时，其也支持开发者对星级图片的自定义操作。

        HCStarRatingView的git地址如下：[https://github.com/hsousa/HCSStarRatingView](https://github.com/hsousa/HCSStarRatingView)。

        HCStarRatingView的使用十分简单，示例如下：

```objectivec
    HCSStarRatingView * starView = [[HCSStarRatingView alloc]initWithFrame:CGRectMake(20, 100, 280, 50)];
    starView.tintColor = [UIColor redColor];
    [starView addTarget:self action:@selector(didChange:) forControlEvents:UIControlEventValueChanged];
    [self.view addSubview:starView];
```

效果如下图：

![](http://static.oschina.net/uploads/space/2016/0706/104250_45tC_2340880.png)

开发者也对其进行一些自定义的设置，列举如下：

```objectivec
//设置最大值
@property (nonatomic) IBInspectable NSUInteger maximumValue;
//设置最小值
@property (nonatomic) IBInspectable CGFloat minimumValue;
//星级视图当前值
@property (nonatomic) IBInspectable CGFloat value;
//星星间间距
@property (nonatomic) IBInspectable CGFloat spacing;
//是否允许选择半星
@property (nonatomic) IBInspectable BOOL allowsHalfStars;
//是否是否允许精确选择 可以根据选择位置进行精确
@property (nonatomic) IBInspectable BOOL accurateHalfStars;
//是否连续调用回调方法 如果设置为YES 则在手指拖动时 会持续调用回调方法 如果设置为NO，则只有拖动结束后才调用回调
@property (nonatomic) IBInspectable BOOL continuous;
//是否允许成为第一响应
@property (nonatomic) BOOL shouldBecomeFirstResponder;
//添加手势时使用
@property (nonatomic, copy) HCSStarRatingViewShouldBeginGestureRecognizerBlock shouldBeginGestureRecognizerBlock;
//自定义星星视图UI
//设置空星的图片
@property (nonatomic, strong) IBInspectable UIImage *emptyStarImage;
//设置半星的图片
@property (nonatomic, strong) IBInspectable UIImage *halfStarImage;
//设置全星时的图片
@property (nonatomic, strong) IBInspectable UIImage *filledStarImage;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: OS X开发：NSProgressIndicator进度指示器控件
date: 2017-07-21
categories: macOS开发
tags: []
---
## OS X开发：NSProgressIndicator进度指示器控件

    NSProgressIndicator是OS X平台上的活动指示器控件，开发者可以设置圆环样式和进度条样式两种。

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    NSProgressIndicator * progressIndicator = [[NSProgressIndicator alloc]initWithFrame:CGRectMake(30, 100, 200, 10)];
    //设置是精准的进度条还是模糊的指示器
    progressIndicator.indeterminate = YES;
    //是否贝塞尔风格
    progressIndicator.bezeled = YES;
    //设置控制器尺寸
    progressIndicator.controlSize = NSControlSizeSmall;
    //设置当前进度
    progressIndicator.doubleValue = 5;
    //设置风格
    progressIndicator.style = NSProgressIndicatorBarStyle;
    //设置是否当动画停止时隐藏
    progressIndicator.displayedWhenStopped = YES;
    [self.view addSubview:progressIndicator];
}

```

效果如图：

![](https://static.oschina.net/uploads/space/2017/0721/101837_KbZG_2340880.png)

NSProgressIndicator类中属性方法解析如下：

```objectivec
//设置是否是模糊模式 牧户模式下，不显示具体的进度，通过动画提示用户正在加载
@property (getter=isIndeterminate) BOOL indeterminate;    
//设置是否贝塞尔风格
@property (getter=isBezeled) BOOL bezeled;
//指示器的控制色
@property NSControlTint controlTint;
//指示器的尺寸设置
/*
typedef NS_ENUM(NSUInteger, NSControlSize) {
    NSControlSizeRegular,//标准
    NSControlSizeSmall,//小
    NSControlSizeMini,//迷你
};
*/
@property NSControlSize controlSize;
//设置当前进度值
@property double doubleValue;
//设置进度值增量，即原始值夹着delta值
- (void)incrementBy:(double)delta;
//进度条最小值
@property double minValue;
//进度条最大值
@property double maxValue;
//是否在多线程中执行动画
@property BOOL usesThreadedAnimation;
//开始动画
- (void)startAnimation:(nullable id)sender;
//结束动画
- (void)stopAnimation:(nullable id)sender;
//设置风格
/*
typedef NS_ENUM(NSUInteger, NSProgressIndicatorStyle) {
    NSProgressIndicatorBarStyle = 0,     //进度条风格
    NSProgressIndicatorSpinningStyle = 1 //风火轮风格
};
*/
@property NSProgressIndicatorStyle style;
//设置动画停止时进度条是否依然显示
@property (getter=isDisplayedWhenStopped) BOOL displayedWhenStopped;
```

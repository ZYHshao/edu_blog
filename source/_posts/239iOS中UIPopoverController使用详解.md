---
title: iOS中UIPopoverController使用详解
date: 2016-07-13
categories: iOS之UI控件
tags: []
---
## iOS中UIPopoverController使用详解

### 一、引言

        UIPopoverController是Pad设备中常用的一种视图控制器，其在UI表现上为在当前视图控制器上面弹出一个子视图控制器，通常用来展示交互列表。示例如下图：

![](http://static.oschina.net/uploads/space/2016/0713/114232_Wju6_2340880.png)

UIPopoverController只能用于iPad，在要兼容iPad和iPhone的项目中，需要根据设备类型使用两套代码。在iOS8之后，系统提供了UIPresentationController来代替她，UIPresentationController可以兼容iPhone与iPad。

### 二、UIPopoverController的使用详解

        首先UIPopoverController是一个容器控制器，其中需要承载一个ViewControler作为内容视图。UIPopoverController使用如下初始化方法创建：

```objectivec
//创建视图控制器的方法 通过一个内容视图控制器创建
- (instancetype)initWithContentViewController:(UIViewController *)viewController;
```

创建出控制器后，调用如下方法可以将控制器弹出：

```objectivec
//这个方法将控制器以一个CGRect区域为基准弹出
/*
UIPopoverArrowDirection为箭头出现的方向
typedef NS_OPTIONS(NSUInteger, UIPopoverArrowDirection) {
    UIPopoverArrowDirectionUp = 1UL << 0,//上
    UIPopoverArrowDirectionDown = 1UL << 1,//下
    UIPopoverArrowDirectionLeft = 1UL << 2,//左
    UIPopoverArrowDirectionRight = 1UL << 3,//右
    UIPopoverArrowDirectionAny = UIPopoverArrowDirectionUp | UIPopoverArrowDirectionDown | UIPopoverArrowDirectionLeft | UIPopoverArrowDirectionRight,//任意方向
    UIPopoverArrowDirectionUnknown = NSUIntegerMax//未知
};
*/
//view参数为选择要在那个View视图上弹出 animated参数设置是否带动画
- (void)presentPopoverFromRect:(CGRect)rect inView:(UIView *)view permittedArrowDirections:(UIPopoverArrowDirection)arrowDirections animated:(BOOL)animated;
//以一个BarButtonItem为基准弹出 其余参数意义同上
- (void)presentPopoverFromBarButtonItem:(UIBarButtonItem *)item permittedArrowDirections:(UIPopoverArrowDirection)arrowDirections animated:(BOOL)animated;
```

UIPopoverController的相关设置方法如下：

```objectivec
//设置代理
@property (nullable, nonatomic, weak) id <UIPopoverControllerDelegate> delegate;
//设置内容视图控制器
@property (nonatomic, strong) UIViewController *contentViewController;
- (void)setContentViewController:(UIViewController *)viewController animated:(BOOL)animated;
//设置界面展示尺寸
@property (nonatomic) CGSize popoverContentSize;
- (void)setPopoverContentSize:(CGSize)size animated:(BOOL)animated;
//获取控制器当前是否正在展示
@property (nonatomic, readonly, getter=isPopoverVisible) BOOL popoverVisible;
//获取控制器箭头方向
@property (nonatomic, readonly) UIPopoverArrowDirection popoverArrowDirection;
//这个属性可以增强控制器的交互能力
/*
默认情况下，当视图控制器弹出时，点击界面上的其他位置，视图控制器会被隐藏 如果需要当视图控制爱弹出时界面上的其他控件依然可以进行用户交互，则需要将这些UI控件设置进这个数组中
*/
@property (nullable, nonatomic, copy) NSArray<__kindof UIView *> *passthroughViews;
//隐藏视图控制器的方法
- (void)dismissPopoverAnimated:(BOOL)animated;
//设置视图控制器的背景颜色
@property (nullable, nonatomic, copy) UIColor *backgroundColor NS_AVAILABLE_IOS(7_0);
//设置视图Margin
@property (nonatomic, readwrite) UIEdgeInsets popoverLayoutMargins NS_AVAILABLE_IOS(5_0);
//这个属性用于自定义PopoverController的UI展现 传入自定义的背景视图类
@property (nullable, nonatomic, readwrite, strong) Class popoverBackgroundViewClass NS_AVAILABLE_IOS(5_0);
```

### 三、自定义UI展现的UIPopoverController

        通过设置UIPopoverController对象的popoverBacjgroundViewClass属性可以将一个自定义的类作为控制器的背景视图，需要注意，此自定义的类必须继承自UIPopoverBackgroundView，并且子类必须覆写父类中的一些列方法，示例如下：

```objectivec
@interface MyView : UIPopoverBackgroundView
@end

@implementation MyView
//这个方法返回箭头宽度
+ (CGFloat)arrowBase{
    return 20;
}
//这个方法中返回内容视图的偏移
+(UIEdgeInsets)contentViewInsets{
    return UIEdgeInsetsMake(20, 20, 20, 20);
}
//这个方法返回箭头高度
+(CGFloat)arrowHeight{
    return 30;
}
//这个方法返回箭头的方向
-(UIPopoverArrowDirection)arrowDirection{
    return UIPopoverArrowDirectionUp;
}
//这个在设置箭头方向时被调用 可以监听做处理
-(void)setArrowDirection:(UIPopoverArrowDirection)arrowDirection{
    
}
//这个方法在设置箭头偏移量时被调用 可以监听做处理
-(void)setArrowOffset:(CGFloat)arrowOffset{
    
}
//重写layout方法来来定义箭头样式
- (void)layoutSubviews
{
    [super layoutSubviews];
    CGSize arrowSize = CGSizeMake([[self class] arrowBase], [[self class] arrowHeight]);
    UIImage * image  = [self drawArrowImage:arrowSize];
    UIImageView * imageView = [[UIImageView alloc]initWithImage:image];
    imageView.frame = CGRectMake(0, 0.0f, arrowSize.width, arrowSize.height);
    [self addSubview:imageView];
}
//这个方法中进行背景色的设置
- (instancetype)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        self.backgroundColor = [UIColor redColor];
    }
    return self;
}
- (instancetype)init
{
    self = [super init];
    if (self) {
        
    }
    return self;
}
//返回值决定是否渲染阴影
+(BOOL)wantsDefaultContentAppearance{
    return NO;
}
//画箭头方法
- (UIImage *)drawArrowImage:(CGSize)size
{
    UIGraphicsBeginImageContextWithOptions(size, NO, 0);
    CGContextRef ctx = UIGraphicsGetCurrentContext();
    [[UIColor clearColor] setFill];
    CGContextFillRect(ctx, CGRectMake(0.0f, 0.0f, size.width, size.height));
    CGMutablePathRef arrowPath = CGPathCreateMutable();
    CGPathMoveToPoint(arrowPath, NULL, (size.width/2.0f), 0.0f);
    CGPathAddLineToPoint(arrowPath, NULL, size.width, size.height);
    CGPathAddLineToPoint(arrowPath, NULL, 0.0f, size.height);
    CGPathCloseSubpath(arrowPath);
    CGContextAddPath(ctx, arrowPath);
    CGPathRelease(arrowPath);
    UIColor *fillColor = [UIColor yellowColor];
    CGContextSetFillColorWithColor(ctx, fillColor.CGColor);
    CGContextDrawPath(ctx, kCGPathFill);
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return image;
}
@end
```

### 四、UIPopoverPresentationController应用解析

    UIPopoverPresentationController是iOS8后系统新引入的控制器，其可以很好的兼容iPhone与iPad。UIPopoverPresentationContriller的使用需要和UIViewController结合进行，使用过程示例如下：

```objectivec
UITableViewController tabCon = [[UITableViewController alloc]initWithStyle:UITableViewStylePlain];
//设置跳转模式为popover模式
tabCon.modalPresentationStyle = UIModalPresentationPopover;
//获取到UIPopoverPresentationController对象
UIPopoverPresentationController* con = tabCon.popoverPresentationController;
//设置弹出的基准视图
con.sourceView = self.view;
[self presentViewController:tabCon animated:YES completion:nil];
```

UIPopoverPresentationController中属性如下：

```objectivec
//设置代理
@property (nullable, nonatomic, weak) id <UIPopoverPresentationControllerDelegate> delegate;
//设置允许的箭头方向
@property (nonatomic, assign) UIPopoverArrowDirection permittedArrowDirections;
//设置基准视图或者区域
@property (nullable, nonatomic, strong) UIView *sourceView;
@property (nonatomic, assign) CGRect sourceRect;
//设置是否覆盖基准视图区域
@property (nonatomic, assign) BOOL canOverlapSourceViewRect NS_AVAILABLE_IOS(9_0);
//设置基准BarButtonItem
@property (nullable, nonatomic, strong) UIBarButtonItem *barButtonItem;
//设置可以进行用户交互的视图
@property (nullable, nonatomic, copy) NSArray<UIView *> *passthroughViews;
//设置背景颜色
@property (nullable, nonatomic, copy) UIColor *backgroundColor;
//设置Margin
@property (nonatomic, readwrite) UIEdgeInsets popoverLayoutMargins;
//设置自定义视图
@property (nullable, nonatomic, readwrite, strong) Class <UIPopoverBackgroundViewMethods> popoverBackgroundViewClass;
```

UIPopoverPresentationControllerDelegate中的方法如下：

```objectivec
//控制器将要弹出时调用
- (void)prepareForPopoverPresentation:(UIPopoverPresentationController *)popoverPresentationController;
//控制器将要消失时调用
- (BOOL)popoverPresentationControllerShouldDismissPopover:(UIPopoverPresentationController *)popoverPresentationController;
//控制器已经消失时调用
- (void)popoverPresentationControllerDidDismissPopover:(UIPopoverPresentationController *)popoverPresentationController;
//控制器接收到弹出消息时调用
- (void)popoverPresentationController:(UIPopoverPresentationController *)popoverPresentationController willRepositionPopoverToRect:(inout CGRect *)rect inView:(inout UIView  * __nonnull * __nonnull)view;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

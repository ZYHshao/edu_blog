---
title: iOS好用的第三方侧边栏控件——MMDrawerController
date: 2016-06-29
categories: iOS第三方库
tags: []
---
## iOS好用的第三方侧边栏控件——MMDrawerController

### 一、引言

        很多应用程序都采用了侧边栏这样的界面结构，MMDrawerController是一个轻量级的侧边栏抽屉控件，其支持左侧抽屉和右侧抽屉，可以很好的支持导航控制器，并且支持开发者对手势和动画进行自定义。MMDrawerController的git地址如下：

[https://github.com/mutualmobile/MMDrawerController](https://github.com/mutualmobile/MMDrawerController)。

![](http://static.oschina.net/uploads/space/2016/0629/142050_1Em1_2340880.png)

### 二、MMDrawerController的使用及相关设置

        MMDrawerController的使用十分简单，只需将中心视图控制器和左边栏视图控制器传入初始化方法即可完成MMDrawerController的创建。示例代码如下：

```objectivec
    UIViewController * leftViewController = [[UIViewController alloc]init];
    leftViewController.view.backgroundColor = [UIColor redColor];
    UIViewController * rightViewController = [[UIViewController alloc]init];
    rightViewController.view.backgroundColor = [UIColor greenColor];
    ViewController * centerViewController = [[ViewController alloc]init];
    centerViewController.view.backgroundColor = [UIColor blueColor];
    //创建控件
    MMDrawerController * rootController = [[MMDrawerController alloc]initWithCenterViewController:centerViewController leftDrawerViewController:leftViewController rightDrawerViewController:rightViewController];
```

MMDrawerController中还提供了两个方法供开发者创建单侧边栏，如下：

```objectivec
//只创建带左侧边栏的视图控制器
-(id)initWithCenterViewController:(UIViewController *)centerViewController leftDrawerViewController:(UIViewController *)leftDrawerViewController;
//只创建带右侧边栏的视图控制器
-(id)initWithCenterViewController:(UIViewController *)centerViewController rightDrawerViewController:(UIViewController *)rightDrawerViewController;
```

MMDrawerController中也提供了许多属性和方法供开发者进行自定义的设置，其中可用属性解析如下：

```objectivec
//设置左侧边栏的最大宽度 默认280
@property (nonatomic, assign) CGFloat maximumLeftDrawerWidth;
//设置右侧边栏的最大宽度 默认280
@property (nonatomic, assign) CGFloat maximumRightDrawerWidth;
//这个是一个只读属性，用于获取可见的左侧边栏宽度
@property (nonatomic, assign, readonly) CGFloat visibleLeftDrawerWidth;
//这个是一个只读属性，用于获取可见的右侧边栏宽度
@property (nonatomic, assign, readonly) CGFloat visibleRightDrawerWidth;
//动画速度，这个参数的意义是每秒移动多少单位 默认为800/s
@property (nonatomic, assign) CGFloat animationVelocity;
//设置是否允许回弹效果，如果设置为YES，当使用手势进行侧边栏的开启时会出现回弹效果
@property (nonatomic, assign) BOOL shouldStretchDrawer;
//获取当前开启的侧边栏类型，MMDrawerSide枚举如下：
/*
typedef NS_ENUM(NSInteger,MMDrawerSide){
    MMDrawerSideNone = 0,//无侧边栏
    MMDrawerSideLeft,    //左侧边栏
    MMDrawerSideRight,   //右侧边栏
};
*/
@property (nonatomic, assign, readonly) MMDrawerSide openSide;
//开启侧边栏的手势模式 MMOpenDrawerGestureMode枚举意义如下
/*
typedef NS_OPTIONS(NSInteger, MMOpenDrawerGestureMode) {
    //没有手势 此模式为默认模式
    MMOpenDrawerGestureModeNone                     = 0,
    //在导航栏上拖动时可以打开侧边栏
    MMOpenDrawerGestureModePanningNavigationBar     = 1 << 1,
    //在中心视图控制器的视图上拖动时可以打开侧边栏
    MMOpenDrawerGestureModePanningCenterView        = 1 << 2,
    //在中心视图控制器的视图边缘20个单位内拖动时可以打开侧边栏
    MMOpenDrawerGestureModeBezelPanningCenterView   = 1 << 3,
    //自定义手势 需配合自定义手势的方法使用
    MMOpenDrawerGestureModeCustom                   = 1 << 4,
    //所有模式兼容
    MMOpenDrawerGestureModeAll                      =   MMOpenDrawerGestureModePanningNavigationBar     |
                                                        MMOpenDrawerGestureModePanningCenterView        |
                                                        MMOpenDrawerGestureModeBezelPanningCenterView   |
                                                        MMOpenDrawerGestureModeCustom,
};
*/
@property (nonatomic, assign) MMOpenDrawerGestureMode openDrawerGestureModeMask;
//关闭侧边栏的手势模式 MMCloseDrawerGestureMode枚举的意义如下
/*
typedef NS_OPTIONS(NSInteger, MMCloseDrawerGestureMode) {
    //没有关闭手势
    MMCloseDrawerGestureModeNone                    = 0,
    //在导航栏上拖动时可以关闭侧边栏
    MMCloseDrawerGestureModePanningNavigationBar    = 1 << 1,
    //在中心视图控制器上推动时可以关闭侧边栏
    MMCloseDrawerGestureModePanningCenterView       = 1 << 2,
    //在中心视图控制器边缘20单位内拖动是可以关闭侧边栏
    MMCloseDrawerGestureModeBezelPanningCenterView  = 1 << 3,
    //点击导航栏时可以关闭侧边栏
    MMCloseDrawerGestureModeTapNavigationBar        = 1 << 4,
    //点击中心视图控制器视图时可以关闭侧边栏
    MMCloseDrawerGestureModeTapCenterView           = 1 << 5,
    //在侧边栏视图上拖动时可以关闭侧边栏
    MMCloseDrawerGestureModePanningDrawerView       = 1 << 6,
    //自定义关闭手势，需要和自定义手势的方法结合使用
    MMCloseDrawerGestureModeCustom                  = 1 << 7,
    //所有模式兼容
    MMCloseDrawerGestureModeAll                     =   MMCloseDrawerGestureModePanningNavigationBar    |
                                                        MMCloseDrawerGestureModePanningCenterView       |
                                                        MMCloseDrawerGestureModeBezelPanningCenterView  |
                                                        MMCloseDrawerGestureModeTapNavigationBar        |
                                                        MMCloseDrawerGestureModeTapCenterView           |
                                                        MMCloseDrawerGestureModePanningDrawerView       |
                                                        MMCloseDrawerGestureModeCustom,
};

*/
@property (nonatomic, assign) MMCloseDrawerGestureMode closeDrawerGestureModeMask;
//设置侧边栏显示时的中心视图控制器的用户交互规则 MMDrawerOpenCenterInteractionMode枚举意义如下
/*
typedef NS_ENUM(NSInteger, MMDrawerOpenCenterInteractionMode) {
    //中心视图控制器不能进行用户交互 默认为此枚举
    MMDrawerOpenCenterInteractionModeNone,
    //中心视图控制器完全可以进行用户交互
    MMDrawerOpenCenterInteractionModeFull,
    //中心视图控制器只有导航可以进行用户交互
    MMDrawerOpenCenterInteractionModeNavigationBarOnly,
};
*/
@property (nonatomic, assign) MMDrawerOpenCenterInteractionMode centerHiddenInteractionMode;
//设置是否显示阴影效果
@property (nonatomic, assign) BOOL showsShadow;
//设置是否显示状态栏的自定义视图 只有在iOS7之后可用
@property (nonatomic, assign) BOOL showsStatusBarBackgroundView;
//设置状态栏视图颜色 只有在iOS7之后可用
@property (nonatomic, strong) UIColor * statusBarViewBackgroundColor;
```

相关方法解析如下：

```objectivec
//切换侧边栏的状态，drawerSide参数为要切换的侧边栏，animated设置是否有动画效果，completion会在切换完成后执行
//注意：如果在切换一个关着的侧边栏时，如果另一个侧边栏正在开启状态，则此方法不会有任何效果
-(void)toggleDrawerSide:(MMDrawerSide)drawerSide animated:(BOOL)animated completion:(void(^)(BOOL finished))completion;
//关闭侧边栏
-(void)closeDrawerAnimated:(BOOL)animated completion:(void(^)(BOOL finished))completion;
//开启侧边栏
-(void)openDrawerSide:(MMDrawerSide)drawerSide animated:(BOOL)animated completion:(void(^)(BOOL finished))completion;
//更换中心视图控制器
-(void)setCenterViewController:(UIViewController *)centerViewController withCloseAnimation:(BOOL)closeAnimated completion:(void(^)(BOOL finished))completion;
-(void)setCenterViewController:(UIViewController *)newCenterViewController withFullCloseAnimation:(BOOL)fullCloseAnimated completion:(void(^)(BOOL finished))completion;
//设置左侧边栏最大宽度
-(void)setMaximumLeftDrawerWidth:(CGFloat)width animated:(BOOL)animated completion:(void(^)(BOOL finished))completion;
//设置右侧边栏最大宽度
-(void)setMaximumRightDrawerWidth:(CGFloat)width animated:(BOOL)animated completion:(void(^)(BOOL finished))completion;
//进行侧边栏的预览操作 默认预览距离为40个单位
-(void)bouncePreviewForDrawerSide:(MMDrawerSide)drawerSide completion:(void(^)(BOOL finished))completion;
//进行侧边栏的预览操作 可以设置预览距离
-(void)bouncePreviewForDrawerSide:(MMDrawerSide)drawerSide distance:(CGFloat)distance completion:(void(^)(BOOL finished))completion;
//这个方法用于进行视图侧边栏视图出现动画的自定义
-(void)setDrawerVisualStateBlock:(void(^)(MMDrawerController * drawerController, MMDrawerSide drawerSide, CGFloat percentVisible))drawerVisualStateBlock;
//这个方法用于设置当一个手势触发完成后的回调
-(void)setGestureCompletionBlock:(void(^)(MMDrawerController * drawerController, UIGestureRecognizer * gesture))gestureCompletionBlock;
//这个方法用于定义自定义的手势操作 要将开启侧边栏与关闭侧边栏的模式设置为MMOpenDrawerGestureModeCustom和MMCloseDrawerGestureModeCustom才有效
-(void)setGestureShouldRecognizeTouchBlock:(BOOL(^)(MMDrawerController * drawerController, UIGestureRecognizer * gesture, UITouch * touch))gestureShouldRecognizeTouchBlock;
```

对于自定义过渡动画的方法：

-(void)setDrawerVisualStateBlock:(void(^)(MMDrawerController * drawerController, MMDrawerSide drawerSide, CGFloat percentVisible))drawerVisualStateBlock;

回调block中会传递进来侧边栏显示完成的百分比，并且在侧边栏出现过程中，这个回调block会被不停刷新调用，开发者可以直接在其中对要过渡的属性进行设置，例如透明度的渐变动画，示例如下：

```objectivec
//进行自定义动画
    [rootController setDrawerVisualStateBlock:^(MMDrawerController *drawerController, MMDrawerSide drawerSide, CGFloat percentVisible) {
        UIViewController * sideDrawerViewController;
        if(drawerSide == MMDrawerSideLeft){
            sideDrawerViewController = drawerController.leftDrawerViewController;
        }
        else if(drawerSide == MMDrawerSideRight){
            sideDrawerViewController = drawerController.rightDrawerViewController;
        }
        [sideDrawerViewController.view setAlpha:percentVisible];
        
    }];
```

### 三、关于MMDrawerController的子类

        开发者如果有特殊的需求，也可以通过继承MMDrawerController来实现自己的侧边栏控制器类，MMDrawerController框架中提供了一个扩展，在编写MMDrawerController时，开发者可以导入MMDrawerController+Subclass.h文件，这个文件中提供了许多控制器的监听方法供开发者重写，解析如下：

```objectivec
//出现单击手势会回调的方法 如果要重写 必须调用父类的此方法
-(void)tapGestureCallback:(UITapGestureRecognizer *)tapGesture __attribute((objc_requires_super));
//出现滑动手势会回调的方法 如果要重写 必须调用父类的此方法
-(void)panGestureCallback:(UIPanGestureRecognizer *)panGesture __attribute((objc_requires_super));
//决定是否响应某个手势
-(BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch __attribute((objc_requires_super));
//准备展示侧边栏时调用的方法
-(void)prepareToPresentDrawer:(MMDrawerSide)drawer animated:(BOOL)animated __attribute((objc_requires_super));
//关闭侧边栏时调用的方法
-(void)closeDrawerAnimated:(BOOL)animated velocity:(CGFloat)velocity animationOptions:(UIViewAnimationOptions)options completion:(void (^)(BOOL))completion __attribute((objc_requires_super));
//打开侧边栏时调用的方法
-(void)openDrawerSide:(MMDrawerSide)drawerSide animated:(BOOL)animated velocity:(CGFloat)velocity animationOptions:(UIViewAnimationOptions)options completion:(void (^)(BOOL))completion __attribute((objc_requires_super));
//设备旋转方向时调用的方法
-(void)willRotateToInterfaceOrientation:(UIInterfaceOrientation)toInterfaceOrientation duration:(NSTimeInterval)duration __attribute((objc_requires_super));
-(void)willAnimateRotationToInterfaceOrientation:(UIInterfaceOrientation)toInterfaceOrientation duration:(NSTimeInterval)duration __attribute((objc_requires_super));

```

### 四、一些辅助类

        MMDrawerController框架中还提供了一个MMDrawerBarButtonItem的辅助类，这个类可以创建三道杠的菜单按钮。其中方法如下：

```objectivec
//初始化方法
-(id)initWithTarget:(id)target action:(SEL)action;
//获取某个状态下的按钮颜色
-(UIColor *)menuButtonColorForState:(UIControlState)state __attribute__((deprecated("Use tintColor instead")));
//设置某个状态的按钮颜色
-(void)setMenuButtonColor:(UIColor *)color forState:(UIControlState)state __attribute__((deprecated("Use tintColor instead")));
```

MMDrawerBarButtonItem继承自UIBarButtonItem，可以直接在导航栏上使用。

        前面有提到，侧边栏的展现动画开发者可以进行自定义，为了使开发者在使用MMDrawerController时更加方便，MMDrawerController框架中还提供了一个动画辅助类MMDrawerVisualState，这个类中封装好了许多动画效果，开发者可以直接使用，示例如下：

```objectivec
//使用提供的动画模板
[rootController setDrawerVisualStateBlock:[MMDrawerVisualState slideAndScaleVisualStateBlock]];
```

MMDrawerVisualState中所提供的动画模板列举如下：

```objectivec
//从后向前渐现
+(MMDrawerControllerDrawerVisualStateBlock)slideAndScaleVisualStateBlock;
//滑动渐现
+(MMDrawerControllerDrawerVisualStateBlock)slideVisualStateBlock;
//立方动画
+(MMDrawerControllerDrawerVisualStateBlock)swingingDoorVisualStateBlock;
//视差动画
+(MMDrawerControllerDrawerVisualStateBlock)parallaxVisualStateBlockWithParallaxFactor:(CGFloat)parallaxFactor;
```

### 五、MMDrawerController无法完成的需求

        为了确保MMDrawerController库的轻量级，其作者在设计时也做了功能上的取舍权衡，MMDrawerController无法完成以下需求：

1.上边栏与下边栏。

2.同时展示左边栏与又边栏。

3.无法设置显示一个最小的抽屉宽度。

4.不能支持UITabBarController容器。

5.不能在中心视图控制器之上呈现侧边栏视图。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

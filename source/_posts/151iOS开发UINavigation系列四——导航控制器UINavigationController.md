---
title: iOS开发UINavigation系列四——导航控制器UINavigationController
date: 2015-11-10
categories: iOS之UI控件
tags: []
---
## iOS开发UINavigation系列四——导航控制器UINavigationController

### 一、引言

        在前面的博客中，我么你介绍了UINavigationBar，UINavigationItem和UIToolBar，UINavigationController是将这些控件和UIViewController紧密的结合了起来，使用导航，我们的应用程序层次会更加分明，对controller的管理也更加方便。前几篇博客地址如下：

UINavigationBar：[http://my.oschina.net/u/2340880/blog/527706](http://my.oschina.net/u/2340880/blog/527706)

UINavigationItem：[http://my.oschina.net/u/2340880/blog/527781](http://my.oschina.net/u/2340880/blog/527781)

UIToolBar：[http://my.oschina.net/u/2340880/blog/528168](http://my.oschina.net/u/2340880/blog/528168)

### 二、导航控制器的创建和controller的管理

        导航控制器是一个堆栈结构，只是其中管理的对象是controller，通过push与pop进行controller的切换，我们有两种方式可以创建导航控制器：

```
//通过一个自定义的导航栏和工具栏创建导航控制器
- (instancetype)initWithNavigationBarClass:(nullable Class)navigationBarClass toolbarClass:(nullable Class)toolbarClass;
//使用系统默认的导航栏和工具栏，通过一个根视图创建导航控制器
- (instancetype)initWithRootViewController:(UIViewController *)rootViewController;
```

通过以下方法对视图控制器进行管理操作：

```
//设置管理的视图控制器
- (void)setViewControllers:(NSArray<UIViewController *> *)viewControllers animated:(BOOL)animated;
//压入新的视图控制器
- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated;
//弹出一个视图控制器 返回的是pop的controller
- (nullable UIViewController *)popViewControllerAnimated:(BOOL)animated;
//弹出到某个视图控制器 返回所有pop的controller
- (nullable NSArray<__kindof UIViewController *> *)popToViewController:(UIViewController *)viewController animated:(BOOL)animated; 
//直接pop到根视图控制器，返回所有被pop的controller
- (nullable NSArray<__kindof UIViewController *> *)popToRootViewControllerAnimated:(BOOL)animated;
```

### 三、导航控制器中的常用方法和属性

```
//返回栈顶的controller
@property(nullable, nonatomic,readonly,strong) UIViewController *topViewController; 
//返回显示的controller
@property(nullable, nonatomic,readonly,strong) UIViewController *visibleViewController;
```

上面两个方法的区别在于，topViewController是返回被push出的最后一个controller，但是如果之后又有present进行莫泰跳转，visibleViewController会返回当前显示的controller。例如A-push-B-present-C，则topViewController会返回B，visibleViewController会返回C。

```
//返回堆栈中所有的controller
@property(nonatomic,copy) NSArray<__kindof UIViewController *> *viewControllers;
//设置隐藏导航栏
@property(nonatomic,getter=isNavigationBarHidden) BOOL navigationBarHidden;
- (void)setNavigationBarHidden:(BOOL)hidden animated:(BOOL)animated;
//导航栏对象，只读属性
@property(nonatomic,readonly) UINavigationBar *navigationBar;
//隐藏状态栏
@property(nonatomic,getter=isToolbarHidden) BOOL toolbarHidden NS_AVAILABLE_IOS(3_0);
- (void)setToolbarHidden:(BOOL)hidden animated:(BOOL)animated;
//状态栏对象
@property(null_resettable,nonatomic,readonly) UIToolbar *toolbar;
//导航中的返回手势对象
//iOS7之后，在导航中右划会进行pop操作，设置这个的enable可以控制设置手势是否失效
@property(nullable, nonatomic, readonly) UIGestureRecognizer *interactivePopGestureRecognizer;
```

### 四、iOS8后导航的新特性

```
//这个方法是为了iOS方法的命名统一，在导航中，其作用和push一样
- (void)showViewController:(UIViewController *)vc sender:(nullable id)sender;
//弹出键盘的时候隐藏导航栏
@property (nonatomic, readwrite, assign) BOOL hidesBarsWhenKeyboardAppears;
//屏幕滑动的时候隐藏导航栏，常用于tableView,上滑隐藏导航栏，下滑显示，带动画效果
@property (nonatomic, readwrite, assign) BOOL hidesBarsOnSwipe;
//滑动隐藏导航栏的手势
@property (nonatomic, readonly, strong) UIPanGestureRecognizer *barHideOnSwipeGestureRecognizer;
//横屏的时候隐藏导航栏
@property (nonatomic, readwrite, assign) BOOL hidesBarsWhenVerticallyCompact;
//敲击屏幕可以隐藏与显示导航栏
@property (nonatomic, readwrite, assign) BOOL hidesBarsOnTap;
//敲击屏幕的手势
@property (nonatomic, readonly, assign) UITapGestureRecognizer *barHideOnTapGestureRecognizer;
```

iOS8中增加的这些方法，不得不说着实在用户体验生进了一大步，从中也可以看出apple对于用户体验度的用心。

### 五、UINavigationDelegate

        导航控制器还提供了一些代理回调方法，如下：

```
//视图将要展示时调用的方法
- (void)navigationController:(UINavigationController *)navigationController willShowViewController:(UIViewController *)viewController animated:(BOOL)animated;
//视图已经展示时调用的方法
- (void)navigationController:(UINavigationController *)navigationController didShowViewController:(UIViewController *)viewController animated:(BOOL)animated;
//设置方法设置导航控制器支持的设备方向
- (UIInterfaceOrientationMask)navigationControllerSupportedInterfaceOrientations:(UINavigationController *)navigationController NS_AVAILABLE_IOS(7_0);
//这个方法设置导航控制器的首选设备方向
- (UIInterfaceOrientation)navigationControllerPreferredInterfaceOrientationForPresentation:(UINavigationController *)navigationController NS_AVAILABLE_IOS(7_0);
//下面两个方法可以对导航的转场动画进行设置
- (nullable id <UIViewControllerInteractiveTransitioning>)navigationController:(UINavigationController *)navigationController interactionControllerForAnimationController:(id <UIViewControllerAnimatedTransitioning>) animationController;
- (nullable id <UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController animationControllerForOperation:(UINavigationControllerOperation)operation fromViewController:(UIViewController *)fromVC toViewController:(UIViewController *)toVC ;
```

### 六、与UIViewController相关

        当一个controller被添加到导航中后，系统会为它分配一些属性，如下：

```
//当前controller对应的导航项
@property(nonatomic,readonly,strong) UINavigationItem *navigationItem;
//push的时候隐藏底部栏，如push后隐藏tabbar
@property(nonatomic) BOOL hidesBottomBarWhenPushed;
//管理它的导航控制器
@property(nullable, nonatomic,readonly,strong) UINavigationController *navigationController;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

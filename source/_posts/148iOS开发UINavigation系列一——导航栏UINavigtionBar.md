---
title: iOS开发UINavigation系列一——导航栏UINavigtionBar
date: 2015-11-08
categories: iOS之UI控件
tags: []
---
## iOS开发UINavigation系列一——导航栏UINavigtionBar

### 一、导航栏的使用

        在iOS开发中，我们通常会使用导航控制器，导航控制器中封装了一个UINavigationBar，实际上，我们也可以在不使用导航控制器的前提下，单独使用导航栏，在UINavigationBar中，也有许多我们可以定制的属性，用起来十分方便。

### 二、UINavigationBar的创建和风格类型

        导航栏继承于UIView，所以我们可以像创建普通视图那样创建导航栏，比如我们创建一个高度为80的导航栏，将其放在ViewController的头部，代码如下：

```
UINavigationBar *bar = [[UINavigationBar alloc]initWithFrame:CGRectMake(0, 0, 320, 80)];
[self.view addSubview:bar];
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/1108/154733_Ssfx_2340880.png)        

我们也可以设置导航栏的风格属性，从iOS6之后，UINavigationBar默认为半透明的样式，从上面也可以看出，白色的导航栏下面透出些许背景的红色。导航栏的风格属性可以通过下面的属性来设置：

```
@property(nonatomic,assign) UIBarStyle barStyle;
```

UIBarStyle是一个枚举，其中大部分的样式都已弃用，有效果的只有如下两个：

```
typedef NS_ENUM(NSInteger, UIBarStyle) {
    UIBarStyleDefault          = 0,//默认
    UIBarStyleBlack            = 1,//黑色
}
```

默认的风格就是我们上面看到的白色的风格，黑色的风格效果瑞如下：

![](http://static.oschina.net/uploads/space/2015/1108/155321_3eoZ_2340880.png)

### 三、导航栏常用属性和方法

        从上面我们可以看到，iOS6后导航栏默认都是半透明的，我们可以通过下面的bool值来设置这个属性，设置为NO，则导航栏不透明，默认为YES：

```
@property(nonatomic,assign,getter=isTranslucent) BOOL translucent;
```

下面一些方法用于设置NavigationBar及上面item的颜色相关属性：

```
@property(null_resettable, nonatomic,strong) UIColor *tintColor;
```

tintColor这个属性会影响到导航栏上左侧pop按钮的图案颜色和字体颜色，系统默认是如下颜色：

![](http://static.oschina.net/uploads/space/2015/1108/160424_TSxu_2340880.png)

```
@property(nullable, nonatomic,strong) UIColor *barTintColor;
```

BarTintColor用于设置导航栏的背景色，这个属性被设置后，半透明的效果将失效：

![](http://static.oschina.net/uploads/space/2015/1108/160642_ubyK_2340880.png)

```
- (void)setBackgroundImage:(nullable UIImage *)backgroundImage forBarMetrics:(UIBarMetrics)barMetrics NS_AVAILABLE_IOS(5_0) UI_APPEARANCE_SELECTOR;
- (nullable UIImage *)backgroundImageForBarMetrics:(UIBarMetrics)barMetrics;
```

上面两个方法用于设置和获取导航栏的背景图案，这里需要注意，默认背景图案是不做缩放处理的，所以我们使用的图片尺寸要和导航栏尺寸匹配，这里面还有一个UIBarMetrics参数，这个参数设置设备的状态，如下：

```
typedef NS_ENUM(NSInteger, UIBarMetrics) {
    UIBarMetricsDefault,//正常竖屏状态
    UIBarMetricsCompact,//横屏状态
};
```

```
//设置导航栏的阴影图片
@property(nullable, nonatomic,strong) UIImage *shadowImage;
//设置导航栏的标题字体属性
@property(nullable,nonatomic,copy) NSDictionary<NSString *,id> *titleTextAttributes;
```

标题字体属性会影响到导航栏的中间标题，如下：

```
   bar.titleTextAttributes = @{NSForegroundColorAttributeName:[UIColor redColor]};
```

![](http://static.oschina.net/uploads/space/2015/1108/162010_I83I_2340880.png)

我们也可以通过下面的属性设置导航栏标题的竖直位置偏移：

```
- (void)setTitleVerticalPositionAdjustment:(CGFloat)adjustment forBarMetrics:(UIBarMetrics)barMetrics;
- (CGFloat)titleVerticalPositionAdjustmentForBarMetrics:(UIBarMetrics)barMetrics;
```

还有一个细节，导航栏左侧pop按钮的图案默认是一个箭头，我们可以使用下面的方法修改：

```
@property(nullable,nonatomic,strong) UIImage *backIndicatorImage;
@property(nullable,nonatomic,strong) UIImage *backIndicatorTransitionMaskImage;
```

### 四、导航栏中item的push与pop操作

        UINavigationBar上面不只是简单的显示标题，它也将标题进行了堆栈的管理，每一个标题抽象为的对象在iOS系统中是UINavigationItem对象，我们可以通过push与pop操作管理item组。

```
//向栈中添加一个item，上一个item会被推向导航栏的左侧，变为pop按钮，会有一个动画效果
- (void)pushNavigationItem:(UINavigationItem *)item animated:(BOOL)animated;
//pop一个item
- (nullable UINavigationItem *)popNavigationItemAnimated:(BOOL)animated; 
//当前push到最上层的item
@property(nullable, nonatomic,readonly,strong) UINavigationItem *topItem;
//仅次于最上层的item，一般式被推向导航栏左侧的item
@property(nullable, nonatomic,readonly,strong) UINavigationItem *backItem;
//获取堆栈中所有item的数组
@property(nullable,nonatomic,copy) NSArray<UINavigationItem *> *items;
//设置一组item
- (void)setItems:(nullable NSArray<UINavigationItem *> *)items animated:(BOOL)animated;
```

### 五、UINavigationBarDelegate

        在UINavigationBar中，还有如下一个属性：

```
@property(nullable,nonatomic,weak) id<UINavigationBarDelegate> delegate;
```

通过代理，我们可以监控导航栏的一些push与pop操作：

```
//item将要push的时候调用，返回NO，则不能push
- (BOOL)navigationBar:(UINavigationBar *)navigationBar shouldPushItem:(UINavigationItem *)item; 
//item已经push后调用
- (void)navigationBar:(UINavigationBar *)navigationBar didPushItem:(UINavigationItem *)item; 
//item将要pop时调用，返回NO，不能pop  
- (BOOL)navigationBar:(UINavigationBar *)navigationBar shouldPopItem:(UINavigationItem *)item; 
//item已经pop后调用 
- (void)navigationBar:(UINavigationBar *)navigationBar didPopItem:(UINavigationItem *)item;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

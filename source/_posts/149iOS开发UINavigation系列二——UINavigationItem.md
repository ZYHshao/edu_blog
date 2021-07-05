---
title: iOS开发UINavigation系列二——UINavigationItem
date: 2015-11-08
categories: iOS之UI控件
tags: []
---
## iOS开发UINavigation系列二——UINavigationItem

### 一、引言

        UINavigationItem是导航栏上用于管理导航项的类，在上一篇博客中，我们知道导航栏是通过push与pop的堆栈操作来对item进行管理的，同样，每一个Item自身也有许多属性可供我们进行自定制。这篇博客，主要讨论UINavigationItem的使用方法。

UINavigationBar：[http://my.oschina.net/u/2340880/blog/527706](http://my.oschina.net/u/2340880/blog/527706)。

### 二、来说说UINavigationItem

        Item，从英文上来理解，它可以解释为一个项目，因此，item不是一个简单的label标题，也不是一个简单的button按钮，它是导航栏中管理的一个项目的抽象。说起来有些难于理解，通过代码，我们就能很好的理解Item的意义。

首先，我们创建一个item，用UINavigationBar导航栏push出来：

```
 UINavigationItem * item = [[UINavigationItem alloc]initWithTitle:@"title"];
 UINavigationBar * bar = [[UINavigationBar alloc]initWithFrame:CGRectMake(0, 0, 320, 64)];
 [bar pushNavigationItem:item animated:YES];
```

我们可以看到，在导航栏上的中间，有title这样一个item：

![](http://static.oschina.net/uploads/space/2015/1108/204226_K0S9_2340880.png)

除了创建一个标题item，我们也可以创建一个View类型的item：

```
        UIView * view = [[UIView alloc]initWithFrame:CGRectMake(0, 0, 30, 30)];
        view.backgroundColor = [UIColor brownColor];
        item.titleView = view;
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/1108/204519_XTxZ_2340880.png)

通过下面的属性，可以给这个Item添加一个说明文字，这段文字会显示在item的上方：

```
item.prompt= @"我是navigationItem的说明文字";
```

![](http://static.oschina.net/uploads/space/2015/1108/204820_TpPd_2340880.png)

上面我们看到的这些，实际上只是一个item的一部分，item还有许多其他的附件，如果我们使导航栏再push出一个item，这时导航栏的左边会出现一个返回按钮，这个返回按钮实际上是数据第一个item的，我们做如下的设置：

```
        UINavigationItem * item = [[UINavigationItem alloc]initWithTitle:@"title"];
        UINavigationItem * item2 = [[UINavigationItem alloc]initWithTitle:@"title2"];
        item.backBarButtonItem = [[UIBarButtonItem alloc]initWithTitle:@"title1" style:nil target:nil action:nil];
        [bar pushNavigationItem:item animated:YES];
        [bar pushNavigationItem:item2 animated:YES];
```

![](http://static.oschina.net/uploads/space/2015/1108/205424_aU5v_2340880.png)

可以看出，虽然当前push出来的item是item2，但是左边的返回按钮是属于item的。这里有一点需要注意，虽然backBarButtonItem的标题我们可以自定义，但是方法和其他属性我们都不能定制，是系统实现好的。

当然，我们也可以设置在push出来新的item的时候，隐藏前面的返回按钮，使用如下属性：

```
@property(nonatomic,assign) BOOL hidesBackButton;
- (void)setHidesBackButton:(BOOL)hidesBackButton animated:(BOOL)animated;
```

默认为NO，设置为YES将会隐藏返回按钮。

### 三、关于UIBarButtonItem

        一个UINavigationItem中，还可以包含许多BarButtonItem，BarButtonItem是一系列的按钮，会出现在导航栏的左侧或者右侧。例如：

```
        UIBarButtonItem * button = [[UIBarButtonItem alloc]initWithTitle:@"按钮" style:UIBarButtonItemStyleDone target:self action:@selector(click)];
        item.leftBarButtonItem = button;
```

![](http://static.oschina.net/uploads/space/2015/1108/210349_Qwcf_2340880.png)

这个barButtonItem是一个按钮，可以触发一个方法，这有时候对我们来说十分有用。但是有一个你一定发现了，如果继续push出来Item，原来的返回按钮不见了，是否隐藏返回按钮，由下面这个属性控制：

```
item.leftItemsSupplementBackButton=YES;
```

![](http://static.oschina.net/uploads/space/2015/1108/210752_ioge_2340880.png)

我们也可以通过下面的方法设置右边的按钮，或者直接设置一组按钮：

```
@property(nullable, nonatomic,strong) UIBarButtonItem *leftBarButtonItem;
@property(nullable, nonatomic,strong) UIBarButtonItem *rightBarButtonItem;
- (void)setLeftBarButtonItem:(nullable UIBarButtonItem *)item animated:(BOOL)animated;
- (void)setRightBarButtonItem:(nullable UIBarButtonItem *)item animated:(BOOL)animated;

@property(nullable,nonatomic,copy) NSArray<UIBarButtonItem *> *leftBarButtonItems;
@property(nullable,nonatomic,copy) NSArray<UIBarButtonItem *> *rightBarButtonItems;
- (void)setLeftBarButtonItems:(nullable NSArray<UIBarButtonItem *> *)items animated:(BOOL)animated;
- (void)setRightBarButtonItems:(nullable NSArray<UIBarButtonItem *> *)items animated:(BOOL)animated;
```

### 四、再看UIBarButtonItem

        上面我们了解到了，一个NavigationItem基本上是有三大部分组成的，当前显示的部分，返回按钮部分，和ButtonItem部分，同样对于创建和设置UIBarButoonItem，也有很多方法供我们使用。

        首先是创建与初始化的方法：

```
- (instancetype)initWithTitle:(nullable NSString *)title style:(UIBarButtonItemStyle)style target:(nullable id)target action:(nullable SEL)action;
```

这个方法通过一个标题创建ButtonItem，其中style参数可以设置一个风格，枚举如下：

```
typedef NS_ENUM(NSInteger, UIBarButtonItemStyle) {
    UIBarButtonItemStylePlain,
    UIBarButtonItemStyleDone,
};
```

这两种风格差别并不大，如下是效果，Done风格的字体加粗一些：

![](http://static.oschina.net/uploads/space/2015/1108/211623_hTPZ_2340880.png)

我们因为可以通过一个图片来创建BarButtonItem：

```
- (instancetype)initWithImage:(nullable UIImage *)image style:(UIBarButtonItemStyle)style target:(nullable id)target action:(nullable SEL)action;
- (instancetype)initWithImage:(nullable UIImage *)image landscapeImagePhone:(nullable UIImage *)landscapeImagePhone style:(UIBarButtonItemStyle)style target:(nullable id)target action:(nullable SEL)action;
```

上面这两个方法中，第一个方法与使用文字创建的方法类似，第二个方法多了一个landscapeImagePhone的参数，这个参数可以设置设备横屏时的图片。

我们也可以使用自定义的View来创建BarButtonItem：

```
- (instancetype)initWithCustomView:(UIView *)customView;
```

除了上面一些自定义的创建方法外，对于BarButtonItem这个对象，系统也封装好了许多原生的可以供我们使用，创建的时候使用如下方法：

```
UIBarButtonItem * button = [[UIBarButtonItem alloc]initWithBarButtonSystemItem:UIBarButtonSystemItemCamera target:self action:nil];
```

![](http://static.oschina.net/uploads/space/2015/1108/212911_VHfF_2340880.png)

上面的SystemItem是系统为我们做好的许多buttonItem的类型，枚举如下：

```
typedef NS_ENUM(NSInteger, UIBarButtonSystemItem) {
    UIBarButtonSystemItemDone,//显示完成
    UIBarButtonSystemItemCancel,//显示取消
    UIBarButtonSystemItemEdit,  //显示编辑
    UIBarButtonSystemItemSave, //显示保存 
    UIBarButtonSystemItemAdd,//显示加号
    UIBarButtonSystemItemFlexibleSpace,//什么都不显示，占位一个空间位置
    UIBarButtonSystemItemFixedSpace,//和上一个类似
    UIBarButtonSystemItemCompose,//显示写入按钮
    UIBarButtonSystemItemReply,//显示循环按钮
    UIBarButtonSystemItemAction,//显示活动按钮
    UIBarButtonSystemItemOrganize,//显示组合按钮
    UIBarButtonSystemItemBookmarks,//显示图书按钮
    UIBarButtonSystemItemSearch,//显示查找按钮
    UIBarButtonSystemItemRefresh,//显示刷新按钮
    UIBarButtonSystemItemStop,//显示停止按钮
    UIBarButtonSystemItemCamera,//显示相机按钮
    UIBarButtonSystemItemTrash,//显示移除按钮
    UIBarButtonSystemItemPlay,//显示播放按钮
    UIBarButtonSystemItemPause,//显示暂停按钮
    UIBarButtonSystemItemRewind,//显示退后按钮
    UIBarButtonSystemItemFastForward,//显示前进按钮
    UIBarButtonSystemItemUndo,//显示消除按钮
    UIBarButtonSystemItemRedo ,//显示重做按钮
    UIBarButtonSystemItemPageCurl ,//在tool上有效
};
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

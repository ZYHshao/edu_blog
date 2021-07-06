---
title: iOS翻页视图控制器UIPageViewController的应用
date: 2016-03-03
categories: iOS之UI控件
tags: []
---
## iOS翻页视图控制器UIPageViewController的应用

### 一、引言

    UIPageViewController是iOS中少见的动画视图控制器之一，通过它既可以创建类似UIScrollView与UIPageControl结合的滚屏视图，也可以创建类似图书效果的炫酷翻页视图。UIPageViewController类似一个视图容器，其中每个具体的视图由各自的ViewController进行维护管理，UIPageViewController只进行协调与动画布置。下图可以很好的展现出UIPageViewControlelr的使用结构：

![](http://static.oschina.net/uploads/space/2016/0303/175904_eIYo_2340880.png)

上图中，UIPageViewControllerDataSource协议为UIPageViewController提供数据支持，DataSource协议提供的数据来自各个ViewContoller自行维护，UIPageViewControllerDelegate中的回调可以对翻页动作，屏幕旋转动作等进行监听。UIPageViewController把从DataSource中获取到的视图数据渲染给View用于当前视图控制器的展示。

### 二、创建一个UIPageViewController

    首先新建一个类作为翻页视图控制器中具体每一页视图的控制器，使其继承于UIViewController：

ModelViewController.h

```
#import <UIKit/UIKit.h>
@interface ModelViewController : UIViewController
+(ModelViewController *)creatWithIndex:(int)index;
@property(nonatomic,strong)UILabel * indexLabel;
@end
```

ModelViewController.m

```
#import "ModelViewController.h"
@interface ModelViewController ()
@end
@implementation ModelViewController
+(ModelViewController *)creatWithIndex:(int)index{
    ModelViewController * con = [[ModelViewController alloc]init];
    con.indexLabel = [[UILabel alloc]initWithFrame:CGRectMake(110, 200, 100, 30)];
    con.indexLabel.text = [NSString stringWithFormat:@"第%d页",index];
    [con.view addSubview:con.indexLabel];
    return con;
}
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    self.view.backgroundColor = [UIColor redColor];
}
@end
```

在工程模板自带的ViewController.m文件中实现如下代码：

```
#import "ViewController.h"
#import "ModelViewController.h"
//遵守协议
@interface ViewController ()<UIPageViewControllerDataSource,UIPageViewControllerDelegate>
{
    //翻页视图控制器对象
    UIPageViewController * _pageViewControl;
    //数据源数组
    NSMutableArray * _dataArray;
}
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    //进行初始化
    _pageViewControl = [[UIPageViewController alloc]initWithTransitionStyle:UIPageViewControllerTransitionStyleScroll navigationOrientation:UIPageViewControllerNavigationOrientationHorizontal options:@{UIPageViewControllerOptionSpineLocationKey:@0,UIPageViewControllerOptionInterPageSpacingKey:@10}];
    self.view.backgroundColor = [UIColor greenColor];
    //设置翻页视图的尺寸
    _pageViewControl.view.bounds=self.view.bounds;
    //设置数据源与代理
    _pageViewControl.dataSource=self;
    _pageViewControl.delegate=self;
    //创建初始界面
    ModelViewController * model = [ModelViewController creatWithIndex:1];
    //设置初始界面
    [_pageViewControl setViewControllers:@[model] direction:UIPageViewControllerNavigationDirectionReverse animated:YES completion:nil];
    //设置是否双面展示
    _pageViewControl.doubleSided = NO;
    _dataArray = [[NSMutableArray alloc]init];
    [_dataArray addObject:model];
    [self.view addSubview:_pageViewControl.view];
}
//翻页控制器进行向前翻页动作 这个数据源方法返回的视图控制器为要显示视图的视图控制器
- (nullable UIViewController *)pageViewController:(UIPageViewController *)pageViewController viewControllerBeforeViewController:(UIViewController *)viewController{
    int index = (int)[_dataArray indexOfObject:viewController];
    if (index==0) {
        return nil;
    }else{
        return _dataArray[index-1];
    }
}
//翻页控制器进行向后翻页动作 这个数据源方法返回的视图控制器为要显示视图的视图控制器
- (nullable UIViewController *)pageViewController:(UIPageViewController *)pageViewController viewControllerAfterViewController:(UIViewController *)viewController{
    int index = (int)[_dataArray indexOfObject:viewController];
    if (index==9) {
        return nil;
    }else{
        if (_dataArray.count-1>=(index+1)) {
            return _dataArray[index+1];
        }else{
            ModelViewController * model = [ModelViewController creatWithIndex:index+2];
            [_dataArray addObject:model];
            return model;
        }
    }
}
//屏幕旋转触发的代理方法
- (UIPageViewControllerSpineLocation) pageViewController:(UIPageViewController *)pageViewController spineLocationForInterfaceOrientation:(UIInterfaceOrientation)orientation{
    return UIPageViewControllerSpineLocationMin;
}
//设置分页控制器的分页数
- (NSInteger)presentationCountForPageViewController:(UIPageViewController *)pageViewController {
    
    return 10;
}
//设置初始的分页点
- (NSInteger)presentationIndexForPageViewController:(UIPageViewController *)pageViewController{
    return 0;
}
@end
```

上面创建了最简单的翻页视图控制器示例，效果如下图：

![](http://static.oschina.net/uploads/space/2016/0303/181903_qkPw_2340880.png)

### 三、UIPageViewController中方法使用解析

```
//创建翻页视图控制器对象
- (instancetype)initWithTransitionStyle:(UIPageViewControllerTransitionStyle)style navigationOrientation:(UIPageViewControllerNavigationOrientation)navigationOrientation options:(nullable NSDictionary<NSString *, id> *)options;
```

上面方法用于创建视图控制器对象，其中UIPageViewControllerTransitionStyle参数设置翻页控制器的风格，枚举如下：

```
typedef NS_ENUM(NSInteger, UIPageViewControllerTransitionStyle) {
    UIPageViewControllerTransitionStylePageCurl = 0, //类似于书本翻页效果
    UIPageViewControllerTransitionStyleScroll = 1 // 类似于ScrollView的滑动效果
};
```

如果设置为UIPageViewControllerTransitionStyleCurl，翻页效果如下图所示：

![](http://static.oschina.net/uploads/space/2016/0303/182447_ukl6_2340880.png)

上面初始化方法中的UIPageViewControllerNavigationOrientation属性设置翻页的方向，枚举如下：

```
typedef NS_ENUM(NSInteger, UIPageViewControllerNavigationOrientation) {
    UIPageViewControllerNavigationOrientationHorizontal = 0,//水平翻页
    UIPageViewControllerNavigationOrientationVertical = 1//竖直翻页
};
```

options参数用于设置翻页视图控制器的配置字典，其可以设置的配置键值如下：

```
//这个键需要设置为UIPageViewControllerOptionSpineLocationKey枚举值对应的NSNumber对象 设置翻页控制器的书轴 后面会介绍
NSString * const UIPageViewControllerOptionSpineLocationKey;
//这个键需要设置为NSNumber类型 设置每页视图的间距 用于滚动视图风格的
NSString * const UIPageViewControllerOptionInterPageSpacingKey;
```

下面是UIPageViewController的一些常用属性与方法：

```
//设置数据源
@property (nullable, nonatomic, weak) id <UIPageViewControllerDelegate> delegate;
//设置代理
@property (nullable, nonatomic, weak) id <UIPageViewControllerDataSource> dataSource;
//获取翻页风格
@property (nonatomic, readonly) UIPageViewControllerTransitionStyle transitionStyle;
//获取翻页方向
@property (nonatomic, readonly) UIPageViewControllerNavigationOrientation navigationOrientation;
//获取书轴类型
@property (nonatomic, readonly) UIPageViewControllerSpineLocation spineLocation;
//设置是否双面显示
@property (nonatomic, getter=isDoubleSided) BOOL doubleSided;
//设置要显示的视图控制器
- (void)setViewControllers:(nullable NSArray<UIViewController *> *)viewControllers direction:(UIPageViewControllerNavigationDirection)direction animated:(BOOL)animated completion:(void (^ __nullable)(BOOL finished))completion;
```

上面只有spineLocation属性有些难于理解，其枚举如下：

```
typedef NS_ENUM(NSInteger, UIPageViewControllerSpineLocation) {
    //对于SCrollView类型的滑动效果 没有书轴 会返回下面这个枚举值
    UIPageViewControllerSpineLocationNone = 0, 
    //以左边或者上边为轴进行翻转 界面同一时间只显示一个View
    UIPageViewControllerSpineLocationMin = 1,  
    //以中间为轴进行翻转 界面同时可以显示两个View
    UIPageViewControllerSpineLocationMid = 2, 
    //以下边或者右边为轴进行翻转 界面同一时间只显示一个View
    UIPageViewControllerSpineLocationMax = 3   
};
```

将上面的示例代码修改几个地方如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    _pageViewControl = [[UIPageViewController alloc]initWithTransitionStyle:UIPageViewControllerTransitionStylePageCurl navigationOrientation:UIPageViewControllerNavigationOrientationVertical options:@{UIPageViewControllerOptionSpineLocationKey:@2,UIPageViewControllerOptionInterPageSpacingKey:@10}];
    self.view.backgroundColor = [UIColor greenColor];
    _pageViewControl.view.bounds=self.view.bounds;
    _pageViewControl.dataSource=self;
    _pageViewControl.delegate=self;
    ModelViewController * model = [ModelViewController creatWithIndex:1];
    ModelViewController * model2 = [ModelViewController creatWithIndex:2];
    [_pageViewControl setViewControllers:@[model,model2] direction:UIPageViewControllerNavigationDirectionReverse animated:YES completion:nil];
    _pageViewControl.doubleSided = YES;
    _dataArray = [[NSMutableArray alloc]init];
    [_dataArray addObject:model];
    [self.view addSubview:_pageViewControl.view];
}
- (UIPageViewControllerSpineLocation) pageViewController:(UIPageViewController *)pageViewController spineLocationForInterfaceOrientation:(UIInterfaceOrientation)orientation{
    return UIPageViewControllerSpineLocationMid;
}
```

运行效果如下图所示：

![](http://static.oschina.net/uploads/space/2016/0303/190900_8U9H_2340880.png)

### 四、UIPageViewControllerDataSource中方法解析

```
//向前翻页展示的ViewController
- (nullable UIViewController *)pageViewController:(UIPageViewController *)pageViewController viewControllerBeforeViewController:(UIViewController *)viewController;
//向后翻页展示的ViewController
- (nullable UIViewController *)pageViewController:(UIPageViewController *)pageViewController viewControllerAfterViewController:(UIViewController *)viewController;
//设置分页控制器的分页点数
- (NSInteger)presentationCountForPageViewController:(UIPageViewController *)pageViewController NS_AVAILABLE_IOS(6_0);
//设置当前分页控制器所高亮的点
- (NSInteger)presentationIndexForPageViewController:(UIPageViewController *)pageViewController NS_AVAILABLE_IOS(6_0);
```

### 五、UIPageViewControllerDelegate中方法解析

```
//翻页视图控制器将要翻页时执行的方法
- (void)pageViewController:(UIPageViewController *)pageViewController willTransitionToViewControllers:(NSArray<UIViewController *> *)pendingViewControllers NS_AVAILABLE_IOS(6_0);
//翻页动画执行完成后回调的方法
- (void)pageViewController:(UIPageViewController *)pageViewController didFinishAnimating:(BOOL)finished previousViewControllers:(NSArray<UIViewController *> *)previousViewControllers transitionCompleted:(BOOL)completed;
//屏幕防线改变时回到的方法，可以通过返回值重设书轴类型枚举
- (UIPageViewControllerSpineLocation)pageViewController:(UIPageViewController *)pageViewController spineLocationForInterfaceOrientation:(UIInterfaceOrientation)orientation;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

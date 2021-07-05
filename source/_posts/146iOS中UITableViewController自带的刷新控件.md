---
title: iOS中UITableViewController自带的刷新控件
date: 2015-11-05
categories: iOS之UI控件
tags: []
---
## iOS中UITableViewController自带的刷新控件

### 一、引言

        在iOS开发中，使用tableView的界面，大多会用到一个下拉刷新的的控件，第三方库中，我们一般会选择比较好用的MJRefresh，其实，在iOS6之后，系统为我们提供了一个原生的刷新控件，使用起来非常方便，只是制定性不强，如果我们没有复杂的需求，使用UIRefreshControl也是不错的一个选择。

### 二、UITableViewController

        相对于UIViewController，UITableViewController只是在内部为我们封装好了一个UITableView，并且遵守好了相关的协议，我们只需要在其中实现方法即可。UITableViewController更多的方面之处是在于下面的这个属性：

```
@property (nonatomic) BOOL clearsSelectionOnViewWillAppear;
```

这是一个bool值，设置为yes后每当当前controller调用ViewWillAppare的时候，都会将cell的选中状态取消，这十分有用，我们在通过点击cell跳转界面后，pop回来不需要在手动修改cell的选中状态了。

        除此之后，TableViewController中还封装了这样一个属性：

```
@property (nonatomic, strong, nullable) UIRefreshControl *refreshControl;
```

这个UIRefreshControl类是iOS6之后引入的一个简单的刷新控件，我们如果设置了它，在tableView下拉的时候，系统会提供给我们一个下拉刷新的效果。

### 三、UIRefreshControl

        这个类也十分简单，通过简单的设置可以展现一个小巧的刷新效果，但是制定性不强，其中主要属性如下：

```
//获取刷新状态
@property (nonatomic, readonly, getter=isRefreshing) BOOL refreshing;
//设置控件颜色
@property (null_resettable, nonatomic, strong) UIColor *tintColor;
//设置控件文字
@property (nullable, nonatomic, strong) NSAttributedString *attributedTitle UI_APPEARANCE_SELECTOR;

// 手动开始刷新
- (void)beginRefreshing NS_AVAILABLE_IOS(6_0);
// 结束刷新
- (void)endRefreshing NS_AVAILABLE_IOS(6_0);
```

需要注意的是，UIRefreshControl是继承于UIControl的，下拉唤醒刷新状态后，会触发UIControleEventValueChange事件，我们可以在其中进行刷新的数据逻辑操作。

例如：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    self.refreshControl = [[UIRefreshControl alloc]init];
    self.refreshControl.tintColor = [UIColor greenColor];
    self.refreshControl.attributedTitle = [[NSAttributedString alloc]initWithString:@"下拉刷新了~~"];
    self.clearsSelectionOnViewWillAppear = YES;
    self.navigationItem.rightBarButtonItem = self.editButtonItem;
    [self.refreshControl addTarget:self action:@selector(change:) forControlEvents:UIControlEventValueChanged];
}

-(void)change:(UIRefreshControl*)con{
    self.refreshControl.attributedTitle = [[NSAttributedString alloc]initWithString:@"开始刷新了~~"];
}
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/1105/171309_HeMn_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

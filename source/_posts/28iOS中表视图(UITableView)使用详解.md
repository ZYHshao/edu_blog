---
title: iOS中表视图(UITableView)使用详解
date: 2015-04-21
categories: iOS之UI控件
tags: [iOS编程]              
---
## iOS中UITableView使用总结

### 一、初始化方法

\- (instancetype)initWithFrame:(CGRect)frame style:(UITableViewStyle)style;  

这个方法初始化表视图的frame大小并且设置一个风格，UITableViewStyle是一个枚举，如下：

```
typedef NS_ENUM(NSInteger, UITableViewStyle) {
    UITableViewStylePlain,                  // 标准的表视图风格
    UITableViewStyleGrouped                 // 分组的表视图风格
};
```

### 二、常用属性

获取表视图的风格(只读属性)

[@property](http://my.oschina.net/property) (nonatomic, readonly) UITableViewStyle style;

设置表示图代理和数据源代理(代理方法后面讨论)

[@property](http://my.oschina.net/property) (nonatomic, assign) id <UITableViewDataSource\> dataSource;

[@property](http://my.oschina.net/property) (nonatomic, assign) id <UITableViewDelegate\>   delegate;

  
 

设置表示图的行高(默认为44)

[@property](http://my.oschina.net/property) (nonatomic)CGFloat rowHeight; 

设置分区的头视图高度和尾视图高度(当代理方法没有实现时才有效)

[@property](http://my.oschina.net/property) (nonatomic)          CGFloat                     sectionHeaderHeight;   

@property (nonatomic)          CGFloat                     sectionFooterHeight; 

  
设置一个行高的估计值(默认为0，表示没有估计,7.0之后可用)

@property (nonatomic)          CGFloat                     estimatedRowHeight; 

注意：这个属性官方的解释是如果你的tableView的行高是可变的，那么设计一个估计高度可以加快代码的运行效率。

下面这两个属性和上面相似，分别设置分区头视图和尾视图的估计高度(7.0之后可用)

@property (nonatomic)          CGFloat            estimatedSectionHeaderHeight; @property (nonatomic)          CGFloat            estimatedSectionFooterHeight;

设置分割线的位置

@property (nonatomic)          UIEdgeInsets                separatorInset;

如果细心，你可能会发现系统默认的tableView的分割线左端并没有顶到边沿。通过这个属性，可以手动设置分割线的位置偏移，比如你向让tableView的分割线只显示右半边，可以如下设置：

```
UITableView * tab = [[UITableView alloc]initWithFrame:self.view.frame style:UITableViewStylePlain];
tab.separatorInset=UIEdgeInsetsMake(0, tab.frame.size.width/2, 0,0);
```

设置tableView背景view视图

@property(nonatomic, readwrite, retain) UIView *backgroundView;

### 三、常用方法详解

重载tableView

\- (void)reloadData;

重载索引栏

\- (void)reloadSectionIndexTitles;

这个方法常用语新加或者删除了索引类别而无需刷新整个表视图的情况下。

获取分区数

\- (NSInteger)numberOfSections;

根据分区获取行数

\- (NSInteger)numberOfRowsInSection:(NSInteger)section;

获取分区的大小(包括头视图，所有行和尾视图)

\- (CGRect)rectForSection:(NSInteger)section; 

根据分区分别获取头视图，尾视图和行的高度

\- (CGRect)rectForHeaderInSection:(NSInteger)section;

\- (CGRect)rectForFooterInSection:(NSInteger)section;

\- (CGRect)rectForRowAtIndexPath:(NSIndexPath *)indexPath;

获取某个点在tableView中的位置信息

\- (NSIndexPath *)indexPathForRowAtPoint:(CGPoint)point;  

获取某个cell在tableView中的位置信息

\- (NSIndexPath *)indexPathForCell:(UITableViewCell *)cell; 

根据一个矩形范围返回一个信息数组，数组中是每一行row的位置信息

\- (NSArray *)indexPathsForRowsInRect:(CGRect)rect; 

通过位置路径获取cell

\- (UITableViewCell *)cellForRowAtIndexPath:(NSIndexPath *)indexPath; 

获取所有可见的cell

\- (NSArray *)visibleCells;

获取所有可见行的位置信息

\- (NSArray *)indexPathsForVisibleRows;

根据分区获取头视图

\- (UITableViewHeaderFooterView *)headerViewForSection:(NSInteger)section;

根据分区获取尾视图

\- (UITableViewHeaderFooterView *)footerViewForSection:(NSInteger)section; 

使表示图定位到某一位置(行)

\- (void)scrollToRowAtIndexPath:(NSIndexPath *)indexPath atScrollPosition:(UITableViewScrollPosition)scrollPosition animated:(BOOL)animated;

 注意：indexPah参数是定位的位置，决定于分区和行号。animated参数决定是否有动画。scrollPosition参数决定定位的相对位置，它使一个枚举，如下：

```
typedef NS_ENUM(NSInteger, UITableViewScrollPosition) {
    UITableViewScrollPositionNone,//同UITableViewScrollPositionTop
    UITableViewScrollPositionTop,//定位完成后，将定位的行显示在tableView的顶部    
    UITableViewScrollPositionMiddle,//定位完成后，将定位的行显示在tableView的中间   
    UITableViewScrollPositionBottom//定位完成后，将定位的行显示在tableView最下面
};
```

使表示图定位到选中行

\- (void)scrollToNearestSelectedRowAtScrollPosition:(UITableViewScrollPosition)scrollPosition animated:(BOOL)animated;

这个函数与上面的非常相似，只是它是将表示图定位到选中的行。

### 四、tableView操作刷新块的应用

在介绍动画块之前，我们先看几个函数：

插入分区

\- (void)insertSections:(NSIndexSet *)sections withRowAnimation:(UITableViewRowAnimation)animation;

animation参数是一个枚举，枚举的动画类型如下

```
typedef NS_ENUM(NSInteger, UITableViewRowAnimation) {
    UITableViewRowAnimationFade,//淡入淡出
    UITableViewRowAnimationRight,//从右滑入
    UITableViewRowAnimationLeft,//从左滑入
    UITableViewRowAnimationTop,//从上滑入
    UITableViewRowAnimationBottom,//从下滑入
    UITableViewRowAnimationNone,  //没有动画
    UITableViewRowAnimationMiddle,          
    UITableViewRowAnimationAutomatic = 100  // 自动选择合适的动画
};
```

删除分区

\- (void)deleteSections:(NSIndexSet *)sections withRowAnimation:(UITableViewRowAnimation)animation;

重载一个分区

\- (void)reloadSections:(NSIndexSet *)sections withRowAnimation:(UITableViewRowAnimation)animation ;

移动一个分区

\- (void)moveSection:(NSInteger)section toSection:(NSInteger)newSection;

插入一些行

\- (void)insertRowsAtIndexPaths:(NSArray *)indexPaths withRowAnimation:(UITableViewRowAnimation)animation;

删除一些行

\- (void)deleteRowsAtIndexPaths:(NSArray *)indexPaths withRowAnimation:(UITableViewRowAnimation)animation;

重载一些行

\- (void)reloadRowsAtIndexPaths:(NSArray *)indexPaths withRowAnimation:(UITableViewRowAnimation)animation;

移动某行

\- (void)moveRowAtIndexPath:(NSIndexPath *)indexPath toIndexPath:(NSIndexPath *)newIndexPath;

了解了上面几个函数，我们来看什么是操作刷新块：

当我们调用的上面的函数时，tableView会立刻调用代理方法进行刷新，如果其中我们所做的操作是删除某行，而然数据源数组我们可能并没有刷新，程序就会崩溃掉，原因是代理返回的信息和我们删除后不符。

IOS为我们提供了下面两个函数解决这个问题：

开始块标志

\- (void)beginUpdates; 

结束快标志

\- (void)endUpdates; 

我们可以将我们要做的操作全部写在这个块中，那么，只有当程序执行到结束快标志后，才会调用代理刷新方法。代码示例如下：

```
[tab beginUpdates];
    [tab deleteRowsAtIndexPaths:@[[NSIndexPath indexPathForRow:1 inSection:0]] withRowAnimation:UITableViewRowAnimationLeft];
    [dataArray removeObjectAtIndex:1];
    [tab endUpdates];
```

注意：不要在这个块中调用reloadData这个方法，它会使动画失效。

### 五、tableView的编辑操作

设置是否是编辑状态(编辑状态下的cell左边会出现一个减号，点击右边会划出删除按钮)

@property (nonatomic, getter=isEditing) BOOL editing;                             

\- (void)setEditing:(BOOL)editing animated:(BOOL)animated;

设置cell是否可以被选中(默认为YES)

@property (nonatomic) BOOL allowsSelection;

设置cell编辑模式下是否可以被选中

@property (nonatomic) BOOL allowsSelectionDuringEditing;  

设置是否支持多选

@property (nonatomic) BOOL allowsMultipleSelection;

设置编辑模式下是否支持多选

@property (nonatomic) BOOL allowsMultipleSelectionDuringEditing;

### 六、选中cell的相关操作

获取选中cell的位置信息

\- (NSIndexPath *)indexPathForSelectedRow; 

获取多选cell的位置信息

\- (NSArray *)indexPathsForSelectedRows;

代码手动选中与取消选中某行

\- (void)selectRowAtIndexPath:(NSIndexPath *)indexPath animated:(BOOL)animated scrollPosition:(UITableViewScrollPosition)scrollPosition;

\- (void)deselectRowAtIndexPath:(NSIndexPath *)indexPath animated:(BOOL)animated;

注意：这两个方法将不会回调代理中的方法。  
 

### 七、tableView附件的相关方法

设置索引栏最小显示行数

@property (nonatomic) NSInteger sectionIndexMinimumDisplayRowCount;                                                      

设置索引栏字体颜色

@property (nonatomic, retain) UIColor *sectionIndexColor;

设置索引栏背景颜色

@property (nonatomic, retain) UIColor *sectionIndexBackgroundColor;

设置索引栏被选中时的颜色

@property (nonatomic, retain) UIColor *sectionIndexTrackingBackgroundColor;

设置分割线的风格

@property (nonatomic) UITableViewCellSeparatorStyle separatorStyle;

这个风格是一个枚举，如下：

```
typedef NS_ENUM(NSInteger, UITableViewCellSeparatorStyle) {
    UITableViewCellSeparatorStyleNone,//无线
    UITableViewCellSeparatorStyleSingleLine,//有线
    UITableViewCellSeparatorStyleSingleLineEtched  
};
```

设置分割线颜色

@property (nonatomic, retain) UIColor           *separatorColor;

设置分割线毛玻璃效果(IOS8之后可用)

@property (nonatomic, copy) UIVisualEffect      *separatorEffect;

注意：这个属性是IOS8之后新的。

设置tableView头视图

@property (nonatomic, retain) UIView *tableHeaderView;  

设置tableView尾视图

@property (nonatomic, retain) UIView *tableFooterView; 

从复用池中取cell

\- (id)dequeueReusableCellWithIdentifier:(NSString *)identifier;

获取一个已注册的cell

\- (id)dequeueReusableCellWithIdentifier:(NSString *)identifier forIndexPath:(NSIndexPath *)indexPath

从复用池获取头视图或尾视图

\- (id)dequeueReusableHeaderFooterViewWithIdentifier:(NSString *)identifier;

通过xib文件注册cell

\- (void)registerNib:(UINib *)nib forCellReuseIdentifier:(NSString *)identifier;

通过OC类注册cell

\- (void)registerClass:(Class)cellClass forCellReuseIdentifier:(NSString *)identifier 

上面两个方法是IOS6之后的方法。

通过xib文件和OC类获取注册头视图和尾视图

\- (void)registerNib:(UINib *)nib forHeaderFooterViewReuseIdentifier:(NSString *)identifier;

\- (void)registerClass:(Class)aClass forHeaderFooterViewReuseIdentifier:(NSString *)

关于tableView的代理方法，因为篇幅原因，总结在下一篇博客中。

错误之处 欢迎指正

学习使用 欢迎转载

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592
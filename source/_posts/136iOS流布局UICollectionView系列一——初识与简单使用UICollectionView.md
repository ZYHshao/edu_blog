---
title: iOS流布局UICollectionView系列一——初识与简单使用UICollectionView image解决方法
date: 2015-10-27
categories: iOS之UI控件
tags: []
---
## iOS流布局UICollectionView系列一——初识与简单使用UICollectionView

### 一、简介

        UICollectionView是iOS6之后引入的一个新的UI控件，它和UITableView有着诸多的相似之处，其中许多代理方法都十分类似。简单来说，UICollectionView是比UITbleView更加强大的一个UI控件，有如下几个方面：

1、支持水平和垂直两种方向的布局

2、通过layout配置方式进行布局

3、类似于TableView中的cell特性外，CollectionView中的Item大小和位置可以自由定义

4、通过layout布局回调的代理方法，可以动态的定制每个item的大小和collection的大体布局属性

5、更加强大一点，完全自定义一套layout布局方案，可以实现意想不到的效果

这篇博客，我们主要讨论CollectionView使用原生layout的方法和相关属性，其他特点和更强的制定化，会在后面的博客中介绍

### 二、先来实现一个最简单的九宫格类布局

        在了解UICollectionView的更多属性前，我们先来使用其进行一个最简单的流布局试试看，在controller的viewDidLoad中添加如下代码：

```
    //创建一个layout布局类
    UICollectionViewFlowLayout * layout = [[UICollectionViewFlowLayout alloc]init];
    //设置布局方向为垂直流布局
    layout.scrollDirection = UICollectionViewScrollDirectionVertical;
    //设置每个item的大小为100*100
    layout.itemSize = CGSizeMake(100, 100);
    //创建collectionView 通过一个布局策略layout来创建
    UICollectionView * collect = [[UICollectionView alloc]initWithFrame:self.view.frame collectionViewLayout:layout];
    //代理设置
    collect.delegate=self;
    collect.dataSource=self;
    //注册item类型 这里使用系统的类型
    [collect registerClass:[UICollectionViewCell class] forCellWithReuseIdentifier:@"cellid"];
   
    [self.view addSubview:collect];
```

这里有一点需要注意，collectionView在完成代理回调前，必须注册一个cell，类似如下:

```
[collect registerClass:[UICollectionViewCell class] forCellWithReuseIdentifier:@"cellid"];
```

这和tableView有些类似，又有些不同，因为tableView除了注册cell的方法外，还可以通过临时创建来做：

```
//tableView在从复用池中取cell的时候，有如下两种方法
//使用这种方式如果复用池中无，是可以返回nil的，我们在临时创建即可
- (nullable __kindof UITableViewCell *)dequeueReusableCellWithIdentifier:(NSString *)identifier;
//6.0后使用如下的方法直接从注册的cell类获取创建，如果没有注册 会崩溃
- (__kindof UITableViewCell *)dequeueReusableCellWithIdentifier:(NSString *)identifier forIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(6_0);
```

我们可以分析：因为UICollectionView是iOS6.0之前的新类，因此这里统一了从复用池中获取cell的方法，没有再提供可以返回nil的方式，并且在UICollectionView的回调代理中，只能使用从复用池中获取cell的方式进行cell的返回，其他方式会崩溃，例如：

```
//这是正确的方法
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewCell * cell  = [collectionView dequeueReusableCellWithReuseIdentifier:@"cellid" forIndexPath:indexPath];
    cell.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    return cell;
}

//这样做会崩溃
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
//    UICollectionViewCell * cell  = [collectionView dequeueReusableCellWithReuseIdentifier:@"cellid" forIndexPath:indexPath];
//    cell.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    UICollectionViewCell * cell = [[UICollectionViewCell alloc]init];
    return cell;
}
```

上面错误的方式会崩溃，信息如下，让我们使用从复用池中取cell的方式：

![](http://static.oschina.net/uploads/space/2015/1027/111953_6cMH_2340880.png)

上面的设置完成后，我们来实现如下几个代理方法：

这里与TableView的回调方式十分类似

```
//返回分区个数
-(NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView{
    return 1;
}
//返回每个分区的item个数
-(NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section{
    return 10;
}
//返回每个item
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewCell * cell  = [collectionView dequeueReusableCellWithReuseIdentifier:@"cellid" forIndexPath:indexPath];
    cell.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    return cell;
}
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/1027/112304_97Vf_2340880.png)

同样，如果内容的大小超出一屏，和tableView类似是可以进行视图滑动的。

还有一点细节，我们在上面设置布局方式的时候设置了垂直布局：

```
layout.scrollDirection = UICollectionViewScrollDirectionVertical;
//这个是水平布局
//layout.scrollDirection = UICollectionViewScrollDirectionHorizontal;
```

这样系统会在一行充满后进行第二行的排列，如果设置为水平布局，则会在一列充满后，进行第二列的布局，这种方式也被称为流式布局

### 三、UICollectionView中的常用方法和属性

```
//通过一个布局策略初识化CollectionView
- (instancetype)initWithFrame:(CGRect)frame collectionViewLayout:(UICollectionViewLayout *)layout;

//获取和设置collection的layout
@property (nonatomic, strong) UICollectionViewLayout *collectionViewLayout;

//数据源和代理
@property (nonatomic, weak, nullable) id <UICollectionViewDelegate> delegate;
@property (nonatomic, weak, nullable) id <UICollectionViewDataSource> dataSource;

//从一个class或者xib文件进行cell(item)的注册
- (void)registerClass:(nullable Class)cellClass forCellWithReuseIdentifier:(NSString *)identifier;
- (void)registerNib:(nullable UINib *)nib forCellWithReuseIdentifier:(NSString *)identifier;

//下面两个方法与上面相似，这里注册的是头视图或者尾视图的类
//其中第二个参数是设置 头视图或者尾视图 系统为我们定义好了这两个字符串
//UIKIT_EXTERN NSString *const UICollectionElementKindSectionHeader NS_AVAILABLE_IOS(6_0);
//UIKIT_EXTERN NSString *const UICollectionElementKindSectionFooter NS_AVAILABLE_IOS(6_0);
- (void)registerClass:(nullable Class)viewClass forSupplementaryViewOfKind:(NSString *)elementKind withReuseIdentifier:(NSString *)identifier;
- (void)registerNib:(nullable UINib *)nib forSupplementaryViewOfKind:(NSString *)kind withReuseIdentifier:(NSString *)identifier;

//这两个方法是从复用池中取出cell或者头尾视图
- (__kindof UICollectionViewCell *)dequeueReusableCellWithReuseIdentifier:(NSString *)identifier forIndexPath:(NSIndexPath *)indexPath;
- (__kindof UICollectionReusableView *)dequeueReusableSupplementaryViewOfKind:(NSString *)elementKind withReuseIdentifier:(NSString *)identifier forIndexPath:(NSIndexPath *)indexPath;

//设置是否允许选中 默认yes
@property (nonatomic) BOOL allowsSelection;

//设置是否允许多选 默认no
@property (nonatomic) BOOL allowsMultipleSelection;

//获取所有选中的item的位置信息
- (nullable NSArray<NSIndexPath *> *)indexPathsForSelectedItems; 

//设置选中某一item，并使视图滑动到相应位置，scrollPosition是滑动位置的相关参数，如下：
/*
typedef NS_OPTIONS(NSUInteger, UICollectionViewScrollPosition) {
    //无
    UICollectionViewScrollPositionNone                 = 0,
    //垂直布局时使用的 对应上中下
    UICollectionViewScrollPositionTop                  = 1 << 0,
    UICollectionViewScrollPositionCenteredVertically   = 1 << 1,
    UICollectionViewScrollPositionBottom               = 1 << 2,
    //水平布局时使用的  对应左中右
    UICollectionViewScrollPositionLeft                 = 1 << 3,
    UICollectionViewScrollPositionCenteredHorizontally = 1 << 4,
    UICollectionViewScrollPositionRight                = 1 << 5
};
*/
- (void)selectItemAtIndexPath:(nullable NSIndexPath *)indexPath animated:(BOOL)animated scrollPosition:(UICollectionViewScrollPosition)scrollPosition;

//将某一item取消选中
- (void)deselectItemAtIndexPath:(NSIndexPath *)indexPath animated:(BOOL)animated;

//重新加载数据
- (void)reloadData;

//下面这两个方法，可以重新设置collection的布局，后面的方法多了一个布局完成后的回调，iOS7后可以用
//使用这两个方法可以产生非常炫酷的动画效果
- (void)setCollectionViewLayout:(UICollectionViewLayout *)layout animated:(BOOL)animated;
- (void)setCollectionViewLayout:(UICollectionViewLayout *)layout animated:(BOOL)animated completion:(void (^ __nullable)(BOOL finished))completion NS_AVAILABLE_IOS(7_0);

//下面这些方法更加强大，我们可以对布局更改后的动画进行设置
//这个方法传入一个布局策略layout，系统会开始进行布局渲染，返回一个UICollectionViewTransitionLayout对象
//这个UICollectionViewTransitionLayout对象管理动画的相关属性，我们可以进行设置
- (UICollectionViewTransitionLayout *)startInteractiveTransitionToCollectionViewLayout:(UICollectionViewLayout *)layout completion:(nullable UICollectionViewLayoutInteractiveTransitionCompletion)completion NS_AVAILABLE_IOS(7_0);
//准备好动画设置后，我们需要调用下面的方法进行布局动画的展示，之后会调用上面方法的block回调
- (void)finishInteractiveTransition NS_AVAILABLE_IOS(7_0);
//调用这个方法取消上面的布局动画设置，之后也会进行上面方法的block回调
- (void)cancelInteractiveTransition NS_AVAILABLE_IOS(7_0);

//获取分区数
- (NSInteger)numberOfSections;

//获取某一分区的item数
- (NSInteger)numberOfItemsInSection:(NSInteger)section;

//下面两个方法获取item或者头尾视图的layout属性，这个UICollectionViewLayoutAttributes对象
//存放着布局的相关数据，可以用来做完全自定义布局，后面博客会介绍
- (nullable UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath;
- (nullable UICollectionViewLayoutAttributes *)layoutAttributesForSupplementaryElementOfKind:(NSString *)kind atIndexPath:(NSIndexPath *)indexPath;

//获取某一点所在的indexpath位置
- (nullable NSIndexPath *)indexPathForItemAtPoint:(CGPoint)point;

//获取某个cell所在的indexPath
- (nullable NSIndexPath *)indexPathForCell:(UICollectionViewCell *)cell;

//根据indexPath获取cell
- (nullable UICollectionViewCell *)cellForItemAtIndexPath:(NSIndexPath *)indexPath;

//获取所有可见cell的数组
- (NSArray<__kindof UICollectionViewCell *> *)visibleCells;

//获取所有可见cell的位置数组
- (NSArray<NSIndexPath *> *)indexPathsForVisibleItems;

//下面三个方法是iOS9中新添加的方法，用于获取头尾视图
- (UICollectionReusableView *)supplementaryViewForElementKind:(NSString *)elementKind atIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(9_0);
- (NSArray<UICollectionReusableView *> *)visibleSupplementaryViewsOfKind:(NSString *)elementKind NS_AVAILABLE_IOS(9_0);
- (NSArray<NSIndexPath *> *)indexPathsForVisibleSupplementaryElementsOfKind:(NSString *)elementKind NS_AVAILABLE_IOS(9_0);

//使视图滑动到某一位置，可以带动画效果
- (void)scrollToItemAtIndexPath:(NSIndexPath *)indexPath atScrollPosition:(UICollectionViewScrollPosition)scrollPosition animated:(BOOL)animated;

//下面这些方法用于动态添加，删除，移动某些分区获取items
- (void)insertSections:(NSIndexSet *)sections;
- (void)deleteSections:(NSIndexSet *)sections;
- (void)reloadSections:(NSIndexSet *)sections;
- (void)moveSection:(NSInteger)section toSection:(NSInteger)newSection;

- (void)insertItemsAtIndexPaths:(NSArray<NSIndexPath *> *)indexPaths;
- (void)deleteItemsAtIndexPaths:(NSArray<NSIndexPath *> *)indexPaths;
- (void)reloadItemsAtIndexPaths:(NSArray<NSIndexPath *> *)indexPaths;
- (void)moveItemAtIndexPath:(NSIndexPath *)indexPath toIndexPath:(NSIndexPath *)newIndexPath;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

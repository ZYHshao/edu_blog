---
title: iOS流布局UICollectionView系列二——UICollectionView的代理方法
date: 2015-10-27
categories: iOS之UI控件
tags: []
---
## iOS流布局UICollectionView系列二——UICollectionView的代理方法

### 一、引言

        在上一篇博客中，介绍了最基本的UICollectionView的使用和其中我们常用的属性和方法，也介绍了瀑布流布局的过程与思路，这篇博客是上一篇的补充，来讨论关于UICollectionView的代理方法的使用。博客地址：

UICollectionView的简介和简单使用：[http://my.oschina.net/u/2340880/blog/522613](http://my.oschina.net/u/2340880/blog/522613)

### 二、UICollectionViewDataSource协议

        这个协议主要用于collectionView相关数据的处理，包含方法如下：

首先，有两个方法是我们必须实现的：

设置每个分区的Item个数

\- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section;

设置返回每个item的属性

\- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath;

下面的方法是可选实现的：

虽然这个方法是可选的，一般我们都会去实现，设置分区数

\- (NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView;

对头视图或者尾视图进行设置

\- (UICollectionReusableView *)collectionView:(UICollectionView *)collectionView viewForSupplementaryElementOfKind:(NSString *)kind atIndexPath:(NSIndexPath *)indexPath;

设置某个item是否可以被移动，返回NO则不能移动

\- (BOOL)collectionView:(UICollectionView *)collectionView canMoveItemAtIndexPath:(NSIndexPath *)indexPath NS\_AVAILABLE\_IOS(9_0);

移动item的时候，会调用这个方法

\- (void)collectionView:(UICollectionView *)collectionView moveItemAtIndexPath:(NSIndexPath *)sourceIndexPath toIndexPath:(NSIndexPath*)destinationIndexPath；

### 三、UICollectionViewDelegate协议

        这个协议用来设置和处理collectionView的功能和一些逻辑，所有方法都是可选实现：

是否允许某个Item的高亮，返回NO，则不能进入高亮状态

\- (BOOL)collectionView:(UICollectionView *)collectionView shouldHighlightItemAtIndexPath:(NSIndexPath *)indexPath;

当item高亮时触发的方法

\- (void)collectionView:(UICollectionView *)collectionView didHighlightItemAtIndexPath:(NSIndexPath *)indexPath;

结束高亮状态时触发的方法

\- (void)collectionView:(UICollectionView *)collectionView didUnhighlightItemAtIndexPath:(NSIndexPath *)indexPath;

是否可以选中某个Item，返回NO，则不能选中

\- (BOOL)collectionView:(UICollectionView *)collectionView shouldSelectItemAtIndexPath:(NSIndexPath *)indexPath;

是否可以取消选中某个Item

\- (BOOL)collectionView:(UICollectionView *)collectionView shouldDeselectItemAtIndexPath:(NSIndexPath *)indexPath;

已经选中某个item时触发的方法

\- (void)collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath;

取消选中某个Item时触发的方法

\- (void)collectionView:(UICollectionView *)collectionView didDeselectItemAtIndexPath:(NSIndexPath *)indexPath;

将要加载某个Item时调用的方法

\- (void)collectionView:(UICollectionView *)collectionView willDisplayCell:(UICollectionViewCell *)cell forItemAtIndexPath:(NSIndexPath *)indexPath NS\_AVAILABLE\_IOS(8_0);

将要加载头尾视图时调用的方法

\- (void)collectionView:(UICollectionView *)collectionView willDisplaySupplementaryView:(UICollectionReusableView *)view forElementKind:(NSString *)elementKind atIndexPath:(NSIndexPath *)indexPath NS\_AVAILABLE\_IOS(8_0);

已经展示某个Item时触发的方法

\- (void)collectionView:(UICollectionView *)collectionView didEndDisplayingCell:(UICollectionViewCell *)cell forItemAtIndexPath:(NSIndexPath *)indexPath;

已经展示某个头尾视图时触发的方法

\- (void)collectionView:(UICollectionView *)collectionView didEndDisplayingSupplementaryView:(UICollectionReusableView *)view forElementOfKind:(NSString *)elementKind atIndexPath:(NSIndexPath *)indexPath;

这个方法设置是否展示长按菜单

\- (BOOL)collectionView:(UICollectionView *)collectionView shouldShowMenuForItemAtIndexPath:(NSIndexPath *)indexPath;

长按菜单中可以触发一下类复制粘贴的方法，效果如下：

![](http://static.oschina.net/uploads/space/2015/1027/151819_6XyQ_2340880.png)

这个方法用于设置要展示的菜单选项

\- (BOOL)collectionView:(UICollectionView *)collectionView canPerformAction:(SEL)action forItemAtIndexPath:(NSIndexPath *)indexPath withSender:(nullable id)sender;

这个方法用于实现点击菜单按钮后的触发方法,通过测试，只有copy，cut和paste三个方法可以使用

\- (void)collectionView:(UICollectionView *)collectionView performAction:(SEL)action forItemAtIndexPath:(NSIndexPath *)indexPath withSender:(nullable id)sender;

通过下面的方式可以将点击按钮的方法名打印出来：

```
-(void)collectionView:(UICollectionView *)collectionView performAction:(SEL)action forItemAtIndexPath:(NSIndexPath *)indexPath withSender:(id)sender{
    NSLog(@"%@",NSStringFromSelector(action));
}
```

collectionView进行重新布局时调用的方法

\- (nonnull UICollectionViewTransitionLayout *)collectionView:(UICollectionView *)collectionView transitionLayoutForOldLayout:(UICollectionViewLayout *)fromLayout newLayout:(UICollectionViewLayout *)toLayout;

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

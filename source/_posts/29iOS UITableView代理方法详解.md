---
title: iOS UITableView代理方法详解
date: 2015-04-22
categories: iOS之UI控件
tags: [UITableView,iOS编程]              
---
## iOS UITableView的代理方法详解

### 一、补充

在上一篇博客中，[http://my.oschina.net/u/2340880/blog/404605](http://my.oschina.net/u/2340880/blog/404605)，我将IOS中tableView(表视图)的一些常用方法总结了一下，这篇将tableView的代理方法作了总结，对上一篇博客进行了补充。

### 二、UITableViewDataSourc（数据源代理）

#### 1、必须实现的回调方法

返回每个分区的行数

\- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section;

返回每一行的cell

\- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section;

#### 2、可选实现的方法

返回分区数(默认为1)

\- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView;   

返回每个分区头部的标题

\- (NSString *)tableView:(UITableView *)tableView titleForHeaderInSection:(NSInteger)section;

返回每个分区的尾部标题

\- (NSString *)tableView:(UITableView *)tableView titleForFooterInSection:(NSInteger)section;

设置某行是否可编辑

\- (BOOL)tableView:(UITableView *)tableView canEditRowAtIndexPath:(NSIndexPath *)indexPath;

设置某行是否可以被移动

\- (BOOL)tableView:(UITableView *)tableView canMoveRowAtIndexPath:(NSIndexPath *)indexPath;

设置索引栏标题数组（实现这个方法，会在tableView右边显示每个分区的索引）

\- (NSArray *)sectionIndexTitlesForTableView:(UITableView *)tableView; 

设置索引栏标题对应的分区

\- (NSInteger)tableView:(UITableView *)tableView sectionForSectionIndexTitle:(NSString *)title atIndex:(NSInteger)index

tableView接受编辑时调用的方法

\- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath;

这个方法中的editingStyle参数是一个枚举，代表了cell被编辑的模式，如下：

```
typedef NS_ENUM(NSInteger, UITableViewCellEditingStyle) {
    UITableViewCellEditingStyleNone,//没有编辑操作
    UITableViewCellEditingStyleDelete,//删除操作
    UITableViewCellEditingStyleInsert//插入操作
};
```

tableView的cell被移动时调用的方法

\- (void)tableView:(UITableView *)tableView moveRowAtIndexPath:(NSIndexPath *)sourceIndexPath toIndexPath:(NSIndexPath *)destinationIndexPath;

### 三、UITableViewDelegate（tableView代理）

cell将要显示时调用的方法

\- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath;

头视图将要显示时调用的方法

\- (void)tableView:(UITableView *)tableView willDisplayHeaderView:(UIView *)view forSection:(NSInteger)section;

尾视图将要显示时调用的方法

\- (void)tableView:(UITableView *)tableView willDisplayFooterView:(UIView *)view forSection:(NSInteger)section;

和上面的方法对应，这三个方法分别是cell，头视图，尾视图已经显示时调用的方法

\- (void)tableView:(UITableView *)tableView didEndDisplayingCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath*)indexPath;

\- (void)tableView:(UITableView *)tableView didEndDisplayingHeaderView:(UIView *)view forSection:(NSInteger)section;

\- (void)tableView:(UITableView *)tableView didEndDisplayingFooterView:(UIView *)view forSection:(NSInteger)section;

设置行高，头视图高度和尾视图高度的方法

\- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath;

\- (CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section;

\- (CGFloat)tableView:(UITableView *)tableView heightForFooterInSection:(NSInteger)section;

设置行高，头视图高度和尾视图高度的估计值(对于高度可变的情况下，提高效率)

\- (CGFloat)tableView:(UITableView *)tableView estimatedHeightForRowAtIndexPath:(NSIndexPath *)indexPath;

\- (CGFloat)tableView:(UITableView *)tableView estimatedHeightForHeaderInSection:(NSInteger)section;

\- (CGFloat)tableView:(UITableView *)tableView estimatedHeightForFooterInSection:(NSInteger)section;

设置自定义头视图和尾视图

\- (UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section;

\- (UIView *)tableView:(UITableView *)tableView viewForFooterInSection:(NSInteger)section; 

  
设置cell是否可以高亮

\- (BOOL)tableView:(UITableView *)tableView shouldHighlightRowAtIndexPath:(NSIndexPath *)indexPath;

cell高亮和取消高亮时分别调用的函数

\- (void)tableView:(UITableView *)tableView didHighlightRowAtIndexPath:(NSIndexPath *)indexPath;

\- (void)tableView:(UITableView *)tableView didUnhighlightRowAtIndexPath:(NSIndexPath *)indexPath;

当即将选中某行和取消选中某行时调用的函数，返回一直位置，执行选中或者取消选中

\- (NSIndexPath *)tableView:(UITableView *)tableView willSelectRowAtIndexPath:(NSIndexPath *)indexPath;

\- (NSIndexPath *)tableView:(UITableView *)tableView willDeselectRowAtIndexPath:(NSIndexPath *)indexPath;

已经选中和已经取消选中后调用的函数

\- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath;

\- (void)tableView:(UITableView *)tableView didDeselectRowAtIndexPath:(NSIndexPath *)indexPath;

设置tableView被编辑时的状态风格，如果不设置，默认都是删除风格

\- (UITableViewCellEditingStyle)tableView:(UITableView *)tableView editingStyleForRowAtIndexPath:(NSIndexPath *)indexPath;

自定义删除按钮的标题

\- (NSString *)tableView:(UITableView *)tableView titleForDeleteConfirmationButtonForRowAtIndexPath:(NSIndexPath *)indexPath;

下面这个方法是IOS8中的新方法，用于自定义创建tableView被编辑时右边的按钮，按钮类型为UITableViewRowAction。

\- (NSArray *)tableView:(UITableView *)tableView editActionsForRowAtIndexPath:(NSIndexPath *)indexPath ;

设置编辑时背景是否缩进

\- (BOOL)tableView:(UITableView *)tableView shouldIndentWhileEditingRowAtIndexPath:(NSIndexPath *)indexPath;

将要编辑和结束编辑时调用的方法

\- (void)tableView:(UITableView*)tableView willBeginEditingRowAtIndexPath:(NSIndexPath *)indexPath;

\- (void)tableView:(UITableView*)tableView didEndEditingRowAtIndexPath:(NSIndexPath *)indexPath;

  
移动特定的某行

\- (NSIndexPath *)tableView:(UITableView *)tableView targetIndexPathForMoveFromRowAtIndexPath:(NSIndexPath *)sourceIndexPath toProposedIndexPath:(NSIndexPath *)proposedDestinationIndexPath;  

疏漏之处 欢迎指正

学习使用 欢迎转载

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592
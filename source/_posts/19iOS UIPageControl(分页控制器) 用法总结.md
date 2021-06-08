---
title: iOS UIPageControl(分页控制器) 用法总结
date: 2015-04-16
categories: iOS之UI控件
tags: [UIPageControl,iOS编程]              
---
UIPageControll 是继承于UIControl的一个IOS系统UI控件，可以提供给开发者设计分页效果的功能。

初始化方法





UIPageControl * page = [[UIPageControl alloc]initWithFrame:CGRectMake(100, 100, 100, 100)];



设置控制器页数（默认为0）



@property(nonatomic) NSInteger numberOfPages;  



设置当前所在页码



@property(nonatomic) NSInteger currentPage;  



设置当总页数为1时，是否自动隐藏控制器





@property(nonatomic) BOOL hidesForSinglePage;  



设置是否延迟自动更新控制器的当前页码（默认为NO）



@property(nonatomic) BOOL defersCurrentPageDisplay;



注意：这个属性如果设置为YES，点击时并不会改变控制器显示的当前页码点，必须手动调用

- (void)updateCurrentPageDisplay; 

这个方法，才会更新。



更新控制器当前页码





- (void)updateCurrentPageDisplay; 



通过页数得到控制器大小





- (CGSize)sizeForNumberOfPages:(NSInteger)pageCount; 

这个属性用于页数会变化的情况下进行大小动态处理



设置控制器页码点得颜色





@property(nonatomic,retain) UIColor *pageIndicatorTintColor;



设置控制器当前所在页码点的颜色





@property(nonatomic,retain) UIColor *currentPageIndicatorTintColor;



学习使用 欢迎转载

专注技术，热爱生活，交流技术，也做朋友。

——珲少 QQ群：203317592


---
title: iOS开发UIScrollView使用详解
date: 2015-06-04
categories: iOS之UI控件
tags: []
---
## iOS开发UIScrollView使用详解

### 一、ScrollView常用方法和属性

[@property](http://my.oschina.net/property)(nonatomic)CGPoint contentOffset;

设置滚动的偏移量

[@property](http://my.oschina.net/property)(nonatomic)CGSize contentSize;

设置滑动区域

[@property](http://my.oschina.net/property)(nonatomic,assign) id<UIScrollViewDelegate>      delegate;

设置UIScrollView的代理

[@property](http://my.oschina.net/property)(nonatomic,getter=isDirectionalLockEnabled) BOOL directionalLockEnabled;

设置是否锁定，这个属性很有意思，默认为NO，当设置为YES时，你的滚动视图只能同一时间在一个方向上滚动，但是当你从对角线拖动时，是时刻在水平和竖直方向同时滚动的。

[@property](http://my.oschina.net/property)(nonatomic) BOOL bounces; 

设置是否开启回弹效果

@property(nonatomic) BOOL alwaysBounceVertical;

是否开启垂直方向的回弹效果

@property(nonatomic) BOOL alwaysBounceHorizontal;

是否开启水平方向的回弹效果

@property(nonatomic,getter=isPagingEnabled) BOOL pagingEnabled;

是否开启翻页效果

@property(nonatomic,getter=isScrollEnabled) BOOL scrollEnabled;  

设置是否可以滑动

@property(nonatomic) BOOL showsHorizontalScrollIndicator;

设置是否显示水平滑动条

@property(nonatomic) BOOL showsVerticalScrollIndicator;

设置是否显示竖直滑动条

@property(nonatomic) UIEdgeInsets scrollIndicatorInsets;

设置滑动条的位置

@property(nonatomic) UIScrollViewIndicatorStyle indicatorStyle;

设置滑动条风格，枚举如下：

```
typedef NS_ENUM(NSInteger, UIScrollViewIndicatorStyle) {
    UIScrollViewIndicatorStyleDefault,     //默认
    UIScrollViewIndicatorStyleBlack,       //黑色风格
    UIScrollViewIndicatorStyleWhite        //白色风格
};
```

@property(nonatomic) CGFloat decelerationRate;

设置滑动速度

\- (void)setContentOffset:(CGPoint)contentOffset animated:(BOOL)animated;

设置滚动视图内容的偏移量，可以带动画效果

\- (void)scrollRectToVisible:(CGRect)rect animated:(BOOL)animated;

设置滚动视图滚动到某个可见区域，可以带动画效果

\- (void)flashScrollIndicators;

显示一个短暂的滚动指示器

@property(nonatomic,readonly,getter=isTracking) BOOL tracking;

获取用户是否触及视图内容

@property(nonatomic,readonly,getter=isDragging) BOOL dragging;

获取用户是否开始拖动视图

@property(nonatomic,readonly,getter=isDecelerating) BOOL decelerating;

获取视图是否开始减速（用户停止拖动但视图仍在滚动）

@property(nonatomic) BOOL delaysContentTouches;

设置视图是否延迟处理触摸事件（会将消息传递给子视图）

@property(nonatomic) BOOL canCancelContentTouches;

设置是否给子视图传递取消动作的消息（默认设置为YES，当scrollView触发事件的时候，其子视图不能触发，如果设置为NO，则子视图会继续触发事件）

\- (BOOL)touchesShouldBegin:(NSSet *)touches withEvent:(UIEvent *)event inContentView:(UIView *)view;

\- (BOOL)touchesShouldCancelInContentView:(UIView *)view;

重写这两个方法可以控制起子视图的事件响应

@property(nonatomic) CGFloat minimumZoomScale;

设置内容最小缩放比例

@property(nonatomic) CGFloat maximumZoomScale; 

设置内容最大缩放比例

@property(nonatomic) CGFloat zoomScale;

设置缩放比例

\- (void)setZoomScale:(CGFloat)scale animated:(BOOL)animated;

设置缩放比例，可以带动画效果

\- (void)zoomToRect:(CGRect)rect animated:(BOOL)animated;

设置缩放显示到某个区域，可以带动画效果

@property(nonatomic) BOOL  bouncesZoom;

设置是否可以缩放回弹

@property(nonatomic,readonly,getter=isZooming) BOOL zooming;

获取是否正在缩放模式

@property(nonatomic,readonly,getter=isZoomBouncing) BOOL zoomBouncing;

获取是否当前的缩放比例超出设置的峰值

@property(nonatomic) BOOL  scrollsToTop;

设置是否点击状态栏滚动到scrollView的最上端

@property(nonatomic) UIScrollViewKeyboardDismissMode keyboardDismissMode;

设置键盘消失的模式，枚举如下：

```
typedef NS_ENUM(NSInteger, UIScrollViewKeyboardDismissMode) {
    UIScrollViewKeyboardDismissModeNone,
    UIScrollViewKeyboardDismissModeOnDrag,      //手指滑动视图键盘就会消失
    UIScrollViewKeyboardDismissModeInteractive, //手指滑动视图后可以与键盘交互，上下滑动键盘会跟随手指上下移动
};
```

### 二、ScrollViewDelegata中常用方法

\- (void)scrollViewDidScroll:(UIScrollView *)scrollView; 

视图已经开始滑动时触发的方法

\- (void)scrollViewDidZoom:(UIScrollView *)scrollView;

视图已经开始缩放时触发的方法

\- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView;

视图开始拖动时触发的方法

\- (void)scrollViewWillEndDragging:(UIScrollView *)scrollView withVelocity:(CGPoint)velocity targetContentOffset:(inout CGPoint *)targetContentOffset;

\- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate;

视图拖动结束时触发的方法

\- (void)scrollViewWillBeginDecelerating:(UIScrollView *)scrollView; 

视图开始减速时触发的方法

\- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView; 

视图减速结束时触发的方法

\- (void)scrollViewDidEndScrollingAnimation:(UIScrollView *)scrollView;

视图动画结束时触发的方法，使用set方法设置偏移量后回触发

\- (UIView *)viewForZoomingInScrollView:(UIScrollView *)scrollView;

返回进行缩放的视图

\- (void)scrollViewWillBeginZooming:(UIScrollView *)scrollView withView:(UIView *)view;

视图内容将要开始缩放时触发的方法

\- (void)scrollViewDidEndZooming:(UIScrollView *)scrollView withView:(UIView *)view atScale:(CGFloat)scale;

视图内容结束缩放时触发的方法

\- (BOOL)scrollViewShouldScrollToTop:(UIScrollView *)scrollView; 

返回yes，开启快捷滚动回顶端，将要滚动时调用

\- (void)scrollViewDidScrollToTop:(UIScrollView *)scrollView;

视图快捷滚动回顶端开始动作时调用

疏漏之处 欢迎指正

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

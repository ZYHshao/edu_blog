---
title: MJRefresh源码分析
date: 2018-01-30
categories: iOS第三方库
tags: []
---
## MJRefresh源码分析

>   每次读优秀的代码都是一次深刻的学习，每一次模仿，都是创造的开始！
> 
> ——QQ 316045346 欢迎交流

### 一、MJRefresh源码结构分析 

    MJRefresh主要为UIScrollView，UITableView和UICollectionView添加头部和尾部刷新控件。其主要由3大块组成，类别工具，核心UIScrollView类别和头部尾部刷新组件。如下图：

![](https://static.oschina.net/uploads/space/2018/0130/180314_SYvo_2340880.png)

### 二、工具类别

    上面示意图中列出的几个工具类别主要提供方便属性访问的功能。其主要是为了方便MJRefresh库自己的调用，当然你也可以对它进行使用。

UIView+MJExtension类别提供了对UIView组件位置和尺寸的快速访问方法，并且都支持快速获取和设置：

```objectivec
@property (assign, nonatomic) CGFloat mj_x;
@property (assign, nonatomic) CGFloat mj_y;
@property (assign, nonatomic) CGFloat mj_w;
@property (assign, nonatomic) CGFloat mj_h;
@property (assign, nonatomic) CGSize mj_size;
@property (assign, nonatomic) CGPoint mj_origin;
```

UIScrollView+MJExtension提供了对UIScrollView的内容尺寸，偏移量等属性的快速访问：

```objectivec
@property (readonly, nonatomic) UIEdgeInsets mj_inset;

@property (assign, nonatomic) CGFloat mj_insetT;
@property (assign, nonatomic) CGFloat mj_insetB;
@property (assign, nonatomic) CGFloat mj_insetL;
@property (assign, nonatomic) CGFloat mj_insetR;

@property (assign, nonatomic) CGFloat mj_offsetX;
@property (assign, nonatomic) CGFloat mj_offsetY;

@property (assign, nonatomic) CGFloat mj_contentW;
@property (assign, nonatomic) CGFloat mj_contentH;
```

NSBundle+MJRefresh这个类别提供对库中资源的访问方法：

```objectivec
+ (instancetype)mj_refreshBundle;
//获取箭头图片
+ (UIImage *)mj_arrowImage;
//获取国际化字符串
+ (NSString *)mj_localizedStringForKey:(NSString *)key value:(NSString *)value;
+ (NSString *)mj_localizedStringForKey:(NSString *)key;
```

### 三、关于UIScrollView+MJRefresh

    这个类别是MJRefresh库的核心，其中提供的mj\_header和mj\_footer两个属性用来添加头部和尾部刷新组件。这两个组件是作为子视图添加在UIScrollView上的，因此和UIScrollView的原生头尾视图都不影响。在以前版本的MJRefresh中，使用的是header和footer属性，容易产生疑惑，因此后面版本框架中都添加了mj前缀。

    UIScrollView+MJRefresh类别在开发者设置mj\_header和mj\_footer属性时，将这两个组件添加为当前滚动视图的最下层子视图，为了满足某些自动加载的需求，这里面有用runtime将UITableView和UICollectionView的reload函数进行替换，这样做的目的是为了在数据加载时统计界面的元素个数。

### 四、刷新组件

    MJRefreshComponent是刷新组件的基类，其中定了一些通用方法。首先，MJRefresh库的刷新组件核心思想是基于状态的，即通过状态来触发某些组件行为，例如正常的常态，下拉的pulling态，释放的refreshing态等等。开发者除了可以手动设置状态外，主要通过监听UIScrollView的偏移量等属性来改变状态。当UIScrollView有偏移量或内容尺寸的变化时，MJRefreshComponent会调用scrollViewContentOffsetDidChange函数，这个函数主要交给其子类实现。

    MJRefreshHeader类是头部刷新组件的基类，其将刷新组件布局在UIScrollView组件的顶部，并且封装了记录上次刷新时间的功能。MJRefreshStateHeader提供了接口供开发者设置不同状态下刷新组件所显示的文字，MJRefreshNormalHeader是一个更加上层的头部刷新组件，其状态文字是默认定义好的，并且支持国际化。MJRefreshGifHeader可以支持显示自定义刷新动画，其可以为某个状态设置一组图片。

    尾部刷新组件的编写逻辑和头部刷新组件的编写逻辑基本一致，MJRefresh中的尾部刷新组件分为了两类，一类是刷新完成后自动消失的，一类是自动刷新，刷新完成后不会自动消失，只是改变状态。MFRefreshFooter与MJRefreshHeader的实现基本一致，MJRefreshBackFooter有刷新完成后自动还原的功能，MJRefreshBackNormalFooter是比较上层的封装，其显示默认的状态文案，并且支持国际化。MJRefreshBackStateFooter则可以手动设置不同状态下刷新组件显示的文字。MJRefreshBackGifFooter用来显示自定义动画的尾部刷新组件。MJRefreshAutoFooter是自动尾部刷新组件的基类，其可以设置当尾部刷新组件出现多少比例时进行刷新(默认是完全出现后进行刷新)。MJRefreshAutoStateFooter可以自定义其各个状态的文案。同样，也有比较上层的MJRefreshAotuNormalFooter组件，这个组件封装好了国际化的文案可以直接使用，MJRefreshAutoGifFooter组件可以显示自定动画的尾部刷新。

### 五、MJRefresh中的编程风格技巧与小亮点

#### 1.复用，复用，再复用

    之所以看MJRefresh库的代码非常舒服，很大一部分源自其深入的复用。首先MJRefreshComponent类抽象出了回调与刷新函数，并且提取出了需要子类复写的通用的布局、监听等函数，让子类的结构非常统一。MJRefreshHeader和MJRefreshFooter作为头部与尾部刷新组件的基类，抽象出了构造函数，并且实现了大部分组件与外部的布局，逻辑动作等函数。再子类则专注与实现子类自身的UI与功能。还有一个小细节，也可以看出MJRefresh对复用的追求，在setState函数的实现中，如果新的状态与旧的状态一致，则不需要做任何逻辑，所有的setState函数都需要这个逻辑，MJRefresh中采用的宏的方式进行替换，使代码变得十分简洁，示例如下：

```objectivec
#define MJRefreshCheckState \
MJRefreshState oldState = self.state; \
if (state == oldState) return; \
[super setState:state];


///////////
- (void)setState:(MJRefreshState)state
{
    MJRefreshCheckState
    
    // 设置状态文字
    self.stateLabel.text = self.stateTitles[@(state)];
}
```

#### 2.另一种安全执行block的方式

    很多时候，我们在执行block的时候都会先检查下这个block是否为nil，下面是我们常用的代码：

```objectivec
if (block) {
     block();
}
```

在MJRefresh中有使用问号冒号的方式来代替if语句，如下：

```objectivec
- (void)executeReloadDataBlock
{
    ! self.mj_reloadDataBlock ? : self.mj_reloadDataBlock(self.mj_totalDataCount);
}
```

这个表达式初看会有一些疑惑，其实?:的作用是返回一个值，如果?:前的表达式为nil的话，则会返回?:后面的值，同样，如果?:前面的表达式不为nil的话，则直接返回，不会执行到后面的表达式，上面的写法其实和第一种if语句的作用完全一致。

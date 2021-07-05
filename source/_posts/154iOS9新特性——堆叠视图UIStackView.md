---
title: iOS9新特性——堆叠视图UIStackView
date: 2015-11-17
categories: iOS之UI控件
tags: []
---
## iOS9新特性——堆叠视图UIStackView

### 一、引言

        随着autolayout的推广开来，更多的app开始使用自动布局的方式来构建自己的UI系统，autolayout配合storyBoard和一些第三方的框架，对于创建约束来说，已经十分方便，但是对于一些动态的线性布局的视图，我们需要手动添加的约束不仅非常多，而且如果我们需要插入或者移除其中的一些UI元素的时候，我们又要做大量的修改约束的工作，UIStackView正好可以解决这样的问题。

### 二、在storyBoard上初识StackView

        UIStackView是一个管理一组堆叠视图的控制器类视图，所谓堆叠视图时一种平铺式的线性布局方式，不可重叠，布局方向也不可交错，如果你做过watchOS的开发，你会发现，其实StackView与watchOS中的group十分能相似。

例如，我们如果需要一个如下效果的布局，在屏幕的中间摆放几个大小一致的色块，无论屏幕朝向如何，其位置都不会变化，并且可以向其中添加和移除色块的数量：

![](http://static.oschina.net/uploads/space/2015/1117/111558_HjYg_2340880.png)       ![](http://static.oschina.net/uploads/space/2015/1117/111558_gGUi_2340880.png)

首先，我们在ViewController中拉入一个stackView：

![](http://static.oschina.net/uploads/space/2015/1117/111721_LIF0_2340880.png)

将一些属性设置如下：

![](http://static.oschina.net/uploads/space/2015/1117/111830_LQxh_2340880.png)

Axis是设置布局的方向，有水平和垂直两种方式，一个StackView只能选择一种布局模式。

Alignment是选择其管理视图的对齐模式，我们这里选择充满。

Distribution是设置其管理视图的排列方式，我们选择等宽充满。

Spacing是设置视图之间的间距，设置为10.

之后有一点需要注意，stackView用于布局其内部管理的视图，对于它本身，我们还需要添加一些约束，将它约束在屏幕的中间。

我们向其中拖入任意数量的view，设置不同的颜色，就实现了我们想要的效果，并且可以随意动态删除和添加其中的view数量，不需要改变约束。

### 三、从代码学习UIStackView

        通过代码创建一个UIStackView也非常简单，首先，我们先通过代码实现上面的效果：

```
 NSMutableArray * array = [[NSMutableArray alloc]init];
    for (int i =0 ; i<5; i++) {
        UIView * view = [[UIView alloc]init];
        view.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
        [array addObject:view];
    }
    UIStackView * stackView = [[UIStackView alloc]initWithArrangedSubviews:array];
    [self.view addSubview:stackView];
    [stackView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.centerX.equalTo(self.view.mas_centerX);
        make.centerY.equalTo(self.view.mas_centerY);
        make.leading.equalTo(self.view.mas_leading).offset(20);
        make.trailing.equalTo(self.view.mas_trailing).offset(-20);
        make.size.height.equalTo(@100);
    }];
    stackView.axis = UILayoutConstraintAxisHorizontal;
    stackView.distribution = UIStackViewDistributionFillEqually;
    stackView.alignment = UIStackViewAlignmentFill;
```

效果图如下：

![](http://static.oschina.net/uploads/space/2015/1117/131959_4APz_2340880.png)   ![](http://static.oschina.net/uploads/space/2015/1117/131959_TK5Y_2340880.png)

我们的布局没有问题，并且可以动态的改变其中view的个数，使用如下方法添加一个view：

```
    UIView * newView = [[UIView alloc]init];
    newView.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    [stackView addArrangedSubview:newView];
```

与之相对，我们可以使用下面的方法移除一个view：

```
    UIView * view = [stackView arrangedSubviews].lastObject;
    [stackView removeArrangedSubview:view];
```

特别注意：addArrangedSubview和addSubview有很大的区别，使用前者是将试图添加进StackView的布局管理，后者只是简单的加在试图的层级上，并不接受StackView的布局管理。

技巧：因为StackView继承于UIView，因此在布局改变的时候，我们可以使用UIView层的动画，如下：

```
        //在添加view的时候会有动画效果，移除的时候没有
        [stackView addArrangedSubview:newView];
        [UIView animateWithDuration:1 animations:^{
            [stackView layoutIfNeeded];
        }];
```

### 四、再来深入理解下UIStackView

        通过上面的介绍，我们已经基本了解了StackView的使用和特点，下面我们再来仔细介绍一下与其相关的属性和方法的使用，使我们能够更加得心应手。

有关被管理视图的添加与移除：

```
//初始化方法，通过数组传入被管理的视图
- (instancetype)initWithArrangedSubviews:(NSArray<__kindof UIView *> *)views; 
//获取被管理的所有视图
@property(nonatomic,readonly,copy) NSArray<__kindof UIView *> *arrangedSubviews;
//添加一个视图进行管理
- (void)addArrangedSubview:(UIView *)view;
//移除一个被管理的视图
- (void)removeArrangedSubview:(UIView *)view;
//在指定位置插入一个被管理的视图
- (void)insertArrangedSubview:(UIView *)view atIndex:(NSUInteger)stackIndex;
```

与StackView布局设置相关：

1.布局模式：

```
@property(nonatomic) UILayoutConstraintAxis axis;
```

上面这个属性用于设置布局的模型，枚举如下：

```
//stackView只有两种布局模式 水平和竖直
typedef NS_ENUM(NSInteger, UILayoutConstraintAxis) {
    //水平布局
    UILayoutConstraintAxisHorizontal = 0,
    //竖直布局
    UILayoutConstraintAxisVertical = 1
};
```

2.对齐模式： 

```
@property(nonatomic) UIStackViewAlignment alignment;
```

这个属性用于设置控件的对其模式，枚举如下：

```
typedef NS_ENUM(NSInteger, UIStackViewAlignment) {
   //水平布局时为高度充满，竖直布局时为宽度充满
    UIStackViewAlignmentFill,
    //前边对其
    UIStackViewAlignmentLeading,
    //顶部对其
    UIStackViewAlignmentTop = UIStackViewAlignmentLeading,
    //第一个控件文字的基线对其 水平布局有效
    UIStackViewAlignmentFirstBaseline, 
    //中心对其
    UIStackViewAlignmentCenter,
    //后边对其
    UIStackViewAlignmentTrailing,
    //底部对其
    UIStackViewAlignmentBottom = UIStackViewAlignmentTrailing,
    //基线对其，水平布局有效
    UIStackViewAlignmentLastBaseline, 
} NS_ENUM_AVAILABLE_IOS(9_0);
```

在上面的例子中，我们设置了对其方式为充满，这样的话，我们就不需要再做过多控件尺寸的约束，如果我们被管理的控件高度或者宽度不一，我们可以设置中心对其，这样的话，我们还需要为每个控件添加一个宽度或者高度的约束，如下：

```
    NSMutableArray * array = [[NSMutableArray alloc]init];
    for (int i =0 ; i<5; i++) {
        UIView * view = [[UIView alloc]init];
        view.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
        float height = arc4random()%90+10;
        [view mas_makeConstraints:^(MASConstraintMaker *make) {
            make.height.equalTo([NSNumber numberWithFloat:height]);
        }];
        [array addObject:view];
    }
    stackView = [[UIStackView alloc]initWithArrangedSubviews:array];
    [self.view addSubview:stackView];
    [stackView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.centerX.equalTo(self.view.mas_centerX);
        make.centerY.equalTo(self.view.mas_centerY);
        make.leading.equalTo(self.view.mas_leading).offset(20);
        make.trailing.equalTo(self.view.mas_trailing).offset(-20);
        make.size.height.equalTo(@100);
    }];
    stackView.axis = UILayoutConstraintAxisHorizontal;
    stackView.distribution = UIStackViewDistributionFillEqually;
    stackView.alignment = UIStackViewAlignmentCenter;
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/1117/134827_ltLN_2340880.png)  ![](http://static.oschina.net/uploads/space/2015/1117/134827_sbPw_2340880.png)

这样，参差不齐的控件布局我们也可以轻松完成。

3.排列方式

```
@property(nonatomic) UIStackViewDistribution distribution;
```

排列方式的枚举如下：

```
typedef NS_ENUM(NSInteger, UIStackViewDistribution) {
    //充满，当只有一个控件时可以使用
    UIStackViewDistributionFill = 0,
    //平分充满，每个控件占据相同尺寸排列充满
    UIStackViewDistributionFillEqually,
    //会优先按照约束的尺寸进行排列，如果没有充满，会拉伸最后一个排列的控件充满
    UIStackViewDistributionFillProportionally,
    //等间距排列
    UIStackViewDistributionEqualSpacing,
    //中心距离相等
    UIStackViewDistributionEqualCentering,
} NS_ENUM_AVAILABLE_IOS(9_0);
```

注意，除了我们选择fill属性时不需约束控件视图的尺寸，其他都需要进行约束，例如如果我们选择等间距，我把改成如下代码：

```
     [view mas_makeConstraints:^(MASConstraintMaker *make) {
            make.height.equalTo([NSNumber numberWithFloat:height]);
            make.width.equalTo(@50);
            
        }];
     stackView.distribution = UIStackViewDistributionEqualSpacing;
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/1117/140235_sULx_2340880.png)![](http://static.oschina.net/uploads/space/2015/1117/140235_O0Fr_2340880.png)

4.其他

```
//设置最小间距
@property(nonatomic) CGFloat spacing;
//设置布局时是否参照基线
@property(nonatomic,getter=isBaselineRelativeArrangement) BOOL baselineRelativeArrangement;
//设置布局时是否以控件的LayoutMargins为标准，默认为NO，是以控件的bounds为标准
@property(nonatomic,getter=isLayoutMarginsRelativeArrangement) BOOL layoutMarginsRelativeArrangement;
```

### 五、UIStackView的嵌套

        一个StackView不允许我们进行水平和竖直的交叉布局，但是我们可以通过嵌套的方式来实现复杂的布局效果，比如我们实现一个类似电影表标签，可以使用水平布局的StackView中嵌套一个竖直布局的StackView：

![](http://static.oschina.net/uploads/space/2015/1117/151409_XvEZ_2340880.png)

十分轻松就可以实现如下的效果：

![](http://static.oschina.net/uploads/space/2015/1117/151437_lsw9_2340880.png)  ![](http://static.oschina.net/uploads/space/2015/1117/151501_QWVK_2340880.png)

看到了吧，通过StackView，我们没有添加过多的约束，使我们布局起来更加轻松了。如果你常常使用storyBoard进行开发，还有一个小技巧可以方便的将两个控件整合到一个StackView中，按住command，选中两个控件，之后点击右下角的如下图标，系统会自动帮我们生成一个StackView，将选中的两个控件整合进去，很酷吧！

![](http://static.oschina.net/uploads/space/2015/1117/151838_YJKS_2340880.png)

  
 
> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

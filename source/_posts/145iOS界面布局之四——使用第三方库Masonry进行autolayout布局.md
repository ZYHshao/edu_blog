---
title: iOS界面布局之四——使用第三方库Masonry进行autolayout布局
date: 2015-11-02
categories: iOS之UI控件
tags: []
---
## iOS界面布局之四——使用第三方库Masonry进行autolayout布局

### 一、引言

        在前面博客，我们讨论了使用iOS原生的框架代码来进行autolayout布局。在使用中，我们会发现，无论是代码量还是结构的清晰度，都十分不能让我们满意，在storyBoard中只需要几条线就可以搞定的事情，用代码缺要写冗余的一大堆。并且有些时候，故事版并不能解决所有问题，某些控件必须我们手写，这样的话，我们就不得不进行代码的autolayout布局，幸运的是，Masonry可以帮助我们轻松愉快的完成这一任务。

使用代码进行autolayout布局：[http://my.oschina.net/u/2340880/blog ](http://my.oschina.net/u/2340880/blog)。

### 二、使用Masonry

        这里说的大部分内容均来自Masonry和官方gitHub，将其内容进行了翻译和解释，源地址如下：[https://github.com/SnapKit/Masonry](https://github.com/SnapKit/Masonry)。

#### 1、布局的控件属性对照

        无论是用storyBoard还是代码，在设置控件之间layout关系的时候，我们都需要设置控件的位置属性。在下面的方法中，这个位置属性就是NSLayoutAttribute对象，他决定的控件对象的参照位置：

```
+(instancetype)constraintWithItem:(id)view1 attribute:(NSLayoutAttribute)attr1 
                                            relatedBy:(NSLayoutRelation)relation 
                                            toItem:(nullable id)view2 
                                            attribute:(NSLayoutAttribute)attr2 
                                            multiplier:(CGFloat)multiplier constant:(CGFloat)c;
```

在Masonry中，有一系列的属性与之成对应关系，对照如下：

![](http://static.oschina.net/uploads/space/2015/1102/105457_YEgT_2340880.png)

#### 2、3个方法让你玩转Masonry约束操作

        Masonry在UIView的类别中，有3个全局的操作约束的方法，通过他们我们可以自由的进行autolayout的设置。

添加约束：

```
- (NSArray *)mas_makeConstraints:(void(^)(MASConstraintMaker *make))block;
```

这个方法用于我们在最开始时为控件设置的约束，在block中进行约束条件的设置，例如我们创建一个label，将其尺寸设置为50*50，放在屏幕中间，使用如下代码：

注意：在添加约束前，必须将视图添加到其父视图上。

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    UILabel * label = [[UILabel alloc]init];
    [self.view addSubview:label];
    [label mas_makeConstraints:^(MASConstraintMaker *make) {
        make.center.equalTo(self.view);
        make.height.equalTo(@50);
        make.width.equalTo(@50);
    }];
    label.backgroundColor = [UIColor redColor];
}
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/1102/111959_GF0u_2340880.png)

更新约束：

当我们需要配合布局改变或者动画效果的时候，我们可能需要将已经添加的约束进行更新操作，使用如下的方法：

```
[label mas_updateConstraints:^(MASConstraintMaker *make) {
        make.height.equalTo(@100);
        make.width.equalTo(@100);
    }];
```

更新约束的作用在于更新已经添加的某些约束，并不会移除掉原有的约束，如果我们需要添加新的约束，可以使用下面的重设约束的方法。

重设约束：

```
[label mas_remakeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(self.view.mas_left).offset(10);
        make.top.equalTo(self.view.mas_top).offset(100);
        make.height.equalTo(@100);
        make.width.equalTo(@100);
    }];
```

#### 3、约束值相关

        在添加具体约束的时候，我们不仅可以将约束值设置为绝对的相等关系，也可以设置一些值域的关系，在Masonry中，有如下三种：

```
//绝对相等
- (MASConstraint * (^)(id attr))equalTo;
//大于等于
- (MASConstraint * (^)(id attr))greaterThanOrEqualTo;
//小于等于
- (MASConstraint * (^)(id attr))lessThanOrEqualTo;
```

对于约束的优先级，使用如下几个量：

```
//手动设置一个优先级参数
- (MASConstraint * (^)(MASLayoutPriority priority))priority;
//优先级低
- (MASConstraint * (^)())priorityLow;
//优先级中等
- (MASConstraint * (^)())priorityMedium;
//优先级高
- (MASConstraint * (^)())priorityHigh;
```

写法如下：

```
[label mas_remakeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(self.view.mas_left).offset(10);
        make.top.equalTo(self.view.mas_top).offset(100);
        make.height.equalTo(@100).priority(1000);
        make.width.equalTo(@100).priorityHigh();
    }];
```

### 三、Masonry设置约束的几个示例

#### 1、设置视图与其父视图的边距约束

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    label = [[UILabel alloc]init];
    [self.view addSubview:label];
    [label mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.equalTo(self.view).insets(UIEdgeInsetsMake(20, 20, 20, 20));
    }];
    label.backgroundColor = [UIColor redColor];
}
```

设置上下左右与其父视图边距为20px，效果如下：

![](http://static.oschina.net/uploads/space/2015/1102/115707_bisI_2340880.png) 

#### 2、约束控件的尺寸为固定值

```
[label mas_makeConstraints:^(MASConstraintMaker *make) {
        make.height.equalTo(@200);
        make.width.equalTo(@200);
        make.center.equalTo(self.view);
    }];
```

位置约束设置在了屏幕的中间，效果如下：

![](http://static.oschina.net/uploads/space/2015/1102/134638_e4Db_2340880.png)

#### 3、约束控件之间的尺寸

```
  [label mas_makeConstraints:^(MASConstraintMaker *make) {
        make.height.equalTo(@100);
        make.width.equalTo(label2);
        make.right.equalTo(label2.mas_left).offset(-100);
        make.leading.equalTo(self.view.mas_leading).offset(20);
        make.centerY.equalTo(self.view);
    }];
    [label2 mas_makeConstraints:^(MASConstraintMaker *make) {
        make.height.equalTo(@100);
        make.centerY.equalTo(label);
        make.trailing.equalTo(self.view.mas_trailing).offset(-20);
    }];
```

设置了两个label宽度一致，相距100px,分别距离左右边距20px，效果如下：

![](http://static.oschina.net/uploads/space/2015/1102/141825_m5VX_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

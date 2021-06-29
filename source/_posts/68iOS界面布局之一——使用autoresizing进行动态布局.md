---
title: iOS界面布局之一——使用autoresizing进行动态布局
date: 2015-06-01
categories: iOS之UI控件
tags: []
---
## iOS界面布局之一——使用autoresizing进行动态布局

autoresizing是iOS中传统的界面自动布局方式，通过它，当父视图frame变换时，子视图会自动的做出相应的调整。

### 一、通过代码进行布局

任何一个view都有autoresizingMask这个属性，通过这个属性可以设置当前view与其父视图的相对关系。我们先来看UIViewAutoresizing这个枚举：

```
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
    UIViewAutoresizingNone                 = 0,//默认
    UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,//与父视图右边间距固定，左边可变
    UIViewAutoresizingFlexibleWidth        = 1 << 1,//视图宽度可变
    UIViewAutoresizingFlexibleRightMargin  = 1 << 2,//与父视图左边间距固定，右边可变
    UIViewAutoresizingFlexibleTopMargin    = 1 << 3,//与父视图下边间距固定，上边可变
    UIViewAutoresizingFlexibleHeight       = 1 << 4,//视图高度可变
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5//与父视图上边间距固定，下边可变
};
```

下面我们通过效果来看这些属性的作用：

先创建两个view，为了区分，设置不同的背景色：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    UIView * view1 = [[UIView alloc]initWithFrame:CGRectMake(20, 40, 200, 200)];
    view1.backgroundColor=[UIColor redColor];
    UIView * view2 = [[UIView alloc]initWithFrame:CGRectMake(10, 10, 100, 100)];
    view2.backgroundColor=[UIColor greenColor];
    [view1 addSubview:view2];
    [self.view addSubview:view1];
}
```

设置view2的自动布局属性如下：

```
 view2.autoresizingMask=UIViewAutoresizingFlexibleBottomMargin;
```

这时的效果如下：

![](http://static.oschina.net/uploads/space/2015/0601/124749_YHKL_2340880.png)

改变view1的frame如下：

```
UIView * view1 = [[UIView alloc]initWithFrame:CGRectMake(20, 40, 300, 300)];
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/0601/125004_BrA5_2340880.png)

这时view2的下边距离相对父视图是可变的。

设置如下：

```
   view2.autoresizingMask=UIViewAutoresizingFlexibleHeight;
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/0601/125558_ruiz_2340880.png)

可以看出，这时子视图的高度是随父视图变化而自动改变的。

如下设置：

```
view2.autoresizingMask=UIViewAutoresizingFlexibleLeftMargin;
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/0601/125747_JRJo_2340880.png)

这时子视图的左边是随父视图变化而可变的。

同理，UIViewAutoresizingFlexibleRightMargin将使子视图右边与父视图的距离可变。

UIViewAutoresizingFlexibleTopMargin将使子视图上边与父视图距离可变。UIViewAutoresizingFlexibleWidth将使子视图的宽度可变。

注意：这些自动布局的属性是可以叠加的，比如保持视图与父视图边距不变，如下设置：

```
view2.autoresizingMask=UIViewAutoresizingFlexibleWidth|UIViewAutoresizingFlexibleHeight;
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/0601/130449_PZ1l_2340880.png)

### 二、nib文件中可视化设置自动布局

在storyboard中我们可以更加轻松的进行autoresizing自动布局。在view设置栏中有autoresizing这个设置，点中相应的箭头，就是刚才我们探讨的设置选项。并且我们把鼠标放在这个上面的时候，右侧会自动为我们预览效果。

![](http://static.oschina.net/uploads/space/2015/0601/130857_8sRL_2340880.png)

如果你觉得autoresizing很强大，那么你就太容易满足了，autoresizing可以满足大部分简单的自动布局需求，可是它有一个致命的缺陷，它只能设置子视图相对于父视图的变化，却不能精确这个变化的度是多少，因此对于复杂的精准的布局需求，它就力不从心了。但是有一个好消息告诉你，iOS6之后的autolayout自动布局方案，正是解决复杂布局的好帮手，我们在下一遍博客中再进行详细讨论。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

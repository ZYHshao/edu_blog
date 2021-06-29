---
title: iOS界面布局之二——初识autolayout布局模型
date: 2015-06-01
categories: iOS之UI控件
tags: []
---
## iOS界面布局之二——初识autolayout布局模型

### 一、引言

     在上一篇博客中介绍了传统的布局方式：autoresizing。随着iphone型号的越来越多，屏幕的标准也更加多样化，通过autoresizing已经不能满足开发的需求，而进行两套布局或者动态代码控制又大大增加了开发者的工作量，autolayout的出现拯救个这一切，它让动态布局变的十分简单便捷。

    autoresizing介绍：[http://my.oschina.net/u/2340880/blog/423357](http://my.oschina.net/u/2340880/blog/423357)。

### 二、autolayout的设计思想

    正如storyboard的设计目的是为了让开发者将更多的精力投入到逻辑实现而不是界面布局一样。autolayout的设计思想是让开发者将布局上更多的精力放在控件关系上而不是坐标。我们只需要关心控件之间的摆放关系，而并不需要关心这是如何实现的。因此你使用autolayout进行布局时，就是在添加一个一个的约束。控件与控件之间的约束，控件与父视图之间的约束。

#### 1、了解几种约束

    点击xcode的storyboard文件，在xcode的导航栏上点击Edito，然后选择Pin，可以看到如图，其中是可以添加的约束类型。

![](http://static.oschina.net/uploads/space/2015/0601/154914_EdDr_2340880.png)

Width：对视图宽度的约束

Height：对视图高度的约束

Horizontal Spacing：对视图间水平距离的约束

Vertical Spacing：对视图间垂直距离的约束

Leading Space to Superview：与父视图左边界的约束

Trailing Space to Superview：与父视图右边界的约束

Top Space to Superview：与父视图上边界的约束

Bottom Space to Superview：与父视图下边界的约束

Widehs Equally：视图等宽约束

Heights Equally：视图等高约束

### 2、网上的一个很简单的约束例子

    了解了上面的几种约束，现在我们来实现一个效果，借用网上关于autolayout自动布局的一个小例子。我们在storyboard中拖入三个label，使它们如下效果：

![](http://static.oschina.net/uploads/space/2015/0601/155935_qJrn_2340880.png)

然后我们将屏幕横过来，会发现这时的效果并不是我们想得到的结果：

![](http://static.oschina.net/uploads/space/2015/0601/160033_8vCZ_2340880.png)

在进行添加约束之前，我们先来理清这三个视图之间的关系，将上面两个视图编号为1.2，下面那个视图编号为3.

（1）1和2的宽和高相等

（2）1距离父视图左边20px

（3）2距离父视图右边20px

（4）3距离父视图左边20px，右边20px

（5）1和2水平间距20px

（6）1与3垂直间距20px

（7）1和2距离父视图上边距50px

（8）3距离父视图下边距20px

（9）3与1和2的高度一样

通过上面的约束，所有视图的位置都将被相对的固定，下面我们只需要按照顺序一一添加即可。

（1）选中1和2视图（按住cmd键可以多选），然后点击Editor->Pin之后选择Widehs Equally，重复上面的过程，选择Heights Equally。我们会看到如下的效果：

![](http://static.oschina.net/uploads/space/2015/0601/161110_qokG_2340880.png)

几点注意：

*线是橙色代表警告，我们没有添加足够的约束来确定位置或者约束有矛盾。

*如果线的中间显示的不是等号，而是数字，则是因为视图1和2的尺寸设置的不等，约束有矛盾。

（2）选中1.重复上面步骤，选择Leading Space to Superview。这时1的左边又会增加一条线：

![](http://static.oschina.net/uploads/space/2015/0601/161620_KkFt_2340880.png)

点击这条线，在右边的设置去将约束值设置为20：

![](http://static.oschina.net/uploads/space/2015/0601/161707_WNXc_2340880.png)

（3）重复上面步骤，选中视图2，添加Trailing Space to Superview约束。

（4）选中视图3，重复上面步骤。

（5）选中1和2，添加Horizontal Spacing，设置为20.

（6）选中1和3，添加Vertical Spacing，设置为20.

（7）为1和2分别添加Top Space to Superview约束。

（8）为3添加Bottom Space to Superview约束。

（9）选中1和3，添加Heights Equally约束。

上面的过程虽然繁琐，但是逻辑性十分清晰，这时你会发现所有的线都变成了蓝色，约束已经添加完整，我们再次运行后横屏，效果如下：

![](http://static.oschina.net/uploads/space/2015/0601/164305_x3yK_2340880.png)

这就是我们想要的结果了。

#### 3、自动布局的几种对其方式

    在xcode导航的Editor菜单中，还有一个子菜单，Align，这里面的选项可以为控件添加对其约束：

![](http://static.oschina.net/uploads/space/2015/0601/165558_w0S0_2340880.png)

Left Edges：控件左对齐

Right Edges：控件右对齐

Top Edges：控件上对齐

Bottom Edges：控件下对齐

Horizontal Centers：控件水平中心对齐

Vertical Centers：控件垂直水平对齐

Horizontal Center in Container：控件与其父视图水平中心对齐

Vertical Center in Container：控件与其父视图垂直中心对齐

### 三、几点小感悟

     到此为止，基本上已经可以使用autolayout自动布局解决复杂的布局需求了，但是切记，正式因为aotulayout的强大使它会隐藏更多的坑，下面是我的几点感悟，再次分享：

1、autolayout的精髓在于足够多的约束，autolayout之所以比autoresizing强大，就在于其布局的精确性，而精确性正是由约束来提供的。

2、切莫画蛇添足，矛盾的约束会使xcode晕掉，所以在添加约束前，我建议将试图间的布局关系先整理出来。

3、应该转变你的思路，如果你已经习惯了使用CGRect、Point等传统的坐标布局模式，那么你应该稍微转变一下，autolayout倡导的是一个相对的概念，你需要将更多的关注放在视图间的关系，比如A和B距离10，A和C右对齐等。具体的坐标会有autolayout帮你算。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

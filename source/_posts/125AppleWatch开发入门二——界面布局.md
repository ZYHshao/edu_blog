---
title: AppleWatch开发入门二——界面布局
date: 2015-10-14
categories: Apple Watch开发手记
tags: []
---
## AppleWatch开发入门二——界面布局

### 一、简介

        在iphone开发中，最基本的布局方式是通过frame，将控件的位置和大小固定在屏幕上，后来，由于手机屏幕的尺寸有了略微变化，有了autoresizing的布局框架，我们可以设置子视图随父视图的改变做一些相应的变化，再后来，iphone的尺寸与分辨率也越来越多，适配各个屏幕也成为了iOS开发者遇到的新的问题，幸运的是，autolayout机制的出现，大大减小了开发者在适配方面的成本。以上提到的两种布局方式，在以前博客中有讨论：

使用autoresizing进行动态布局：[http://my.oschina.net/u/2340880/blog/423357](http://my.oschina.net/u/2340880/blog/423357)

使用autolayout进行动态布局：[http://my.oschina.net/u/2340880/blog/423500](http://my.oschina.net/u/2340880/blog/423500)

        在watch的布局方式中，我们需要抛弃iphone里的思路，重新接受一套新的布局框架。

        首先，watch的屏幕不大，目前只有38mm和42mm两个尺寸，我们不可能在这个有限的空间里做非常复杂的界面效果，因此，在界面开发中，应该遵循便于使用和一目了然的原则。watch上的布局方式采用的是一种平面堆放的方式，不再有frame，也不再有约束，控件的布局方式只是一个挨着一个的平面堆放，也不可重叠。但在watch中，提供了group这样一种布局方式，可以让我们在布局中体现自由与个性的方面。         

### 二、最基础的堆放布局

        我们在不使用group的时候，watch的布局采用的是最基础的堆放方式，从上到下依次排开，例如，我们添加四个label，效果如下：

![](http://static.oschina.net/uploads/space/2015/1014/140603_GZgH_2340880.png)

通过改变label的添加顺序，可以改变其上下位置：

![](http://static.oschina.net/uploads/space/2015/1014/140712_inlI_2340880.png)

这种方式的布局高度并没有限制，我们可以一直往下排列，在watch上，会出现滑动的效果：

![](http://static.oschina.net/uploads/space/2015/1014/140944_AZT7_2340880.png)    ![](http://static.oschina.net/uploads/space/2015/1014/144748_Ewoz_2340880.png)

### 三、使用Group进行复杂的界面布局

        通过上面的布局方式，我们只能进行纵向的排列布局，这并不能达到我们的需求，WatchKit中提供那一套布局的模型：Group。

        可以这样理解，group就是将屏幕分成了几各分区，我们可以设置各个分区的排列方式，例如水平或者垂直，通过这样的思路，完成复杂的watch界面布局，例如下面的效果：

![](http://static.oschina.net/uploads/space/2015/1014/150437_pwgQ_2340880.png)

这样效果的一个界面，就是将在屏幕中添加了三个Group，最上面的Gorup设置为水平排列模式，在其中添加了两个按钮和一个分割线，中间一个Group是垂直排列模式，放入了一个选择器和一个按钮，最下面一个Group也是水平排列模式，放入了一个按钮和一个时间栏。

#### 扩展：所谓Group

        Group在界面布局上，不仅可以起到分区屏幕的作用，其还可以设置一些属性来使布局更加漂亮。在storyBoard右侧的设置菜单中，我们可以对这些属性进行操作：

Layout：设置布局模式，分为水平布局和垂直布局两种

insert：可以设置内容区域偏移量，通过这个属性，我们可以使其中填充的控件四周留白

Spacing：其中填充的控件的间距

BackGround：设置Group的背景图案

Mode：设置背景图案的填充方式

Animate：出现时带动画

color：设置Group的背景颜色

Radius：设置Group的圆角度

### 四、布局中控件的位置和尺寸设置

        在iphone中，我们使用frame或者约束来控制控件的位置和尺寸，在watch中则简单很多，尺寸和位置都是固定的模式，我们只需要做一些设置即可。

#### 1、控件尺寸的控制

        对于控件的尺寸，有三种模式，控件的width和Height都是通过这三个模式设置的：

Relative to Container：自身的尺寸是按照容器的尺寸比例设置的。例如设置为0.5的话，当前控件的尺寸就是容纳其Group的一半。

Size To Fit Content:自身的尺寸与自身内容相关，例如，label中字数的多少决定了label的尺寸。

Fixed：手动设置一个固定的值。

#### 2、控件位置的控制

        因为watch的界面十分简洁，对于控件的位置设置，是通过水平和垂直两个维度来设置的，通过设置每个维度的属性来控制其在容纳它的Group中的位置：

Horizontal：left（左），center（中心），right（右）

Vertical：top（上），center（中心），bottom（下）

### 一点注意:

        关于图片素材，你可以发现，在Extension和App文件夹中各有一个Assets.xcasssets组，只有将素材放入APP文件夹下的这个组watch才能使用。

![](http://static.oschina.net/uploads/space/2015/1014/155020_hTLg_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

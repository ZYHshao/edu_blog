---
title: AppleWatch开发入门六——Glance(预览)扩展的应用
date: 2015-10-16
categories: Apple Watch开发手记
tags: []
---
## AppleWatch开发入门六——Glance(预览)扩展的应用

### 一、简介

        Glance是watchOS中类似iOS的today插件一样的预览扩展。提供了Glance功能的WatchApp可以在手表主页上唤起Glance，展示app相关信息，然而这个扩展只能作为展示作用，并不能进行太多的交互，界面的布局也有很大的限制，因此，Glance的应用主要在于展示备忘信息等。特点如下：

1、扩展的样式布局我们并不能完全个性化，只能通过系统模板来布局。

2、扩展中不能添加交互功能，只能展示信息，点击界面间唤起WatchApp。

3、一个app只能享有一个Glance界面，并且是单屏的不可滑动。

### 二、创建一个Glance

        在我们创建WatchApp的时候，可以勾选创建Glance：

![](http://static.oschina.net/uploads/space/2015/1016/104938_Tiqf_2340880.png)

同样，如果这里没有勾选，我们也可以在storyBoard中拉入一个Glance界面：

![](http://static.oschina.net/uploads/space/2015/1016/105024_4Yau_2340880.png)

可以发现，这里面的布局样式，我们不能做修改，只能使用系统提供的一些模板：

![](http://static.oschina.net/uploads/space/2015/1016/105636_MvA0_2340880.png)

我们创建一个模板，可以将其中元素与文件关联，进行代码的动态设置。

在Xcode7中，在Scream中选择Glance项目，进行运行：

![](http://static.oschina.net/uploads/space/2015/1016/110128_GMIL_2340880.png)

模拟器效果如下：

![](http://static.oschina.net/uploads/space/2015/1016/111131_56ws_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

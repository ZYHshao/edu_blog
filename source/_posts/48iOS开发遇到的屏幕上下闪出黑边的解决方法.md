---
title: iOS开发遇到的屏幕上下闪出黑边的解决方法
date: 2015-05-07
categories: iOS逻辑初窥
tags: []
---
## iOS开发遇到的屏幕上下闪出黑边的解决方法

在IOS开发时，使用的时IOS的模拟器，程序中任何有关坐标的地方也是根据屏幕获取的，而在IOS7的系统上运行，却发现屏幕小了一截，上下各闪出一块黑色区域。后经过查找原因，解决方法如下:

项目的App Icon and Launch Images设置中，本来是这样的：

![](http://static.oschina.net/uploads/space/2015/0507/114106_K6Ia_2340880.png)

点击Use Asset Catalog，之后点击Migrate，设置界面如下图模样：

![](http://static.oschina.net/uploads/space/2015/0507/114352_YBhO_2340880.png)

这时在IOS7上就能充满屏幕了。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

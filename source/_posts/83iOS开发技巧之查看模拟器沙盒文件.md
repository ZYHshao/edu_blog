---
title: iOS开发技巧之查看模拟器沙盒文件
date: 2015-07-19
categories: iOS逻辑初窥
tags: []
---
## iOS开发技巧之查看模拟器沙盒文件

iOS开发中，在对数据库进行操作时，有时我们需要直观的查看数据库的内容，那么我们如何找到沙盒中的这个文件呢，步骤很简单：

1.点击Finder选项栏上的前往菜单：

![](http://static.oschina.net/uploads/space/2015/0719/200139_cOsO_2340880.png)

2.选择前往文件夹选项：

![](http://static.oschina.net/uploads/space/2015/0719/200257_iDll_2340880.png)

前往的文件路径为：/Users/username/Library/Application Support/iPhone Simulator/

其中username为当前mac电脑的用户名。

3.界面类似如下模样，选择一个版本的模拟器，应用的沙盒文件就在Applications中。

![](http://static.oschina.net/uploads/space/2015/0719/200542_67db_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

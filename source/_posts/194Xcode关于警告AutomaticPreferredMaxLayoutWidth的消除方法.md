---
title: Xcode关于警告AutomaticPreferredMaxLayoutWidth的消除方法
date: 2016-04-12
categories: 日常技巧
tags: []
---
## Xcode关于警告AutomaticPreferredMaxLayoutWidth的消除方法

     在iOS开发中，如果使用到了storyboard与xib文件并且使用autolayout进行自动布局，有时会报出Automatic Preferred Max Layout Width before iOS8.0的警告。工程中如果兼容的iOS版本为iOS8.0一下，并且使用了多行UILabel控件，往往在autolayout自动布局时会出现上述警告，上述警告的主要原因是在iOS8.0后系统会自动计算多行UILabel控件的理想换行宽度，iOS8以下则不会，需要开发者手动设置一个确定的值。

    解决方案如下，找到xib或storyboard中的多行UILabel控件，勾选Explicit属性，设置为一个固定的值，例如0。如下图所示：

![](http://static.oschina.net/uploads/space/2016/0412/151709_shq0_2340880.png)

之后上述警告即可消除，事实上，使用了autolayout后，这个属性并没有任何效果，仅仅为了消除警告，直接设置为0即可。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

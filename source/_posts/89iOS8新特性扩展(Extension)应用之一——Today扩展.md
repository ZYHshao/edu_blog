---
title: iOS8新特性扩展(Extension)应用之一——Today扩展
date: 2015-07-30
categories: iOS逻辑初窥
tags: []
---
## iOS8新特性扩展(Extension)应用之一——Today扩展

### 一、理解扩展

#### 1、简介

        基于iOS系统的安全性考虑，其应用的数据存储是通过沙盒模式进行的，要实现应用之间的数据共享十分困难，功能共享就更加棘手。在iOS8系统中，apple为我们提供了一个革命性的功能：扩展。我们可以通过扩展来使app间数据甚至功能进行共享。

#### 2、几种扩展模式

##### （1）今日视图扩展:today

        这个扩展也被叫做 widget。该扩展可以将今日发生的简短消息放到消息中心的「今日」视图里。这个功能类似于安卓系统中的小控件，只是安卓的可以直接放在桌面上，更加自由。示例如下：

![](http://static.oschina.net/uploads/space/2015/0730/110439_3xEr_2340880.png)

##### （2）分享功能扩展

        该扩展允许应用向在线服务上传照片、链接或者其他文件。在以前版本中，我们若要实现分享功能，必须进行复杂的操作。

##### （3）个性操作

        通过这个功能，可以实现两个APP中共享一些内容，例如编辑文字中的图片，翻译网页中的文字。

##### （4）照片操作

        这个类型的扩展可以允许我们在ipone相机中拍摄的照片使用其他图片编辑软件进行编辑。

##### （5）文件分享

        该扩展可以让软件将文件保存在各种云存储服务商。

##### （6）自定义键盘

        允许用户使用第三方的键盘输入法。

### 二、ToDay扩展的创建

        扩展是一个独立的构成，和其有关的两个概念是宿主APP和主机APP，宿主APP是扩展存放的地方，与扩展可以实现资源共享，主机APP是扩展运行的程序，例如ToDay扩展有抽屉中的Today应用进行运行。要创建一个ToDay扩展，首先我们需要创建一个宿主APP：

新建一个工程：

![](http://static.oschina.net/uploads/space/2015/0730/112000_OjFF_2340880.png)

选择xcode工具栏中的File->new->target

![](http://static.oschina.net/uploads/space/2015/0730/112158_NBCa_2340880.png)

在Application Extension中有上面提到的6中扩展，我们选择Today。

这是我们的项目中会多了一个扩展的文件夹：

![](http://static.oschina.net/uploads/space/2015/0730/112405_XDFb_2340880.png)

这个文件夹中有一个ViewController，我们可以在里面进行布局，还有一个plist文件，可是配置扩展的一些属性。

我们创建一个按钮：

```
 UIButton * btn = [[UIButton alloc]initWithFrame:CGRectMake(0, 0, 100, 30)];
    [btn setTitle:@"231" forState:UIControlStateNormal];
    [self.view addSubview:btn];
```

之后我们运行这个扩展：

![](http://static.oschina.net/uploads/space/2015/0730/112628_X6SW_2340880.png)

xcode会让我们选择运行扩展的主机程序，因为这是一个today类型的扩展，我们选择Today：

![](http://static.oschina.net/uploads/space/2015/0730/112737_MJvy_2340880.png)

运行后，在系统的通知抽屉中，就会出现我们的这个扩展：

![](http://static.oschina.net/uploads/space/2015/0730/112842_TP6R_2340880.png)

同样，我们可以创建tableView，imageView以及其他复杂的视图效果，我们也可以编写很多逻辑功能，跳转APP等。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

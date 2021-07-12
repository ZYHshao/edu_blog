---
title: Android Studio2.x版本无法自动关联源码的解决方法
date: 2016-08-20
categories: Android小记
tags: []
---
## Android Studio2.x版本无法自动关联源码的解决方法

        在学习android开发过程中，对于一个不熟悉的类，阅读源码是一个很好的学习方式，使用andorid studio开发工具的SDK Manager管理工具可以十分方便的下载SDK源码，打开SDK Manager工具，下载源码界面如下图所示：

![](http://static.oschina.net/uploads/space/2016/0820/101700_jPjx_2340880.png)

在对应的SDK版本中可以下载SDK源码。

        安卓源码下载完成后，在类名上按住command键，点击鼠标左键会跳转进对应源码文件，如果是Windows系统，使用按住control键点击鼠标左键。如果android studio的版本为2.0以上，需要注意，尽管下载了源码文件，可以在跳转源码的时候，会报错误找不到源码 Sources for 'Android API 23 Platform' not found，并且会跳转类对应的class文件。如下图：

![](http://static.oschina.net/uploads/space/2016/0820/104706_KdRI_2340880.png)

我猜想出现这样的原因是android studio2.x工具的一个小bug，下载源码后，它没有自动对源码路径进行关联，我们可以手段添加源码路径来解决这个问题。

        1.检查andriod sdk源码是否下载成功：首先进入andorid sdk路径下的sources目录，如果其中有源码文件，说明andorid sdk的源码文件已经下载成功。在OS系统中，这个路径一般是：~/Library/Android/sdk/sources。

        2.在android studio偏好设置jdk.table.xml文件中添加源码路径，这个文件在android studio开发工具的配置目录中，路径如下：

在Windows系统中，一般为：系统盘:\\Users\\username\\.你的android studio名称及版本\\config\\options

在OS系统中，一般为：~/Library/Preferences/你的android studio名称及版本/options

打开jdk.table.xml文件后，找到对应SDK版本的源码路径配置标签，将第一步中检查的源码文件路径添加进入，如下图：

![](http://static.oschina.net/uploads/space/2016/0820/104248_Sodz_2340880.png)

        3.完全关闭android studio开发工具，重新启动，这次可以成功跳进源码了，Have fun。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：435043639

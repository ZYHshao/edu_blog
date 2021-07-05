---
title: Xcode真机测试could not find developer disk image解决方法
date: 2015-10-25
categories: 日常技巧
tags: []
---
## Xcode真机测试could not find developer disk image解决方法

        在使用Xcode进行真机调试的时候，有时根据真机的系统不同，会出现could not find developer disk image 错误，这是由于真机系统过高或者过低，Xcode中没有匹配的配置包文件，我们可以通过这个路径进入配置包的存放目录：

/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport

里面有类似这样的一些文件夹，如果这些文件夹中没有包含我们真机的系统，则不能进行真机测试。但是我们可以通过将相应的配置包添加入这个文件夹来解决问题：

![](http://static.oschina.net/uploads/space/2015/1025/095617_WYzU_2340880.png)

说了解决的方法，不提供文件，会让大家觉得坑爹，下面给大家一个链接，里面有从iOS4.2到9.1所有版本的配置包，大家各取所需，不用感谢我：

http://pan.baidu.com/s/1qYIQWjE。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

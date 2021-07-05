---
title: iOS9系列专题四——APP传输安全
date: 2015-09-28
categories: iOS9专题
tags: []
---
## APP Transport Security——加密传输方式

        随着ios9的推出和Xcode的升级，apple将默认开发者使用https的传输方式，相比http的传输协议，这无疑会增加一些安全性，对于开发者而言，一下子将http协议全部升级为https协议，不是一件容易的事，我们可以通过Xcode的一些配置，使其支持http的传输协议。

        如果在Xcode7上运行http协议的应用，会出现如下信息：

![](http://static.oschina.net/uploads/space/2015/0928/093358_oGGu_2340880.png)

这个信息也很清晰，需要我们在info.plist文件中配置一些参数来支持http。

首先，在项目的Info.plist中加入NSAppTransportSecurity这个键，类型为Dictionary,在字典中添加一对键值，键为Boolen类型的NSAllowsArbitraryLoads，值为YES，如下：

![](http://static.oschina.net/uploads/space/2015/0928/093946_herm_2340880.png)

这时再运行项目，就可以正常取到数据了。

几点注意：

1.总有朋友说plist文件中配置了依然没有效果，一开始我很奇怪，后来发现了原因，info.plist文件有两个，一个是正式项目中的，一个是测试项目中的，一定要配置在正式项目中。

2.可能Xcode的还有些缺陷，这些键值不能通过自动补全提示出来，需要我们无误的手打。

后续：Xcode7.1中已经支持自动补全的功能。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

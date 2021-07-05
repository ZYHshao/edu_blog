---
title: iOS生成标准的注释工具——VVDocumenter
date: 2015-09-16
categories: 小码工具
tags: []
---
## iOS生成标准的注释工具——VVDocumenter

        在程序开发中，我们免不了要写许多注释，方便帮别人也方便我们自己以后检查我们的代码。然而，写注释是一件十分浪费我们时间与精力的事，要写符合文档格式的注释，更是会消耗我们很多的功夫，幸运的是，VVDocumenter可以帮我们很大的忙。

        gitHub地址：[https://github.com/onevcat/VVDocumenter-Xcode](https://github.com/onevcat/VVDocumenter-Xcode)。

安装与使用方法：下载github源码，使用xcode打开工程，运行一下，如果成功，插件就安装好了，这时，我们必须将xcode重新启动一下，才可以使用。

重启xcode，在任意一个地方输入///，即会自动出现如下的注释模板，参数部分已经由占位符写好

```
/**
 *  <#Description#>
 *
 *  @param data <#data description#>
 */
- (void)updateWithData:(id)data;
```

是不是写注释变成了一件非常有趣的事，你还可以对其进行一些设置，在xcode->window菜单栏中，有VVDocumenter这个标签，里面可以对生成注释的模板进行一些设置，比如生成注释的快捷键，注释的对齐模式，注释显示创建者和时间等。例如如下设置就会生成这样的注释：

![](http://static.oschina.net/uploads/space/2015/0916/134312_neUM_2340880.png)

```
/**
 *  @author Elephant, 15-09-16 13:09:28
 *
 *  @brief  <#Description#>
 *
 *  @param data <#data description#>
 */
 - (void)updateWithData:(id)data;
```

最后，推荐这款小插件给你，祝你写代码愉快O(∩_∩)O。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

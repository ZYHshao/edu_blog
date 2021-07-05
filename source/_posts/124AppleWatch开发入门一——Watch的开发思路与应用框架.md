---
title: AppleWatch开发入门一——Watch的开发思路与应用框架
date: 2015-10-13
categories: Apple Watch开发手记
tags: []
---
## AppleWatch开发入门一——Watch的开发思路与应用框架

### 一、引言

        Apple Watch无疑是apple在智能手表领域的一次革命，如何在Watch上开发出实用且具有美感的应用，是iOS开发者们开始思考的一个问题，由于watch的随身性和快捷性，在某些方面，它有比iphone更加大的优势，要抓住watch的这些特点，开发出淋漓尽致的应用，就需要改变一些在iphone开发的思路，正如一句话：只有忘掉经验，才会有意想不到的突破。

        这一系列博客，首先是总结我在公司watch项目开发中的一些经验，其次，还会加入一些我的理解和想法，如有错误，欢迎指正，如果对你有帮助，也欢迎撒花，😄。

### 二、开发前我们需要准备什么

        如果你没有开发过iphone，直接来做watch，我建议你不要这么做，不是不可以，而是目前所有的第三方应用都必须基于iphone的扩展，原生的watch应用，苹果目前还没有开放给开发者，因此实际上，我们在watch上可以做的事情十分有限，或许后续apple会开放更多接口，但是目前，我们必须放弃iphone开发的思路，从新开始。

#### 1、watch应用的架构

        如上所说，完全脱离iphone的原生watch应用，我们目前还不能开发，所有第三方的watch应用必须基于一个iphone的host app。我们可以通过创建一个watch应用来观察一下，首先，在Xcode6.3后虽然支持watch的开发，但watch模拟器并不十分好用，Xcode7进行了优化，通过模拟器，基本可以完成我们的开发。用Xcode新建一个项目，之后我们在Xcode菜单中创建一个target：

![](http://static.oschina.net/uploads/space/2015/1013/183409_QkG3_2340880.png)

选择apple Watch中的项目：

![](http://static.oschina.net/uploads/space/2015/1013/183625_Aoa0_2340880.png)

在如下的设置中，我们先将include Notification和Include Glance都勾选上，他们也是watch应用的一种表现方式，后面我们再说：

![](http://static.oschina.net/uploads/space/2015/1013/183652_Q271_2340880.png)

之后可以看到，我们的项目中会多了这样的几个文件夹：

![](http://static.oschina.net/uploads/space/2015/1013/183900_C7mc_2340880.png)

我们只需要关注下结尾为Extension和App的这两个，从目录结构我们也可以看到，App文件夹中有Storyboard这个文件，Extension文件夹中主要是一些代码文件，这也正是我们需要了解的watch app的机制，实际运行与我们手表上的是App文件夹中的界面，而逻辑的代码实际上是运行在我们的手机中的，作为iphone App的扩展而存在，通过手机与手表的交互，来达到watch上的一些操作。

        由此，我们可以理解，目前的第三方watch应用，watch类似于一个UI容器，通过与iphone的交互来达到一些逻辑和效果。

#### 2、三种watch应用方式的用途

        在我们创建watch的扩展时，我们勾选了两个Scene，从字面我们也可以理解的差不多，这里加上我的理解，不是官方的解释：

watch app：watch应用的主体，可以通过watch上的图标进入，可以与iphone进行交互与数据共享。

Notification：watch通知，会和iphone通知同步，包括本地的和远程的，这里和iphone不同在于有长通知和短通知的分别，在实际开发中，我们可以通过在后台添加参数来区分。在storyboard中的界面如下：

![](http://static.oschina.net/uploads/space/2015/1013/185620_Lmjj_2340880.png)           

Glance：预览界面，没有复杂的交互能力，也不能滑动，只能在单屏展示一些数据，点击后会进入主体watchApp中：

![](http://static.oschina.net/uploads/space/2015/1013/185745_JRFu_2340880.png)

#### 3、在模拟器上运行一个watch app

        选中我们的watch App工程，在Xcode7中运行如下：

![](http://static.oschina.net/uploads/space/2015/1013/190428_W7T7_2340880.png)

![](http://static.oschina.net/uploads/space/2015/1013/190440_oJqH_2340880.png)

如果你是以前版本的Xcode，可能需要在模拟器的Hardware中将其调出。

运行后，我们可以在watch模拟器上使用command+H来回到watch的主界面。

### 三、几点watch app的开发思路

1、优秀的watch app无疑必须是简单，朴素，快捷而时效的。

2、watch上不能自定义手势，我们可以使用的只有滑动，点击和长按

3、必须改变iphone布局的思想，完全接受新的watch布局特点，进行创新

4、iphone的特点是界面的绚丽，watch则是简约

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

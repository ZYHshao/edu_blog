---
title: mac端的优秀抓包工具——Charles使用
date: 2015-09-20
categories: 设计模式两三谈
tags: []
---
## mac端的优秀抓包工具——Charles使用

### 一、简介

        Charles是mac端的一款截取与分析网络请求的工具，在网络开发中使用其作分析，可以大大提高我们的开发效率。Charles是收费软件，一般可以试用三十天，但是可以通过相应的破解来获取服务（这里只做演示使用，希望大家购买正版软件）。Charles软件和破解包下载地址：[http://pan.baidu.com/s/1ySsUy](http://pan.baidu.com/s/1ySsUy)。

### 二、安装与使用

        下载好压缩包后，解压打开，将软件包拖入应用程序文件夹中，这时候一个原版的软件就可以让我们使用，只是有一个试用期，右键单击我们的Charles应用，显示包内容：

![](http://static.oschina.net/uploads/space/2015/0920/103424_CvAj_2340880.png)

将如下文件夹中的jar包替换为我们破解文件夹中的jar包：

![](http://static.oschina.net/uploads/space/2015/0920/103610_hP31_2340880.png)

### 三、使用Charles在mac上进行抓包分析

        在软件安装完成后，我们已经可以在mac上截取一般的网络请求了，打开软件，将Proxy设置中的Mac OS X Proxy勾选，设置为网络代理，这时候如果发生网络请求，就可以被Charles截获到

![](http://static.oschina.net/uploads/space/2015/0920/104102_FjE0_2340880.png)

如果我们需要截取SSL协议的网络请求，这时候我们还需要安装一个证书：[http://yun.baidu.com/s/1o6J2Crg](http://yun.baidu.com/s/1o6J2Crg)。注意将证书权限设置为始终信任。

抓获信息的界面如下：

![](http://static.oschina.net/uploads/space/2015/0920/104631_2Qse_2340880.png)

软件的功能十分强大，Structure是将请求按域名排序，Sequence是将请求时间排序，下面的Request和Response分别为请求的数据包和返回的数据包，如果是json数据，还会自动帮我们解析格式。

注意：如果iOS模拟器上抓不到请求包，重启模拟器即可。

### 四、在移动设备上进行抓包

        导入证书的过程和在mac上一样，在移动设备上访问[http://yun.baidu.com/s/1o6J2Crg](http://yun.baidu.com/s/1o6J2Crg)。进行证书下载，安装：

![](http://static.oschina.net/uploads/space/2015/0920/110439_MvK0_2340880.png)

在移动设备上截获网络请求，我们的移动设备必须和电脑在同一网段，在我们电脑的网络设置中查看IP地址，然后在移动设备上点击我们连接的电脑上的网络，在代理一栏中，选择手动，将我们刚才查看的ip地址填写在这里，并且设置一个端口号。

在Charles中的Proxy setting中如下勾选并配置端口号

![](http://static.oschina.net/uploads/space/2015/0920/111733_xvjF_2340880.png)

我们在设备上再访问网络，请求包就可以被我们抓取到。

### 五、Charles的更多应用

#### 1、过滤网络请求

有时候我们只想抓取某个主机的网络请求，我们可以设置过滤网络，在Proxy菜单中的Recording Setting中，我们选择include标签，可以在里面添加一个白名单，这样Charles就只截取在这个主机下的请求：

![](http://static.oschina.net/uploads/space/2015/0920/113301_ueZ3_2340880.png)

#### 2、模拟限速网络

很多时候，我们需要测试在网络不佳时应用请求的相关数据，我们可以模拟设置限速网络，在Proxy菜单中的Throttle Settings中将，Enable Throttling勾选，并可以在下面进行网路设置，only for selected host可以设置一个指定的主机访问进行限制网络。

![](http://static.oschina.net/uploads/space/2015/0920/113815_Nh5I_2340880.png)

#### 3、修改网络信息，多次请求

在测试接口时，有时候我们需要反复进行不同参数的接口请求，Charles也支持我们进行请求参数的修改和多次请求，在请求上点击右键，现则edit：

![](http://static.oschina.net/uploads/space/2015/0920/114818_HBz1_2340880.png)

其中的参数，请求类型等我们都可以修改，之后点击execute进行重新请求![](http://static.oschina.net/uploads/space/2015/0920/115000_XxUp_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

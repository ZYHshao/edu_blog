---
title: iOS网络编程之一——iOS网络框架简介
date: 2016-02-22
categories: iOS逻辑初窥
tags: []
---
## iOS网络编程之一——iOS网络框架简介

### 一、引言

        通过网络，一款应用才能够内容丰富，才能够完成用户操作与后台数据的交互。网络编程是移动应用或游戏开发开发中不可缺少的技术。iOS原生的网络框架也十分完善，其中涉及到的许多类和协议用于处理网络开发中的各种需求

### 二、URL加载框架

    iOS的URL加载系统包含许多类与协议，这些类和协议相互协作完成URL加载的信息配置，协议支持，身份验证，cookie和缓存等功能。APPLE开发文档中有如下图表示他们之间的关系：

![](http://static.oschina.net/uploads/space/2016/0222/102510_ve8P_2340880.png)

    关于URL加载系统，在iOS7之后，NSURLSession是首选的API框架，在iOS9中NSURLConnection相关的方法被弃用，如果需要兼容十分旧的版本，依然需要使用NSURLConnection。

### 三、一些辅助类

####         1.NSURLRequest

        NSURLRequest类负责一个具体的网络请求，其内部封装一个请求路径NSURL对象。如果需要对请求参数进行配置，可以使用NSMutableURLRequest。

####         2.NSURLResponse

        NSURLResponse类封装了相应数据，相应数据包括两部分，一部分是返回数据的状态码，数据长度、编码等信息，另一部分是内容数据本身。

####         3.NSURLCredential、NSURLProtectionSpace、NSURLCredentialStorage、NSURLAuthenticatioChallenge

        一些访问请求需要证书或者身份凭证进行验证，上面4个类对请求凭证进行相关设置。

####         4.NSURLCache

        在应用程序的开发中，为了减小对网络的依赖，提高程序性能，常常会对一些非实时性的数据进行缓存处理，NSURLCache类用于管理NSURLRequest请求缓存。

####         5.NSHTTPCookieStorage、NSHTTPCookie

        NSHTTPCookieStorage与NSHTTPCookie用于持久化的存储HTTP请求的Cookie数据。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

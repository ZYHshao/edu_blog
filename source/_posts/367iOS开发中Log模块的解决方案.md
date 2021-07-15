---
title: iOS开发中Log模块的解决方案
date: 2018-03-15
categories: 小码工具
tags: []
---
## iOS开发中Log模块的解决方案

    在软件开发中，调试模块，Log模块，可视化监控模块等都属于技术需求，并非业务需求，因此在进行这类模块的构建时，我们更多的应该以面向切面的思想来编程。例如Log模块，其往往只是在Debug模式下需要，在编写时就要注意让其可以自动适应编译环境而不需代码做切换操作。

    本篇博客主要介绍为项目添加Log模块的开发思路，并且推荐一款开源并且支持Cocoapods的Log库。

### 一、接口的提供

    面向切面编程的核心就是要足够简洁，不影响主体工程模块，不依赖也不引入任何其他模块的内容。Log引擎的接口设计可以全部采用宏的模式，使用预编译关键字可以十分容易的对Debug和Release环境进行分别处理，如下：

```objectivec
#ifndef YHDevLog
#define YHDevlOG



#ifdef DEBUG
#define START_DEBUG_MODE() [YHDevLogManager installDevLogView];
#define WARN_LOG(msg,...) [YHDevLogManager pushLog:0 format:msg,##__VA_ARGS__,nil];
#define ERROR_LOG(msg,...) [YHDevLogManager pushLog:1 format:msg,##__VA_ARGS__,nil];
#define LOG(msg,...) [YHDevLogManager pushLog:2 format:msg,##__VA_ARGS__,nil];
#else
#define START_DEBUG_MODE()
#define WARN_LOG(msg,...)
#define ERROR_LOG(msg,...)
#define LOG(msg,...)
#endif

#endif
```

其中，WARN\_LOG，ERROR\_LOG，LOG三个宏用来进行不同级别的Log打印，并且提供了格式化字符串的支持。START\_DEBUG\_MODE()宏用来开启模块，可以在应用程序启动完成后调用开启。

### 二、设计一个Model来描述Log信息

    Log信息是纯文本的，但是我们需要将其抽象成一种Model来进行描述，区分Log的级别，类型或者其他逻辑，YHDevLog中的Model设计如下：

```objectivec
@interface YHDevLogModel : NSObject

@property(nonatomic,strong)NSString * content;


/**
 0 warn 1 error 2 plain
 */
@property(nonatomic,assign)int type;
//是否展开详情
@property(nonatomic,assign)BOOL isOpen;

@end
```

### 三、Log窗口的设计

    关于Log窗口，我们可以采用悬浮window的方式，为了避免影响主应用功能，窗口的悬浮模式应该可以自由调整，窗口中可以使用TableView来展示Log信息，使用功能按钮来控制窗口尺寸和进行Log的分类和清空等。关于UI方面的代码，因为采用的是纯手写的Autolayout，这里就不在列举代码，有兴趣的可以再如下地址找到：

[https://github.com/ZYHshao/YHDevLog](https://github.com/ZYHshao/YHDevLog)

一些效果图：

> ![](https://static.oschina.net/uploads/space/2018/0315/140716_wl68_2340880.png)  ![](https://static.oschina.net/uploads/space/2018/0315/140756_KDOU_2340880.png)   ![](https://static.oschina.net/uploads/space/2018/0315/140817_UA8A_2340880.png)
> 
> ![](https://static.oschina.net/uploads/space/2018/0315/140832_XFrB_2340880.png)    ![](https://static.oschina.net/uploads/space/2018/0315/140849_6Raj_2340880.png)    ![](https://static.oschina.net/uploads/space/2018/0315/140902_tH7o_2340880.png)

使用下面的Pod可以直接使用此Log组件：

> pod  'YHDevLog' 

欢迎共同探讨，一起进步！

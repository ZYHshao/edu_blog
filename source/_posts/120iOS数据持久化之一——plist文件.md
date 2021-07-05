---
title: iOS数据持久化之一——plist文件
date: 2015-10-08
categories: iOS逻辑初窥
tags: []
---
## iOS数据持久化之一——plist文件

        iOS开发中，我们时常会将一些简单的数据进行持久化的存储，方便我们保存程序的一些配置和用户的一些数据，plist文件就是我们保存这些数据的最佳选择。

### 一、何为plist

        plist是一种文件格式，其内容规则是xml文件，后缀为.plist，因此，我们更习惯于成它问plist文件，在iOS开发中，这种文件常用来保存一些简单的配置数据，例如项目中的info.plist。

![](http://static.oschina.net/uploads/space/2015/1008/101222_ZCKH_2340880.png)

通过plist文件编辑器，我们可以很方便的查看和编辑层次清晰的plist文件。

### 二、通过操作plist文件进行数据持久化的几种方式

#### 1、操作系统为我们准备的用户配置文件——NSUserDefaults

        对于NSUserDefaults，具体用法和一些小技巧在以前的一篇博客中有详细的描述，一般的用户配置信息，我们都会选择通过这种方式来进行持久化，地址如下：[http://my.oschina.net/u/2340880/blog/411344](http://my.oschina.net/u/2340880/blog/411344)。

#### 2、在项目包中手动创建一个plist文件，通过代码对其进行操作

        这种方式创建的plist文件非常自由且直观，我们可以创建多个根据功能进行分类存储，并且可以通过Xcode的可视化工具进行可视化的修改。

首先，我们新创建一个文件，在Resource中选择 Property List文件：

![](http://static.oschina.net/uploads/space/2015/1008/103954_Chsr_2340880.png)

之后，我们通过Xcode，在其中添加一些数据：

![](http://static.oschina.net/uploads/space/2015/1008/111550_x6Ut_2340880.png)

通过代码，我们来获取这些数据：

```
 //获取myInfo文件地址
    NSString * path = [[NSBundle mainBundle]pathForResource:@"myInfo" ofType:@"plist"];
    NSMutableDictionary * dic =[NSMutableDictionary dictionaryWithContentsOfFile:path];
    NSLog(@"%@",dic);
```

打印结果如下：

![](http://static.oschina.net/uploads/space/2015/1008/111713_f5al_2340880.png)

这种方式添加的plist文件，我们只能在xcode中配置好，然后再程序中读取使用，但是不能在程序中修改这些数据，可以应用于一些固定的数据的存储，例如地图shu xing列表等。

#### 3、在沙盒目录中创建和使用plist文件

        我们还可以通过代码在沙盒中创建我们自己的plist文件，进行数据的存储。同时可以支持add，delete，replace，find等操作。

```
    //获取沙盒目录
    NSArray *paths=NSSearchPathForDirectoriesInDomains(NSDocumentDirectory,NSUserDomainMask,YES);
    NSString *plistPath1 = [paths objectAtIndex:0];
    
    //得到完整的文件名
    NSString *filename=[plistPath1 stringByAppendingPathComponent:@"my.plist"];
    NSDictionary * dic = @{@"my":@"haha"};
    [dic  writeToFile:filename atomically:YES];
    
    //取数据
    NSDictionary * getDic = [NSDictionary dictionaryWithContentsOfFile:filename];
    NSLog(@"%@",getDic);
```

打印如下：

![](http://static.oschina.net/uploads/space/2015/1008/113352_9wsE_2340880.png)

这种方式无疑会更加安全，存取也更加自由。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

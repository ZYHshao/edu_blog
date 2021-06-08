---
title: iOS 封装静态库(.a文件)
date: 2015-04-11   
categories: iOS编程技巧
tags: [静态库,iOS编程]              
---
iOS中导入外部文件，一种是将源码导入，一种是导入静态库，有很多第三方库都是以静态库的形式提供给我们使用的，如何制作一个静态库呢？

一、xCode创建文件时，选择创建静态库文件：cacoaTouchStaticLibrary

![](http://static.oschina.net/uploads/space/2015/0411/112616_9CTZ_2340880.png)

创建完成后，我们在里面写我们的方法和实现：

.h文件和.m文件

```
#import <Foundation/Foundation.h>
@interface MyStaticLibrary : NSObject
-(void)myLog;
@end
```

```
#import "MyStaticLibrary.h"
@implementation MyStaticLibrary
-(void)myLog{
    NSLog(@"myLog");
}
@end
```

二、生成静态库文件：  
这里需要将设备选成IOS Device  
![](http://static.oschina.net/uploads/space/2015/0411/113223_GHhR_2340880.png)  
然后 使用command+B进行编译，如果xcode报出这样的一个错误：  
![](http://static.oschina.net/uploads/space/2015/0411/113459_c2G7_2340880.png)  
我们需要在Peoject->Code Signing ->Code Signing Identity 改成IOS Developer  
![](http://static.oschina.net/uploads/space/2015/0411/113736_o9nD_2340880.png)  
再次编译，成功。然后你会看到，Products中的.a文件由红色编程了黑色。我们右键show in finder，就可以看到编译成功的静态库文件了。

三、合并静态库

在文件夹中，我们看到有两个.a文件，分别用在模拟器调试和真机调试中，如果我们在开发时需要真机模拟器不停的切换，我们可以将这两个静态库文件合并成为一个：

在终端使用：lipo -create  -output 命令：

![](http://static.oschina.net/uploads/space/2015/0411/114943_3feF_2340880.png)

这时，我们的静态库文件就做好了。

三、静态库文件的使用：

将.a和.h文件导入工程，在需要的文件中导入头文件，即可使用。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

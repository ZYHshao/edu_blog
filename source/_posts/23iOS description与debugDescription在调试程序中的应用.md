---
title: iOS description与debugDescription在调试程序中的应用
date: 2015-04-17
categories: iOS逻辑初窥
tags: [iOS编程]              
---
## iOS 中打印函数description与debugDescription的应用

### 一、description和debugDescription是什么

        description和debugDescription是NSObject协议中的声明的两个方法，同时NSObject类也实现了这个方法，如果子类没有重写，则会调用父类的description和debugDescription方法。首先，这两个方法适用于程序代码的调试的，当我们调用打印Log时，会向对象发送一个这样的消息。

我们先来看声明部分的代码：

```
+ (NSString *)description;
+ (NSString *)debugDescription;
```

这里返回的字符串就是我们打印在控制台显示的信息。

### 二、NSObject基类中的description方法是如何实现的

我们写如下的测试代码：

```
 NSObject * objc = [[NSObject alloc]init];
 NSLog(@"objc:%@",objc);
```

控制台输出的信息如下：

![](http://static.oschina.net/uploads/space/2015/0417/114852_NnBl_2340880.png)

可以看到，方法的实现大致是这样的：

```
-(NSString *)description{
    return [NSString stringWithFormat:@"<%@:%p>",[self class],&self];
}
```

### 三、重写description方法

通过上面的介绍，我们大致知道description方法的原理了，在程序调试时，我们可以充分利用这个方法带来的便利，大大缩减我们调试程序所需要的时间。例如：创建一个Test类，给它定义两个属性如下：

Text.h

```
#import <Foundation/Foundation.h>
@interface TestObject : NSObject
@property(nonatomic,strong)NSString  * name;
@property(nonatomic,strong)NSString  * age;
@end
```

我们在.m文件中将description方法重写：

```
#import "TestObject.h"
@implementation TestObject
-(NSString *)description{
    return [NSString stringWithFormat:@"%@",@{@"name":_name,@"age":_age}];
}
@end
```

重写的方法将Test类对象的属性打印了出来，这时我们在调用NSLog函数时，打印结果如下：

![](http://static.oschina.net/uploads/space/2015/0417/115838_4GP6_2340880.png)

是不是很炫酷，如此一来，我们可以将我们基本不会用到的类名和地址转换成打印数据，极大的方便了我们代码的调试工作。

### 四、description与debugDescription的区别

这两个方法的区别仅仅在于调试的位置不同，调用不同的函数。description是我们在程序中打Log会调用的方法，debugDescription则是我们在断点调试时，在控制台使用po命令打印会调用的方法，比如我们重写Test类的这个方法：

```
-(NSString *)debugDescription{
    return [NSString stringWithFormat:@"<%@:%p>:%@",[self class],&self,@{@"name":_name,@"age":_age}];
}
```

然后我们在程序中加个断点运行，在程序断掉之后，我们在调试区输入：po text，回车之后，会出现如下的信息：

![](http://static.oschina.net/uploads/space/2015/0417/134542_LobI_2340880.png)

学习使用 欢迎转载

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592
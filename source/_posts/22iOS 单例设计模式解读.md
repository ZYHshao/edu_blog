---
title: iOS 单例设计模式解读
date: 2015-04-17
categories: 设计模式
tags: [iOS编程]              
---
## iOS 中单例设计模式的解读与用法

### 一、单例的作用

      顾名思义，单例，即是在整个项目中，这个类的对象只能被初始化一次。它的这种特性，可以广泛应用于某些需要全局共享的资源中，比如管理类，引擎类，也可以通过单例来实现传值。UIApplication、NSUserDefaults等都是IOS中的系统单例。

### 二、单例的写法

       单例的写法常用的有两种方式：

####        方式1、不考虑线程

```
static SingleCase *manager = nil;  
   
+ (SingleCase *)defaultManager {  
    if (!manager){ 
        SingleCase = [[self alloc] init];  
        return manager; 
        }
}
```

####           方式2、考虑线程安全

```
+ (SingleCase *)sharedManager  
{  
        static SingleCase *ManagerInstance = nil;  
        static dispatch_once_t predicate;  
        dispatch_once(&predicate, ^{  
                ManagerInstance = [[self alloc] init];   
        });  
    return ManagerInstance;  
}
```

### 三、代码的优化

        通过上面的方法，我们已经可以使用类方法来得到这个单例，但很多时候，项目的工程量很大，还有可能会很多开发者同时参与一个项目的开发，为了安全与管理代码的方便，也为了给不是这个单例的创作者但会用到这个单例的开发人员一些提示，我们通常会重写一些方法：

首先我们自己实现一个alloc方法：

```
+(instancetype)myAlloc{
    return [super allocWithZone:nil];
}
```

将我们的单例实现方法略作修改：

```
+(ZYHPayManager *)sharedMamager{
    static ZYHPayManager * manager;
    if (manager==nil) {
        manager=[[ZYHPayManager myAlloc]init];
    }
    return manager;
}
```

将一些视图实例化对象的方法重写：

```
+(instancetype)alloc{
    NSAssert(0, @"这是一个单例对象，请使用+(ZYHPayManager *)sharedMamager方法");
    return nil;
}
+(instancetype)allocWithZone:(struct _NSZone *)zone{
    return [self alloc];
}
-(id)copy{
    NSLog(@"这是一个单例对象，copy将不起任何作用");
    return self;
}
+(instancetype)new{
    return  [self alloc];
}
```

注意：这里的alloc使用了断言，让任何视图通过alloc创建对象的程序段断在此处，给程序员提示。copy方法这里只是简单的返回了原对象，并未做任何处理，打印信息给程序员提示。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

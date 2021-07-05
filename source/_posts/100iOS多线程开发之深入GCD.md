---
title: iOS多线程开发之深入GCD
date: 2015-08-11
categories: iOS逻辑初窥
tags: []
---
## iOS多线程开发之深入GCD

### 一、前言

        在以前的一些系列博客中，对iOS中线程的管理做了总结，其中涵盖了GCD的相关基础知识：[http://my.oschina.net/u/2340880/blog/417746](http://my.oschina.net/u/2340880/blog/417746)。那里面将GCD的线程管理能力，列队组能力，通过信号和消息控制程序流程的能力都有介绍，这里，我们继续深入GCD的功能，通过GCD来处理一些逻辑更加复杂的代码功能。

### 二、延时追加任务

        当我们在程序中处理延时任务的时候，我们一般会通过两种方式，一种是通过定时器进行延时执行，另外一种是通过如下的函数：

\- (void)performSelector:(SEL)aSelector withObject:(id)anArgument afterDelay:(NSTimeInterval)delay;  
  
然而，如果我们需要在多线程中进行延时操作，上面两种方式会显得十分麻烦，并且徒增代码的复杂度。GCD为我们提供了一种方式：

void  dispatch\_after(dispatch\_time\_t when, dispatch\_queue\_t queue,  dispatch\_block_t block);

这个方法有三个参数，第一个参数延时的时间，第二个参数为将任务加入的队列，第三个block为要执行的任务。示例如下：

```
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"1233");
    });
```

这里通过dispatch_time来创建时间对象，将打印信息的方法在3S后加入主线程队列。需要注意的是，这里只是将任务延时加入队列，并不是执行，如果是加入同步队列中，则会进入等待状态。

### 三、数据存取的线程安全问题

        在进行多线程编程时，或许总会遇到一类问题，数据的竞争与线程的安全。这些问题如果我们通过程序手动来控制难度将会非常大。GCD同样为我们简单的解决了这样的问题。

首先，如果只是在读取数据，而不对数据做任何修改时，我们并不需要处理安全问题，可以让多个任务同时进行读取，可是如果要对数据进行写的操作，那么在同一时间，我们就必须只能有一个任务在写，GCD中有一个方法帮我们完美的解决了这个问题，代码如下：

```
//创建一个队列
dispatch_queue_t queue = dispatch_queue_create("oneQueue", DISPATCH_QUEUE_CONCURRENT);
    //几个任务同时读操作
    dispatch_async(queue, ^{
        for (int i=0; i<5; i++) {
            NSLog(@"read1:%d",i);
        }
    });
    dispatch_async(queue, ^{
        for (int i=0; i<5; i++) {
            NSLog(@"read2:%d",i);
        }
    });
    //此处进行写操作
    /*
    下面这个函数在加入队列时不会执行，会等待已经开始的异步执行全部完成后再执行，并且在执行时，会阻塞其他任务
    执行完成后，其他任务重新进入异步执行
    */    
    dispatch_barrier_async(queue, ^{
        for (int i=0; i<5; i++) {
            NSLog(@"write:%d",i);
        }
    });
    //继续进行异步读操作
    dispatch_async(queue, ^{
        for (int i=0; i<5; i++) {
            NSLog(@"read3:%d",i);
        }
    });
    dispatch_async(queue, ^{
        for (int i=0; i<5; i++) {
            NSLog(@"read4:%d",i);
        }
    });
    dispatch_async(queue, ^{
        for (int i=0; i<5; i++) {
            NSLog(@"read5:%d",i);
        }
    });
```

打印信息如下：

![](http://static.oschina.net/uploads/space/2015/0811/113250_sIYo_2340880.png)

可以看出，读操作是异步进行的，写操作是等待后阻塞任务队列独立进行，结束后队列恢复异步执行读操作，这正是我们需要的效果。

### 四、GCD模式的单例

        通常情况下，我们的单例会是如下的样子：

```
+(instancetype)shared{
    static Auto * obj;
    if (obj==nil) {
        obj = [[Auto alloc]init];
    }
    return obj;
}
```

这种通过读取静态变量的方式在大多数情况下是没问题的，可是并不能保证程序百分百的安全，因为在多线程的操作中，会有可能初始化多个对象，在GCD中，我们可以使用如下方式：

```
+(instancetype)shared{
    static Auto * obj;
    //dispatch_once_t对象可以只保证执行一次
    static dispatch_once_t once;
    dispatch_once(&once, ^{
        obj = [[Auto alloc]init];
    });
     return obj;
    
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

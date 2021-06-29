---
title: iOS多线程编程之一——NSThread线程管理
date: 2015-05-19
categories: iOS逻辑初窥
tags: []
---
## iOS多线程编程之一——NSThread线程管理

NSTread是iOS中进行多线程开发的一个类，其结构逻辑清晰，使用十分方便，但其封装度和性能不高，线程周期，加锁等需要手动处理。

### 一、NSThread类方法总结

获取当前线程

```
+ (NSThread *)currentThread;
```

这个方法通过开启一个新的线程执行选择器方法

```
+ (void)detachNewThreadSelector:(SEL)selector toTarget:(id)target withObject:(id)argument;
```

线程用法示例如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    [NSThread detachNewThreadSelector:@selector(log) toTarget:self withObject:nil];
    for (int i=0; i<100; i++) {
        NSLog(@"%@=%d",[NSThread currentThread],i);
    }
}
-(void)log{
    for (int i=0; i<100; i++) {
        NSLog(@"%@=%d",[NSThread currentThread],i);
    }
}
```

运行后的打印信息：

![](http://static.oschina.net/uploads/space/2015/0518/172925_pg0a_2340880.png)

可以清晰的看出来，新启的线程和主线程是异步的。

程序是否是多线程执行

```
+ (BOOL)isMultiThreaded;
```

线程字典，我们可以为特殊的线程设置键值对

```
@property (readonly, retain) NSMutableDictionary *threadDictionary;
```

线程在某个时间执行

```
+ (void)sleepUntilDate:(NSDate *)date;
```

线程在等待一个时间间隔后执行

```
+ (void)sleepForTimeInterval:(NSTimeInterval)ti;
```

结束线程

```
+ (void)exit;
```

设置线程的优先级，取值的范围为0-1，1的优先级最高

```
+ (double)threadPriority;
+ (BOOL)setThreadPriority:(double)p;
```

这个属性是iOS8之后的新特性，将优先级更人性化的封装了起来

```
@property NSQualityOfService qualityOfService;
```

NSQualityOfService的枚举如下：

```
typedef NS_ENUM(NSInteger, NSQualityOfService) {
    //刷新UI级别的线程
    NSQualityOfServiceUserInteractive = 0x21,
    //用户请求的无需精确的任务的线程，例如点击加载邮件
    NSQualityOfServiceUserInitiated = 0x19,
    //周期性的任务线程，例如定时刷新
    NSQualityOfServiceUtility = 0x11,
    //后台任务的线程
    NSQualityOfServiceBackground = 0x09,
    //优先级未知的线程，优先级介于UserInteractive和Utility之间
    NSQualityOfServiceDefault = -1
};
```

判断是否是主线程

```
+ (BOOL)isMainThread;
```

获取主线程

```
+ (NSThread *)mainThread;
```

### 二、属性与成员方法总结

初始化方法，选择器可以带一个参数

```
- (instancetype)initWithTarget:(id)target selector:(SEL)selector object:(id)argument;
```

线程是否正在执行

```
@property (readonly, getter=isExecuting) BOOL executing;
```

线程是否已经执行结束

```
@property (readonly, getter=isFinished) BOOL finished;
```

线程是否已经取消执行

```
@property (readonly, getter=isCancelled) BOOL cancelled;
```

### 三、隐式的通过NSThread进行多线程编程

NSObject的一个类别中提供了支持多线程的方法，如下：

这个函数指定在主线程执行一个选择器，arg是参数，wait是是否立即执行，如果YES，则会阻塞当前主线程的任务，NO则会等待当前任务结束后执行。

```
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait;
```

这个函数指定在某个线程执行选择器

```
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(id)arg waitUntilDone:(BOOL)wait;
```

指定在后台线程中执行选择器

```
- (void)performSelectorInBackground:(SEL)aSelector withObject:(id)arg;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

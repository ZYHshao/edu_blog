---
title: iOS中RunLoop机制浅探
date: 2015-08-13
categories: iOS逻辑初窥
tags: []
---
## iOS中RunLoop机制浅探

### 一、浅识RunLoop

        RunLoop这个家伙在iOS开发中，我们一直在用，却从未注意过他，甚至都不从见过他的面孔，那个这个神秘的家伙究竟是做什么的？首先，我们先来观察一下我们的程序运行机制。

        无论是面向对象的语言或是面向过程的语言，代码的执行终究是面向过程的。线程也一样，一个线程从开始代码执行，到结束代码销毁。就像HELLO WORLD程序，打印出字符串后程序就结束了，那么，我们的app是如何实现如下这样的机制的呢：app从运行开始一直处于待命状态，接收到类似点击事件等用户交互后执行相应操作，完成后继续等待交互响应，直到我们将程序杀死。通过这个过程的分析，我们可能会猜到，我们执行的主线程一定是在一个死循环中，没有任务的时候进行休眠，接收到任务后被激活执行任务。现在我们可以理解了，这样一个管理线程执行任务的机制就是RunLoop机制，线程在执行中的休眠与激活就是由RunLoop对象进行管理的。

### 二、RunLoop与线程的关系

        上面我们说到，RunLoop是用来管理线程的，那么他们直接有着怎样的关系，又是怎样进行交互的呢。事实上，每一个线程中都有一个Runloop对象，可以通过具体方法获得。这里有一点需要我们注意，官方文档上描述，虽然每一个线程中都可以获取RunLoop对象，但是并不是每一个线程中都有这个实例对象，我们可以这样理解：如果我们不获取runloop，这个runloop就不存在，我们获取时，如果不存在，就会去创建。在主线程中，这个MainRunLoop是默认创建并运行激活的。

### 三、认识NSRunLoop

        NSRunLoop是Cocoa框架中的类，与之对应，在Core Fundation中是CFRunLoopRef类。这两者的区别是前者不是线程安全的，而后者是线程安全的。我们这里只来讨论NSRunLoop的属性和方法：

\+ (NSRunLoop *)currentRunLoop;

获取当前线程的RunLoop：有则获取，无则创建

\+ (NSRunLoop *)mainRunLoop ;

获取主线程的RunLoop

[@property](http://my.oschina.net/property) (readonly, copy) NSString *currentMode;

获取当前runloop的执行模式，两种模式如下：

NSString * const NSDefaultRunLoopMode;

默认模式，接收大部分输入源的响应

NSString * const NSRunLoopCommonModes;

多种模式的集合

\- (CFRunLoopRef)getCFRunLoop;

获取RunLoop的CFRunLoopRef对象

\- (void)addTimer:(NSTimer *)timer forMode:(NSString *)mode;

将定时器添加到runloop中

\- (void)addPort:(NSPort *)aPort forMode:(NSString *)mode;

添加输入源端口到runloop中，NSPort对象可以理解为详细的载体，会传递消息与其代理。

\- (void)removePort:(NSPort *)aPort forMode:(NSString *)mode;

将某个输入源端口移除

\- (NSDate *)limitDateForMode:(NSString *)mode;

获取下个响应时间

解释：例如定时器的执行，其并不是按时间的间隔进行调用方法，而是在定时器注册到runloop中后，runloop会设置一个一个的时间点进行调用，比如10，20，30。如果错过了某个时间点，定时器并不会延时调用，而是直接等待下一个时间点调用，所以定时器并不是精准的。

\- (void)acceptInputForMode:(NSString *)mode beforeDate:(NSDate *)limitDate;

在某个时间期限前接收响应

\- (void)run;  
开始运行

\- (void)runUntilDate:(NSDate *)limitDate;

到某个时间点运行

\- (BOOL)runMode:(NSString *)mode beforeDate:(NSDate *)limitDate;

在某个期限前运行

### 四、RunLoop的应用

        正如前面所说，我们一直在使用他，却很少见到他。并且，我们在大多数情况下，都不需要显式的创建或者启动RunLoop，有两种情况，我们却必须手动设置它：

#### 1、在分线程中使用定时器

        定时器的实现便是基于runloop的，平时我们使用定时器你或许并没有对runloop做什么操作，那是因为主线程的runloop默认是开启运行的，如果我们在分线程中也需要重复执行某一动作，如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    queue = dispatch_queue_create("myQueue", DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(queue, ^{
        NSTimer * timer = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(time) userInfo:nil repeats:YES];
    });
    
}
-(void)time{
    NSLog(@"run");
}
```

你会发现，程序运行后并没有打印任何信息，方法并没有被调用，我们必须在线程中手动的执行如下代码：

```
   [[NSRunLoop currentRunLoop] run];
```

定时器才能正常工作。

#### 2、当你在线程中使用如下方法时

        某些延时函数和选择器在分线程中的使用，我们也必须手动开启runloop，这些方法如下：

[@interface](http://my.oschina.net/u/996807) NSObject (NSDelayedPerforming)  
  
\- (void)performSelector:(SEL)aSelector withObject:(id)anArgument afterDelay:(NSTimeInterval)delay inModes:(NSArray *)modes;

\- (void)performSelector:(SEL)aSelector withObject:(id)anArgument afterDelay:(NSTimeInterval)delay;

\+ (void)cancelPreviousPerformRequestsWithTarget:(id)aTarget selector:(SEL)aSelector object:(id)anArgument;

\+ (void)cancelPreviousPerformRequestsWithTarget:(id)aTarget;  
  
  
\- (void)performSelector:(SEL)aSelector target:(id)target argument:(id)arg order:(NSUInteger)order modes:(NSArray *)modes;

\- (void)cancelPerformSelector:(SEL)aSelector target:(id)target argument:(id)arg;  
\- (void)cancelPerformSelectorsWithTarget:(id)target;

### 五、补充

        RunLoop更强大的地方在于对消息的监听，因为CFRunLoopRef的线程安全优势，我们通常会更多使用后者。

        细心的你可能会发现，输入源被注册进Runloop中时会有方法进行remove，但是定时器却没有，但是定时器中的invalidate方法可以将其从runloop中移除，正如官方文档的说明：invalidate是重要也是唯一的可以将定时器从runloop的注销的方法，所以如果我们创建了定时器，就一定要在不使用时调用invalidate方法。我不知道apple为何将定时器的方法分离开来，可能的原因是让开发者更少的显式调用runloop的方法，你若是知道原因，恳请留言指导。

        关于定时器的问题，在另一篇博客中有介绍：[http://my.oschina.net/u/2340880/blog/398598](http://my.oschina.net/u/2340880/blog/398598)。

  
 

学习使用 欢迎转载

疏漏之处 欢迎指正

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

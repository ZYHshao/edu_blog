---
title: iOS NSTimer 定时器用法总结
date: 2015-04-10   
categories: iOS编程技巧
tags: [NSTimer,iOS编程]              
---
NSTimer在IOS开发中会经常用到，尤其是小型游戏，然而对于初学者时常会注意不到其中的内存释放问题，将其基本用法总结如下：

一、初始化方法：有五种初始化方法，分别是

\+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti invocation:(NSInvocation *)invocation repeats:(BOOL)yesOrNo;

```
- (void)viewDidLoad {
    [super viewDidLoad];
    //初始化一个Invocation对象
    NSInvocation * invo = [NSInvocation invocationWithMethodSignature:[[self class] instanceMethodSignatureForSelector:@selector(init)]];
    [invo setTarget:self];
    [invo setSelector:@selector(myLog)];
    NSTimer * timer = [NSTimer timerWithTimeInterval:1 invocation:invo repeats:YES];
    //加入主循环池中
    [[NSRunLoop mainRunLoop]addTimer:timer forMode:NSDefaultRunLoopMode];
    //开始循环
    [timer fire];
}
```

\+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti invocation:(NSInvocation *)invocation repeats:(BOOL)yesOrNo;

```
  NSTimer * timer = [NSTimer scheduledTimerWithTimeInterval:1 invocation:invo repeats:YES];
```

\+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(id)userInfo repeats:(BOOL)yesOrNo;

```
NSTimer * timer = [NSTimer timerWithTimeInterval:1 target:self selector:@selector(myLog) userInfo:nil repeats:NO]
```

\+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(id)userInfo repeats:(BOOL)yesOrNo;

```
NSTimer * timer = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(myLog:) userInfo:@"123" repeats:YES]
```

\- (instancetype)initWithFireDate:(NSDate *)date interval:(NSTimeInterval)ti target:(id)t selector:(SEL)s userInfo:(id)ui repeats:(BOOL)rep 

```
 NSTimer * timer = [[NSTimer alloc]initWithFireDate:[NSDate distantPast] interval:1 target:self selector:@selector(myLog:) userInfo:nil repeats:YES];
    [[NSRunLoop mainRunLoop]addTimer:timer forMode:NSDefaultRunLoopMode];
```

注意：这五种初始化方法的异同：

        1、参数repeats是指定是否循环执行，YES将循环，NO将只执行一次。

        2、timerWithTimeInterval这两个类方法创建出来的对象如果不用 addTimer: forMode方法手动加入主循环池中，将不会循环执行。并且如果不手动调用fair，则定时器不会启动。

        3、scheduledTimerWithTimeInterval这两个方法不需要手动调用fair，会自动执行，并且自动加入主循环池。

        4、init方法需要手动加入循环池，它会在设定的启动时间启动。

二、成员变量

[@property](http://my.oschina.net/property) (copy) NSDate *fireDate;

这是设置定时器的启动时间，常用来管理定时器的启动与停止

```
    //启动定时器
    timer.fireDate = [NSDate distantPast];
    //停止定时器
    timer.fireDate = [NSDate distantFuture];
```

[@property](http://my.oschina.net/property) (readonly) NSTimeInterval timeInterval;

这个是一个只读属性，获取定时器调用间隔时间。

[@property](http://my.oschina.net/property) NSTimeInterval tolerance;

这是7.0之后新增的一个属性，因为NSTimer并不完全精准，通过这个值设置误差范围。

[@property](http://my.oschina.net/property) (readonly, getter=isValid) BOOL valid;

获取定时器是否有效

[@property](http://my.oschina.net/property) (readonly, retain) id userInfo;

获取参数信息

三、关于内存释放

如果我们启动了一个定时器，在某个界面释放前，将这个定时器停止，甚至置为nil，都不能是这个界面释放，原因是系统的循环池中还保有这个对象。所以我们需要这样做：

```
-(void)dealloc{
    NSLog(@"dealloc:%@",[self class]);
}
- (void)viewDidLoad {
    [super viewDidLoad];
    timer= [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(myLog:) userInfo:nil repeats:YES];
    UIButton *btn = [[UIButton alloc]initWithFrame:CGRectMake(0, 0, 100, 100)];
    btn.backgroundColor=[UIColor redColor];
    [btn addTarget:self action:@selector(btn) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:btn];
}
-(void)btn{
    if (timer.isValid) {
        [timer invalidate];
    }
    timer=nil;
    [self dismissViewControllerAnimated:YES completion:nil];
}
```

在官方文档中我们可以看到 \[timer invalidate\]是唯一的方法将定时器从循环池中移除。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

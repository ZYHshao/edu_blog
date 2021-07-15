---
title: iOS中的CADisplayLink定时器
date: 2018-07-05
categories: iOS逻辑初窥
tags: []
---
## iOS中的CADisplayLink定时器

    说到定时器，在iOS中最常用的为NSTimer类，其实CADisplayLink类在某些场景下使用，要比NSTimer类更加适合。首先CADisplayLink也是一种定时器，并且其和屏幕的刷新率始终保持一致(很多时候会使用CADisplayLink来检测屏幕的帧率)。由于CADisplayLink的这种特性，使用它来实现流畅的动画效果非常合适。

    CADisplayLink类非常简单，解析如下：

```objectivec
//创建CADisplayLink对象 
/*
需要注意 定时器对象创建后 并不会马上执行 需要添加到runloop中
*/
+ (CADisplayLink *)displayLinkWithTarget:(id)target selector:(SEL)sel;
//将当前定时器对象加入一个RunLoop中
- (void)addToRunLoop:(NSRunLoop *)runloop forMode:(NSRunLoopMode)mode;
//将当前定时器对象从一个RunLoop中移除 如果这个Runloop是定时器所注册的最后一个  移除后定时器将被释放
- (void)removeFromRunLoop:(NSRunLoop *)runloop forMode:(NSRunLoopMode)mode;
//将定时器失效掉 调用这个函数后 会将定时器从所有注册的Runloop中移除
- (void)invalidate;
//当前时间戳
@property(readonly, nonatomic) CFTimeInterval timestamp;
//距离上次执行所间隔的时间
@property(readonly, nonatomic) CFTimeInterval duration;
//预计下次执行的时间戳
@property(readonly, nonatomic) CFTimeInterval targetTimestamp;
//设置是否暂停
@property(getter=isPaused, nonatomic) BOOL paused;
//设置预期的每秒执行帧数 例如设置为1 则以每秒一次的速率执行
@property(nonatomic) NSInteger preferredFramesPerSecond CA_AVAILABLE_IOS_STARTING(10.0, 10.0, 3.0);
//同上 
@property(nonatomic) NSInteger frameInterval
  CA_AVAILABLE_BUT_DEPRECATED_IOS (3.1, 10.0, 9.0, 10.0, 2.0, 3.0, "use preferredFramesPerSecond");
```

我的博客即将搬运同步至腾讯云+社区，邀请大家一同入驻：https://cloud.tencent.com/developer/support-plan?invite_code=29qwh7m53g4kc

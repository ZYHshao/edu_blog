---
title: iOS中通知中心(NSNotificationCenter)的使用总结
date: 2015-04-25
categories: iOS逻辑初窥
tags: []
---
## iOS中通知中心NSNotificationCenter应用总结

### 一、了解几个相关的类

#### 1、NSNotification

这个类可以理解为一个消息对象，其中有三个成员变量。

这个成员变量是这个消息对象的唯一标识，用于辨别消息对象。

[@property](http://my.oschina.net/property) (readonly, copy) NSString *name;

这个成员变量定义一个对象，可以理解为针对某一个对象的消息。

[@property](http://my.oschina.net/property) (readonly, retain) id object;

这个成员变量是一个字典，可以用其来进行传值。

[@property](http://my.oschina.net/property) (readonly, copy) NSDictionary *userInfo;

NSNotification的初始化方法：

\- (instancetype)initWithName:(NSString *)name object:(id)object userInfo:(NSDictionary *)userInfo;

\+ (instancetype)notificationWithName:(NSString *)aName object:(id)anObject;

\+ (instancetype)notificationWithName:(NSString *)aName object:(id)anObject userInfo:(NSDictionary *)aUserInfo;

注意:官方文档有明确的说明，不可以使用init进行初始化

#### 2、NSNotificationCenter

这个类是一个通知中心，使用单例设计，每个应用程序都会有一个默认的通知中心。用于调度通知的发送的接受。

添加一个观察者，可以为它指定一个方法，名字和对象。接受到通知时，执行方法。

\- (void)addObserver:(id)observer selector:(SEL)aSelector name:(NSString *)aName object:(id)anObject;

发送通知消息的方法

\- (void)postNotification:(NSNotification *)notification;

\- (void)postNotificationName:(NSString *)aName object:(id)anObject;

\- (void)postNotificationName:(NSString *)aName object:(id)anObject userInfo:(NSDictionary *)aUserInfo;

移除观察者的方法

\- (void)removeObserver:(id)observer;

\- (void)removeObserver:(id)observer name:(NSString *)aName object:(id)anObject;

几点注意：

1、如果发送的通知指定了object对象，那么观察者接收的通知设置的object对象与其一样，才会接收到通知，但是接收通知如果将这个参数设置为了nil，则会接收一切通知。

2、观察者的SEL函数指针可以有一个参数，参数就是发送的死奥西对象本身，可以通过这个参数取到消息对象的userInfo，实现传值。

### 二、通知的使用流程

首先，我们在需要接收通知的地方注册观察者，比如：

```
    //获取通知中心单例对象
    NSNotificationCenter * center = [NSNotificationCenter defaultCenter];
    //添加当前类对象为一个观察者，name和object设置为nil，表示接收一切通知
    [center addObserver:self selector:@selector(notice:) name:@"123" object:nil];
```

之后，在我们需要时发送通知消息

```
    //创建一个消息对象
    NSNotification * notice = [NSNotification notificationWithName:@"123" object:nil userInfo:@{@"1":@"123"}];
    //发送消息
       [[NSNotificationCenter defaultCenter]postNotification:notice];
```

我们可以在回调的函数中取到userInfo内容，如下：

```
-(void)notice:(id)sender{
    NSLog(@"%@",sender);
}
```

打印结果如下：

![](http://static.oschina.net/uploads/space/2015/0425/104111_TqQ7_2340880.png)

疏漏之处 欢迎指正

学习使用 欢迎转载

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

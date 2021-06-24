---
title: iOS中使用本地通知为你的APP添加提示用户功能
date: 2015-04-23
categories: iOS逻辑初窥
tags: []
---
## iOS中使用本地通知为你的APP添加提示用户功能

首先，我们先要明白一个概念，这里的本地通知是UILocalNotification类，和系统的NSNotificationCenter通知中心是完全不同的概念。

### 一、我们可以通过本地通知做什么

通知，实际上是由IOS系统管理的一个功能，比如某些后台应用做了某项活动需要我们处理、已经退出的应用在某个时间提醒我们唤起等等，如果注册了通知，系统都会在通知触发时给我们发送消息。由此，我们可以通过系统给我们的APP添加通知用户的功能，并且应用非常广泛。例如，闹种类应用，有按时签到相似功能的应用。下面，我们就来介绍如何注册并且设置一个本地通知。

### 二、了解UILocalNotification类

顾名思义，这个类就是我们需要使用的本地通知类，先来看它的几个属性：

设置系统发送通知的时间(如果是过去的时间或者0，则会立刻发起通知)

[@property](http://my.oschina.net/property)(nonatomic,copy) NSDate *fireDate;

设置时间的时区

[@property](http://my.oschina.net/property)(nonatomic,copy) NSTimeZone *timeZone;

设置周期性通知

[@property](http://my.oschina.net/property)(nonatomic) NSCalendarUnit repeatInterval;

NSCalendarUnit对象是枚举，设定通知的周期

```
typedef NS_OPTIONS(NSUInteger, NSCalendarUnit) {
        NSCalendarUnitEra                = kCFCalendarUnitEra,
        NSCalendarUnitYear               = kCFCalendarUnitYear,
        NSCalendarUnitMonth              = kCFCalendarUnitMonth,
        NSCalendarUnitDay                = kCFCalendarUnitDay,
        NSCalendarUnitHour               = kCFCalendarUnitHour,
        NSCalendarUnitMinute             = kCFCalendarUnitMinute,
        NSCalendarUnitSecond             = kCFCalendarUnitSecond,
        NSCalendarUnitWeekday            = kCFCalendarUnitWeekday,
        NSCalendarUnitWeekdayOrdinal     = kCFCalendarUnitWeekdayOrdinal,
        }
```

设置周期性通知参照的日历表

[@property](http://my.oschina.net/property)(nonatomic,copy) NSCalendar *repeatCalendar;

下面这两个函数是IOS8的新功能，在用户进去或者离开某一区域时发送通知

[@property](http://my.oschina.net/property)(nonatomic,copy) CLRegion *region;

设置区域检测通知是否重复(如果为YES，则没次进去出来都会发送，否则只发送一次)

@property(nonatomic,assign) BOOL regionTriggersOnce;

设置通知的主体内容

@property(nonatomic,copy) NSString *alertBody;

是否隐藏滑动启动按钮

@property(nonatomic) BOOL hasAction;

设置滑动打开的提示文字

@property(nonatomic,copy) NSString *alertAction;

设置点击通知后启动的启动图片

@property(nonatomic,copy) NSString *alertLaunchImage; 

下面这个方法是IOS8的新方法，是iwatch的接口，通知的短标题

@property(nonatomic,copy) NSString *alertTitle;

收到通知时，播放的系统音

@property(nonatomic,copy) NSString *soundName;

设置应用程序Icon头标数字

@property(nonatomic) NSInteger applicationIconBadgeNumber; 

用户字典，可用于传递通知消息参数

@property(nonatomic,copy) NSDictionary *userInfo;

注意：这个字符串是系统默认的提示音

NSString *const UILocalNotificationDefaultSoundName;

### 三、本地通知的设计流程  
首先，想让我们的APP实现本地通知功能，必须得到用户的授权，在Appdelegate中实现如下代码：

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    //如果已经得到授权，就直接添加本地通知，否则申请询问授权
    if ([[UIApplication sharedApplication]currentUserNotificationSettings].types!=UIUserNotificationTypeNone) {
        [self addLocalNotification];
    }else{
        [[UIApplication sharedApplication]registerUserNotificationSettings:[UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeAlert|UIUserNotificationTypeBadge|UIUserNotificationTypeSound  categories:nil]];
    }
    return YES;
}
```

当用户点击允许或者不允许后，会执行如下代理方法，我们把处理逻辑在其中实现

```
-(void)application:(UIApplication *)application didRegisterUserNotificationSettings:(UIUserNotificationSettings *)notificationSettings{
    if (notificationSettings.types!=UIUserNotificationTypeNone) {
        [self addLocalNotification];
    }
}
```

添加本地通知的方法：

```
-(void)addLocalNotification{
    //定义本地通知对象
    UILocalNotification *notification=[[UILocalNotification alloc]init];
    //设置调用时间
    notification.fireDate=[NSDate dateWithTimeIntervalSinceNow:0];//立即触发
    //设置通知属性
    notification.alertBody=@"HELLO，我是本地通知哦!"; //通知主体
    notification.applicationIconBadgeNumber=1;//应用程序图标右上角显示的消息数
    notification.alertAction=@"打开应用"; //待机界面的滑动动作提示 
    notification.soundName=UILocalNotificationDefaultSoundName;//收到通知时播放的声音，默认消息声音
    //调用通知
    [[UIApplication sharedApplication] scheduleLocalNotification:notification];
}
```

实现了上面三个步骤，本地通知的发出和接受基本都已完成，还有一些细节我们需要考虑：

应用进入前台后，将Icon上的头标清除：

```
-(void)applicationWillEnterForeground:(UIApplication *)application{
    [[UIApplication sharedApplication]setApplicationIconBadgeNumber:0];//进入前台取消应用消息图标
}
```

当不再需要这个通知时，清除它

```
 [[UIApplication sharedApplication] cancelAllLocalNotifications];
```

### 四、获取通知中的用户参数字典

在上面，我们提到了一个参数

@property(nonatomic,copy) NSDictionary *userInfo; 

我们可以在注册通知时将这个参数设置，然后在收到通知时使用get方法得到，但是这里有两种情况：

#### 1、如果我们的APP在前台或者后台进入前台时

-(void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification;

这个方法是APP在前台或者后台收到通知进入前台时调用的方法

#### 2、如果我们的APP在关闭状态

如果是这种情况，我们只能从下面函数的launchOptions中取到我们想要的参数

\- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions;

代码示例如下：

```
 //接收通知参数   
  UILocalNotification *notification=[launchOptions valueForKey:UIApplicationLaunchOptionsLocalNotificationKey];
    NSDictionary *userInfo= notification.userInfo;
```

疏漏之处 欢迎指正

学习使用 欢迎转载

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: iOS初级教程之三:Crash分析实践
date: 2021-03-23
categories: iOS逻辑初窥
tags: []
---
# \[iOS初级教程之三\]Crash分析实践

## 一、引言

    Crash分析与治理是移动端开发人员的必备技能，Crash相关数据也是衡量应用程序质量的重要指标。本篇文章，我们将讨论在iOS开发中基础的Crash治理实践经验，帮助初学者快速的掌握Crash治理技能，提升工作能力。文章将从如下几个方面进行介绍：

-   Crash的统计和分析
-   如何通过友盟APM平台做监控和报警
-   SDK收集工具的集成
-   各种类型的Crash分析实践

      Crash治理的重要一步是对Crash进行统计和分析，有了Crash的统计数据，我们才能具体的对某些Crash问题进行分析和处理，友盟U-APM平台提供了非常好的辅助工具，开发者的接入非常简单容易，其可以帮助开发者快速发现问题，统计问题，分析问题最终解决问题。

![](https://oscimg.oschina.net/oscnet/up-f184b7190b9b6741c20db1428375d60e93b.png)    ![](https://oscimg.oschina.net/oscnet/up-3b9497b601200b063e556e2180b44959169.png)

![](https://oscimg.oschina.net/oscnet/up-d8ab32847eaf6f3c2d0ae0982fe4cbdee9c.png)    ![](https://oscimg.oschina.net/oscnet/up-89c44e674146e66a59c52fb7b2b2766a1e4.png)

## 二、U-APM SDK的集成

      客户端应用集成U-APM SDK主要用来进行崩溃检测，卡顿检测以及场景记录等功能。如果使用CocoaPods工具其接入非常简单，在Podfile文件中添加如下依赖即可：

```ruby
pod 'UMCommon'
pod 'UMDevice'
pod 'UMAPM'
pod 'UMCCommonLog'

```

其中UMCommon是友盟SDK基础的支持库，提供SDK初始化等功能，UMDevice库与设备信息功能相关，UMAPM用来做性能与崩溃统计，UMCCommonLog是一个调试库，在开发时我们可以将其引用，用来查看上报情况。

      如果项目没有使用CocoaPods，也可以采用手动引入的方式来集成SDK。在如下地址可以根据需求下载到指定的SDK资源：

[https://developer.umeng.com/sdk/android?spm=a213m0.21038855.9168240680.3.6a311904uispVD](https://developer.umeng.com/sdk/android?spm=a213m0.21038855.9168240680.3.6a311904uispVD)

手动集成SDK还需要做一些简单的工程配置：

1.需要依赖如下系统库：

```
CoreTelephony.framework
libz.tbd
libsqlite.tbd
libc++.tbd
```

2.在工程的Targets->BuildSettings 中 ， Other Linker Flags增加-ObjC参数。

      完成了上面的配置过程，需要编写代码来完成U-APM SDK的接入工作，示例代码如下：

```objectivec
#import "AppDelegate.h"
#import <UMCommon/UMCommon.h>
#import <UMAPM/UMCrashConfigure.h>
#import <UMCommonLog/UMCommonLogHeaders.h>

@interface AppDelegate ()

@end

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // 1. 初始化SDK
    [UMConfigure initWithAppkey:@"602505af668f9e17b8aef059" channel:nil];
    // 2. 进行异常捕获
    [UMCrashConfigure setCrashCBBlock:^NSString * _Nullable{
        return @"发生了我们测试的Crash";
    }];
    // 3. 初始化Log
    [UMCommonLogManager setUpUMCommonLogManager];
    // 4. 开启Log
    [UMConfigure setLogEnabled:YES];
    return YES;
}

@end

```

由于需要选择一个较早的实际来进行SDK的初始化，因此我们通常会将初始化的相关代码放入didFinishLaunching方法中，也可以根据具体需求选择初始化的时机，接入SDK基本分为了4个步骤，上面示例代码中有详细的注释，在第1步初始化SDK中，传入的AppKey的值是在友盟后台创建应用后得到的。第2步调用的setCrashCBBlock用来设置上报Crash时的额外信息，通常在这个回调中我们可以将当前登录的用户信息等进行上报。

      我们可以手动写一些常见的会产生Crash的代码，在真机上运行上面示例代码后，可以在友盟的APM后台看到记录的异常信息，在上报的日志的自定义字段中，可以看到我们设置的额外上报数据，如下图所示：

![](https://oscimg.oschina.net/oscnet/up-bd5c0c34e5d25056129541e6843f28ffb83.png)

此时，你会发现我们收集到的很多堆栈信息都是未符号化的，即都是内存地址，并没与类与方法的信息，这是因为我们还需要配置下应用的符号表，使用Xcode在构建工程时，默认只会在生产环境生成符号表，我们也可以将Build Settings->Debug Information Format选项设置为DWARF with DSYM File来使其在Debug环境下也生成符号表，如下：

![](https://oscimg.oschina.net/oscnet/up-cfa699cff39c900e6542573409598346da0.png)

编译后生成的符号表会与App包放在同一文件下，我们需要在友盟U-APM后台的设置页面将此符号表文件进行上传，之后就可以正常的对堆栈信息进行解析。如下图：

![](https://oscimg.oschina.net/oscnet/up-0259d69a144215cf3e2ca0c339e4d93ac78.png)

## 三、分析用户路径与监控告警

      有时候我们记录到了线上的Crash，并且定位到了具体的页面，但是依然无法复现出相同的问题。很多情况下这是因为我们的复现路径与用户的操作路径并不一样，在友盟APM后台，对于收集到了异常问题，除了有详细的堆栈日志和自定义的上报数据外，还可以获取到用户的页面操作路径和设备信息，页面操作路径是非常重要的分析数据，根据这个路径我们可以大致还原出用户打开应用程序后的操作路径，方便我们对问题进行分析复现。如下图所示：

![](https://oscimg.oschina.net/oscnet/up-6c1159ca740e1beb27379ed38c8ffbf8433.png)

在设备信息页面可以对设备与操作系统相关信息进行查看，如下图：

![](https://oscimg.oschina.net/oscnet/up-6aee95632c09b9821823434ffb586437eb4.png)

      U-APM后台还提供了非常强大的监控与告警功能，我们可以设置一定的阈值作为报警条件。当某一刻异常问题触发了我们的报警规则，我们可以及时的收到反馈并及时的做出响应。在U-APM后台的检测报警功能页面，我们可以创建一种告警计划，如下图所示：

![](https://oscimg.oschina.net/oscnet/up-a6c8e6ae366b9be597c7a2922f87c524cb8.png)

在创建告警计划时，可以设置一些触发条件，例如在最近一小时内触发的错误数超过阈值，则进行告警。对于告警的方式，有钉钉机器人提醒，邮件，企业微信等，可以参照文档根据需要进行配置。

## 四、常见Crash分析实践

### 1.未实现的选择器

       未实现的选择器应该是开发中最常见的Crash原因之一，初学者在编写代码时，经常会在控制台看到如下类型的错误提示：

```
unrecognized selector sent to instance

```

这通常是因为调用了没有实现的方法或者执行方法的对象类型不对，我们将这类问题统称为未实现的选择器问题。产生这类的问题的场景通常有如下几种：

#### ①.声明方法未实现

      例如在.h文件中声明了一个方法，并在其他地方对此方法进行了调用，但是此方法并没有在.m文件中实现，此时编译工程是不会有问题的，在运行时如果调用到了此未实现的方法会产生崩溃。

#### ②.协议方法未实现

      这种场景与声明方法未实现类似，有时候，协议中定义的方法并不一定都是必须实现的，为了避免出现此类问题，我们可以在调用协议方法之前先进行安全判断，如下：

```objectivec
if ([self.delegate respondsToSelector:@selector(protocolMehtod)]) {
    [self.delegate protocolMehtod];
}
```

#### ③.copy修饰了可变属性

      在定义属性时，如果我们将一个可变的属性使用了copy进行修饰，则在赋值时会隐式的将其拷贝成不可变的类型，这时如果我们调用了可变属性的方法就会产生异常，例如：

```objectivec
@property (nonatomic, copy) NSMutableArray *mutableArray;
```

这种场景具有很好的隐秘性，无论是赋值还是方法的调用，Xcode的自动检查功能都不能提前将问题指出，也不会有警告产生。

#### ④.动态调用了未知方法

      Objective-C本身是一种动态的语言，有很多种方式可以动态的进行方法的调用，这类调用是不会做编译时检查的，方法名写错或对象类型不对都会产生异常，因此最好在动态方法调用前，都进行安全判断，例如：

```objectivec
if ([self respondsToSelector:@selector(unknow)]) {
    [self performSelector:@selector(unknow)];
}
```

#### ⑤.低版本使用了高版本的API

      当低版本系统使用了高版本才有的接口时，也会产生未实现的选择器异常，对于这种场景，Xcode会有警告提示，我们可以在调用方法前，先进行版本的判断，示例如下：

```objectivec
// iOS 13 之后API
if (@available(iOS 13.0, *)) {
    [self canPerformUnwindSegueAction:@selector(test) fromViewController:self sender:nil];
} else {
    // Fallback on earlier versions
}
```

### 2.KVC相关异常

      KVC(Key Value Coding)又称键值编码，其指在iOS开发中，可以允许开发者通过key名直接访问对象的属性或者给对象的属性赋值，而不需要调用明确的存取方法。这样就可以在运行时动态地访问和修改对象的属性。很多高级的iOS开发技巧都是基于KVC实现的。

     KVC的几个核心方法列举如下：

```objectivec
//核心方法：
- (nullable id)valueForKey:(NSString *)key;
- (void)setValue:(nullable id)value forKey:(NSString *)key;  
- (nullable id)valueForKeyPath:(NSString *)keyPath;   
- (void)setValue:(nullable id)value forKeyPath:(NSString *)keyPath;

//高级方法：
+ (BOOL)accessInstanceVariablesDirectly;
- (nullable id)valueForUndefinedKey:(NSString *)key;
- (void)setValue:(nullable id)value forUndefinedKey:(NSString *)key;
- (void)setNilValueForKey:(NSString *)key;
- (NSDictionary<NSString *, id> *)dictionaryWithValuesForKeys:(NSArray<NSString *> *)keys;
- (NSDictionary<NSString *, id> *)dictionaryWithValuesForKeys:(NSArray<NSString *> *)keys;
- (void)setValuesForKeysWithDictionary:(NSDictionary<NSString *, id> *)keyedValues;
```

KVC相关的Crash场景主要有两种：

#### ①. 所使用了值为nil的key

      当我们使用KVC的方式向对象的属性进行赋值时，要保证Key值不为nil，否则会产生异常，在使用时要做下Key值的判空，如下：

```objectivec
NSString *key = nil;
if (key) {
    [self setValue:@"value" forKey:key];
}
```

#### ②. 使用了对象中不存在的key值

      在调用setValue：forKey：方法时，即是传入的Key值不为nil，也有可能会产生异常，默认情况下，如果要操作的属性对象中并不存在，则也会产生Crash，我们可以实现KVC中的如下两个方法来做兼容：

```objectivec
// 为不存在的属性进行KVC赋值时会调用这个方法
- (void)setValue:(id)value forUndefinedKey:(NSString *)key {
    NSLog(@"setForUndefinedKey:%@, %@", key, value);
}

// 获取不存在的属性的值的时候会调用这个方法
- (id)valueForUndefinedKey:(NSString *)key {
    return nil;
}
```

### 3.野指针相关异常

      由于野指针问题产生的相关异常通常是比较难处理和定位的。野指针通常指所指向的对象已经被释放的指针，其所指向的内存地址存储的数据也被称为僵尸对象。我们可以通过开启Xcode的僵尸对象功能来在开发阶段提前进行预防。在Xcode的scheme编辑中，将Zombie Objects进行勾选即可。如下：

![](https://oscimg.oschina.net/oscnet/up-f6222f7860463546cfbc423d2aedbdcf90b.png)

野指针相关问题的异常场景主要有如下几种：

#### ①. 使用了未初始化的对象

#### ②. ARC下，使用了assign或unsafe_unretained修饰对象

如下：

```objectivec
@property (nonatomic, assign) UIView *subView;
```

这种场景下，对象释放后，ARC不会自动的帮我们做指针置空操作。

#### ③.runtime关联对象使用了不合适的修饰，如OBJC\_ASSOCIATION\_ASSIGN 

原因与场景2类似，对于对象属性的修饰要使用正确的修饰符。

### 4.KVO相关异常

      KVO全称Key Value Observing，是Apple提供的一套事件通知机制。其允许一个对象监听另外一个对象特定属性的变化，由于KVO的实现机制的原因，一般继承自NSObject的对象才能使用，并且其只对属性才会发生作用。

      KVO和NSNotificationCenter都是iOS中观察者模式的一种实现。区别在于，相对于被观察者和观察者之间的关系，KVO是一对一的，而不一对多的。KVO对被监听对象无侵入性，不需要修改其内部代码即可实现监听，KVO可以监听单个属性的变化，也可以监听集合对象的变化。

     在某些场景下如果不恰当的使用KVO，也会产生Crash，常见场景如下：

#### ①.被观察者是局部变量

#### ②.观察者是局部变量

#### ③.未实现监听方法

#### ④.重复移除监听对象

要避免上述问题，在使用KVO时要把握两个核心重点：

1\. 注意监听对象与被监听对象的生命周期

2\. addObserver和removeObserver要成对出现

### 5.集合对象操作相关Crash

      这类Crash主要指不当的操作数组或字典所产生的的。

#### ①.数组越界问题

#### ②.向数组中添加nil元素

#### ③.遍历数组过程中使用了错误的方式修改了数组

#### ④. 字典设置nil值

### 6.多线程操作相关Crash

      和野指针问题类似，多线程产生的异常往往也是比较难定位和解决的。这类异常通常并不好复现，我们在编写代码时要将尽量将逻辑梳理清楚。常见问题场景如下：

#### ①. group enter 与 group leave

      在使用GCD多多线程开发时，dispatch\_group\_t是很常用的一种进行任务依赖编程的方式，需要注意，在使用dispatch\_group\_t时，要确保group enter 与 group leave是成对调用的，否则极易出现死锁问题。

#### ②.子线程做UI操作

     在子线程中操作UI不仅会造成页面更新不及时，页面混乱等问题，也极易产生异常从而Crash，因此在做UI操作时，一定要保证是在主线程执行。

#### ③.多线程对对象进行释放

    在多个线程中对变量进行赋值操作会造成，会造成变量所引用的旧的对象的多线程释放问题，会出现偶现crash。因此，如果有多线程对外部变量进行赋值的操作，我们可以使用信号量进行加锁，保证变量的赋值是串行的，示例代码如下：

```objectivec
__block NSObject *obj;
dispatch_semaphore_t sem = dispatch_semaphore_create(1);
dispatch_async(dispatch_get_global_queue(0, 0), ^{
    while (YES) {
        dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER);
        obj = [NSObject new];
        dispatch_semaphore_signal(sem);
    }
});
while (YES) {
    dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER);
    obj = [NSObject new];
    dispatch_semaphore_signal(sem);
}
```

#### ④.多线程同时操作数组

      多线程同时对数组进行操作也是比较危险的行为，例如当一个线程在对数据进行遍历时，另一个线程改变了数组元素的个数，会由于索引错乱而产生意想不到的问题甚至Crash。在多线程中遍历数组时，可以将数组拷贝一份在进行操作。

### 7.watch dog异常

      为了防止一个应用占用过多的系统资源，Apple设计了一个名为“看门狗”( watchdog )的机制。在不同的场景下，“看门狗”会监测应用的性能。如果超出了该场景所规定的运行时间，“看门狗”就会强制终结这个应用的进程。开发者们在 crashlog 里面，会看到诸如 0x8badf00d 这样的错误代码。异常代码：“0x8badf00d ”(看上去非常像 bad food)。

    Watch Dog的Crash本身并不是代码错误，其是一种保护机制，当我们收集到的异常有发现这类问题时，就要着重考虑下应用的性能，同时检查是否会有死锁等异常逻辑的产生。

## 五、建议

1\. 重视每一个Crash处理

2\. 有监控，紧急问题可以及时响应

3\. 积累治理经验

4\. 代码规范，安全

5\. 逻辑设计尽量简单，多线程场景要清晰

---
title: iOS通过NSUserDefaults实现简单的应用间数据传递
date: 2015-05-06
categories: iOS逻辑初窥
tags: []
---
## iOS中NSUserDefaults详解

NSUserDefaults是用于保存应用程序设置，应用信息等轻量级数据的的一个类，其本质是将数据写为plist文件的形式保存在本地。在IOS中，系统为每一个应用程序都默认创建了一个NSUserDefaults对象。

### 一、常用方法总结

\+ (NSUserDefaults *)standardUserDefaults;

获取系统默认创建的应用程序设置表

\+ (void)resetStandardUserDefaults;

这个方法用于将默认的UserDefaults释放掉，并在下次使用时创建一个新的对象，需要注意的是，调用这个方法后，对原UserDefaults单例进行的KVO监听将失效。

\- (instancetype)initWithSuiteName:(NSString *)suitename;

这个方法创建一个新的域：根据名字可以创建一些不同的域，分别存储几套设置信息。

\- (id)objectForKey:(NSString *)defaultName;

\- (void)setObject:(id)value forKey:(NSString *)defaultName;

\- (void)removeObjectForKey:(NSString *)defaultName;

上面三个方法是对对象存储进行的操作，分别是存储，获取和删除。

\- (NSString *)stringForKey:(NSString *)defaultName;

获取字符串数据

\- (NSArray *)arrayForKey:(NSString *)defaultName;

获取数组数据

\- (NSDictionary *)dictionaryForKey:(NSString *)defaultName;

获取字典数据

\- (NSData *)dataForKey:(NSString *)defaultName;

获取data数据

\- (NSArray *)stringArrayForKey:(NSString *)defaultName;

获取字符串数组数据

\- (NSInteger)integerForKey:(NSString *)defaultName;

获取整型数据

\- (float)floatForKey:(NSString *)defaultName;

获取浮点型数据

\- (double)doubleForKey:(NSString *)defaultName;

获取双精度浮点型数据

\- (BOOL)boolForKey:(NSString *)defaultName;

获取布尔诗句

\- (NSURL *)URLForKey:(NSString *)defaultName;

获取网址数据

下面是一些对应的set方法

\- (void)setInteger:(NSInteger)value forKey:(NSString *)defaultName;

\- (void)setFloat:(float)value forKey:(NSString *)defaultName;

\- (void)setDouble:(double)value forKey:(NSString *)defaultName;

\- (void)setBool:(BOOL)value forKey:(NSString *)defaultName;

\- (void)setURL:(NSURL *)url forKey:(NSString *)defaultName;

\- (void)registerDefaults:(NSDictionary *)registrationDictionary;

这个方法可以通过字典对数据表进行赋值

\- (void)addSuiteNamed:(NSString *)suiteName;

添加一个域

\- (void)removeSuiteNamed:(NSString *)suiteName;

移除一个域

\- (NSDictionary *)dictionaryRepresentation;

返回系统设置的信息，也是NSGlobalDomain域中的信息。

[@property](http://my.oschina.net/property) (readonly, copy) NSArray *volatileDomainNames;

返回一个数组，其中是所有不稳定域的名字

\- (NSDictionary *)volatileDomainForName:(NSString *)domainName;

根据名字获取不稳定域中的数据

\- (void)setVolatileDomain:(NSDictionary *)domain forName:(NSString *)domainName;

根据名字设置不稳定域

\- (void)removeVolatileDomainForName:(NSString *)domainName;

根据名字移除不稳定域

\- (NSDictionary *)persistentDomainForName:(NSString *)domainName;

根据名字获取稳定域的数据

\- (void)setPersistentDomain:(NSDictionary *)domain forName:(NSString *)domainName;

根据名字设置稳定域

\- (void)removePersistentDomainForName:(NSString *)domainName;

根据名字移除稳定域

\- (BOOL)synchronize;

对象的同步方法，将内存中的数据写入磁盘。

\- (BOOL)objectIsForcedForKey:(NSString *)key;

判断某个键值的数据是否存在

\- (BOOL)objectIsForcedForKey:(NSString *)key inDomain:(NSString *)domain;

判断某个域中某个键值的数据是否存在

注：目前的iOS版本已经不能通过下面的方法在应用间进行传值！！！

### 二、三个特殊的域及实现简单的应用间信息传递

我们应该了解到，在IOS中，因为沙盒模式的存在，应用间是不允许互相访问数据与传值通信的。这样做的好处显而易见：

1、保证了数据的安全性

2、数据的管理更加简洁

3、当我们删除数据时，只需要将沙盒删除。

在某些需求下，我们可能会需要应用程序间的传值与通信，当然除了通过网络外，对于非常小的数据量，比如验证另一应用从程序是否登录，是否安装并且开启过一次，我们也可以通过NSUserDefaults的一个全局的数据表来实现。

NSUserDefaults的三个特殊的系统域如下：

NSString \* const NSGlobalDomain;

这个是一个系统级别的全局的域，存储这系统配置信息，我们可以通过它实现应用程序间传值

NSString \* const NSArgumentDomain;

这个是在本应用程序内可访问的域，存储着应用程序的信息

NSString \* const NSRegistrationDomain;

这个是存放临时数据的域

代码示例如下：

首先在第一个工程中，我们写如下代码运运行一下：

```
    //获取全局的域
    NSDictionary * dic = [[NSUserDefaults standardUserDefaults]persistentDomainForName:NSGlobalDomain];
    NSMutableDictionary * temDic = [NSMutableDictionary dictionaryWithDictionary:dic];
    [temDic setObject:@"传递的值" forKey:@"应用1"];
    //重设
    [[NSUserDefaults standardUserDefaults]setPersistentDomain:temDic forName:NSGlobalDomain];
    //同步
    [NSUserDefaults resetStandardUserDefaults];
    NSLog(@"%@",dic);
```

打印的结果是许多系统信息。

在第二个工程中，我们这样做：

```
 NSDictionary * dic = [[NSUserDefaults standardUserDefaults]persistentDomainForName:NSGlobalDomain];
    NSLog(@"%@\n--------------\n%@=%@",dic,@"应用1",[dic objectForKey:@"应用1"]);
```

结果如下：

```
2015-05-06 15:48:49.897 321[4100:186745] {
    AppleITunesStoreItemKinds =     (
        newsstand,
        podcast,
        "itunes-u",
        artist,
        booklet,
        document,
        eBook,
        software,
        "software-update",
        "podcast-episode"
    );
    AppleLanguages =     (
        en,
        fr,
        de,
        "zh-Hans",
        "zh-Hant",
        ja,
        nl,
        it,
        es,
        "es-MX",
        ko,
        pt,
        "pt-PT",
        da,
        fi,
        nb,
        sv,
        ru,
        pl,
        tr,
        uk,
        ar,
        hr,
        cs,
        el,
        he,
        ro,
        sk,
        th,
        id,
        ms,
        "en-GB",
        "en-AU",
        ca,
        hu,
        vi,
        hi
    );
    AppleLocale = "en_US";
    MSVLoggingMasterSwitchEnabledKey = 0;
    "\U5e94\U75281" = "\U4f20\U9012\U7684\U503c";
}
--------------
应用1=传递的值
```

这样，我们就简单实现了应用程序间的传值，但是建议最好不要轻易操作系统的这个域。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

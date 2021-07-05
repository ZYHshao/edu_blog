---
title: iOS9系列专题一——3D Touch
date: 2015-09-26
categories: iOS9专题
tags: []
---
## 新的触摸体验——iOS9的3D Touch

### 一、引言

        在iphone6s问世之后，很多果粉都争先要体验3D Touch给用户带来的额外维度上的交互，这个设计之所以叫做3D Touch，其原理上是增加了一个压力的感触，通过区分轻按和重按来进行不同的用户交互。

### 二、在模拟器上学习和测试3D Touch

        3D Touch是一个很新颖的设计，可是苹果文档有言：

> -   With Xcode 7.0 you must develop on a device that supports 3D Touch. Simulator in Xcode 7.0 does not support 3D Touch.
>     

看到这句话心是不是凉了一半，是的，xcode7是支持3D Touch开发的，可是模拟器并不支持这个手势，我们只能在真机上进行学习与测试，但是在IT的世界，从来都不缺拯救世界的人物，github上有人为我们提供了这样的一个插件，可以让我们在模拟器上进行3D Touch的效果测试：

git地址：[https://github.com/DeskConnect/SBShortcutMenuSimulator](https://github.com/DeskConnect/SBShortcutMenuSimulator)。

#### 附.SBShortcutMenuSimulator的安装和使用

        其实安装和使用并不需要怎么介绍，git主页里介绍的很清楚，这里在记录一遍，其中只有一点需要注意，如果你像我一样，电脑中装有Xcode6和Xcode7两个版本，那个Xcode的编译路径，需要做一下修改。

安装：

在终端中一次运行如下指令：

```
git clone https://github.com/DeskConnect/SBShortcutMenuSimulator.git
cd SBShortcutMenuSimulator
make
```

如果电脑中有多个Xcode版本，先做如下操作，如果只有Xcode7，则可以跳过

```
sudo xcode-select -switch /Applications/Xcode2.app/Contents/Developer/
```

注意：上面命令中，Xcode2.app是你电脑中Xcode的名字，这里如要特别注意，如果名字中有空格，需要修改一下，把空格去掉，否则会影响命令的执行。

之后在SBShortcutMenuSimulator的目录中执行如下操作：

```
xcrun simctl spawn booted launchctl debug system/com.apple.SpringBoard --environment DYLD_INSERT_LIBRARIES=$PWD/SBShortcutMenuSimulator.dylib
xcrun simctl spawn booted launchctl stop com.apple.SpringBoard
```

如果没有报错，我们可以通过向指定端口发送消息的方法来在模拟器上模拟3D Touch的效果：

```
echo 'com.apple.mobilecal' | nc 127.0.0.1 8000
```

其中，com.apple.mobilecal是应用的Bundle ID ，如果要测试我们的应用，将其改为我们应用的BundleID即可，上面的示例应用是系统日历，可以看到模拟器的效果如下：

![](http://static.oschina.net/uploads/space/2015/0926/110201_r1gd_2340880.png)

### 三、3D Touch的主要应用

        文档给出的应用介绍主要有两块：

> -   1.A user can now press your Home screen icon to immediately access functionality provided by your app.
>     
> -   2.Within your app, a user can now press views to see previews of additional content and gain accelerated access to features.
>     

        第一部分的应用是我们可以通过3D手势，在主屏幕上的应用Icon处，直接进入应用的响应功能模块。这个功能就例如我们上面的日历示例，会在Icon旁边出现一个菜单，点击菜单我们可以进入相应的功能单元。

        我个人理解，这个功能，push消息功能加上iOS8推出的扩展today功能，这三个机制使iOS应用变得无比灵活方便，用户可以不需付出寻找的时间成本来快速使用自己需要的功能。

        第二部分是对app的一个优化，用户可以通过3D Touch手势在view上来预览一些预加载信息，这样的设计可以使app更加简洁大方，交互性也更强。

### 四、3D Touch的三大模块

        在我们的app中使用3D Touch功能，主要分为以下三个模块：

#### 1、Home Screen Quick Actions

        通过主屏幕的应用Icon，我们可以用3D Touch呼出一个菜单，进行快速定位应用功能模块相关功能的开发。如上面的日历。

#### 2、peek and pop

        这个功能是一套全新的用户交互机制，在使用3D Touch时，ViewController中会有如下三个交互阶段：

        （1）提示用户这里有3D Touch的交互，会使交互控件周围模糊

![](http://static.oschina.net/uploads/space/2015/0926/113409_5jmO_2340880.png)

        （2）继续深按，会出现预览视图

![](http://static.oschina.net/uploads/space/2015/0926/113531_icSc_2340880.png)

        （3）通过视图上的交互控件进行进一步交互

![](http://static.oschina.net/uploads/space/2015/0926/113626_O35G_2340880.png)

这个模块的设计可以在网址连接上进行网页的预览交互。

#### 3.Force Properties

        iOS9为我们提供了一个新的交互参数:力度。我们可以检测某一交互的力度值，来做相应的交互处理。例如，我们可以通过力度来控制快进的快慢，音量增加的快慢等。

### 五、Home Screen Quick Action使用与相关api详解

    iOS9为我们提供了两种屏幕标签，分别是静态标签和动态标签。

#### 1、静态标签

    静态标签是我们在项目的配置plist文件中配置的标签，在用户安装程序后就可以使用，并且排序会在动态标签的前面。

我们先来看静态标签的配置：

首先，在info.plist文件中添加如下键值（我在测试的时候，系统并没有提示，只能手打上去）：

![](http://static.oschina.net/uploads/space/2015/0926/171313_aywB_2340880.png)

先添加了一个UIApplicationShortcutItems的数组，这个数组中添加的元素就是对应的静态标签，在每个标签中我们需要添加一些设置的键值：

必填项（下面两个键值是必须设置的）：

UIApplicationShortcutItemType 这个键值设置一个快捷通道类型的字符串 

UIApplicationShortcutItemTitle 这个键值设置标签的标题

选填项（下面这些键值不是必须设置的）：

UIApplicationShortcutItemSubtitle 设置标签的副标题

UIApplicationShortcutItemIconType 设置标签Icon类型

UIApplicationShortcutItemIconFile  设置标签的Icon文件

UIApplicationShortcutItemUserInfo 设置信息字典(用于传值)

我们如上截图设置后，运行程序，用我们前面的方法进行测试，效果如下：

![](http://static.oschina.net/uploads/space/2015/0926/172431_lbhm_2340880.png)

#### 2、动态标签

动态标签是我们在程序中，通过代码添加的，与之相关的类，主要有三个：

UIApplicationShortcutItem 创建3DTouch标签的类

UIMutableApplicationShortcutItem 创建可变的3DTouch标签的类

UIApplicationShortcutIcon 创建标签中图片Icon的类

因为这些类是iOS9中新增加的类，所以其api的复杂程度并不大，下面我们来对其中方法与属性进行简要讲解：

```
@interface UIApplicationShortcutItem : NSObject <NSCopying, NSMutableCopying>
//下面是两个初始化方法 通过设置type，title等属性来创建一个标签，这里的icon是UIApplicationShortcutIcon对象，我们后面再说
- (instancetype)initWithType:(NSString *)type localizedTitle:(NSString *)localizedTitle localizedSubtitle:(nullable NSString *)localizedSubtitle icon:(nullable UIApplicationShortcutIcon *)icon userInfo:(nullable NSDictionary *)userInfo NS_DESIGNATED_INITIALIZER;
- (instancetype)initWithType:(NSString *)type localizedTitle:(NSString *)localizedTitle;
//下面这是一些只读的属性，获取相应的属性值
@property (nonatomic, copy, readonly) NSString *type;
@property (nonatomic, copy, readonly) NSString *localizedTitle;
@property (nullable, nonatomic, copy, readonly) NSString *localizedSubtitle;
@property (nullable, nonatomic, copy, readonly) UIApplicationShortcutIcon *icon;
@property (nullable, nonatomic, copy, readonly) NSDictionary<NSString *, id <NSSecureCoding>> *userInfo;
```

```
//这个类继承于 UIApplicationShortcutItem，创建的标签可变
@interface UIMutableApplicationShortcutItem : UIApplicationShortcutItem
@property (nonatomic, copy) NSString *type;
@property (nonatomic, copy) NSString *localizedTitle;
@property (nullable, nonatomic, copy) NSString *localizedSubtitle;
@property (nullable, nonatomic, copy) UIApplicationShortcutIcon *icon;
@property (nullable, nonatomic, copy) NSDictionary<NSString *, id <NSSecureCoding>> *userInfo;

@end
```

```
//这个类创建标签中的icon
@interface UIApplicationShortcutIcon : NSObject <NSCopying>
//创建系统风格的icon
+ (instancetype)iconWithType:(UIApplicationShortcutIconType)type;
//创建自定义的图片icon
+ (instancetype)iconWithTemplateImageName:(NSString *)templateImageName;
@end
```

创建好标签后，将其添加如application的hortcutItems数组中即可，示例如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    //创建
    UIApplicationShortcutItem * item = [[UIApplicationShortcutItem alloc]initWithType:@"two" localizedTitle:@"第二个标签" localizedSubtitle:@"看我哦" icon:[UIApplicationShortcutIcon iconWithType:UIApplicationShortcutIconTypePlay] userInfo:nil];
    添加
    [UIApplication sharedApplication].shortcutItems = @[item];
}
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/0926/174937_Ahnk_2340880.png)

这里，将系统风格icon的枚举列举如下：

```
typedef NS_ENUM(NSInteger, UIApplicationShortcutIconType) {
    UIApplicationShortcutIconTypeCompose,//编辑的图标
    UIApplicationShortcutIconTypePlay,//播放图标
    UIApplicationShortcutIconTypePause,//暂停图标
    UIApplicationShortcutIconTypeAdd,//添加图标
    UIApplicationShortcutIconTypeLocation,//定位图标
    UIApplicationShortcutIconTypeSearch,//搜索图标
    UIApplicationShortcutIconTypeShare//分享图标
} NS_ENUM_AVAILABLE_IOS(9_0);
```

#### 3、响应标签的行为

类似推送，当我们点击标签进入应用程序时，也可以进行一些操作，我们可以看到，在applocation中增加了这样一个方法：

\- (void)application:(UIApplication *)application performActionForShortcutItem:(UIApplicationShortcutItem *)shortcutItem completionHandler:(void(^)(BOOL succeeded))completionHandler NS\_AVAILABLE\_IOS(9_0);

当我们通过标签进入app时，就会在appdelegate中调用这样一个回调，我们可以获取shortcutItem的信息进行相关逻辑操作。

这里有一点需要注意：我们在app的入口函数：

\- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions;

也需要进行一下判断，在launchOptions中有UIApplicationLaunchOptionsShortcutItemKey这样一个键，通过它，我们可以区别是否是从标签进入的app，如果是则处理结束逻辑后，返回NO，防止处理逻辑被反复回调。 

几点注意：

1、快捷标签最多可以创建四个，包括静态的和动态的。

2、每个标签的题目和icon最多两行，多出的会用...省略

### 六、结语

        关于3DTouch在UIView中的预览功能和UITouch中新增加的力度属性的应用，因为不好演示，这里就不再总结，大家可以通过头文件中相应的类和属性来了解他们，最后，如有疏漏和错误之处，欢迎指正。

欢迎转载 请注明出处

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

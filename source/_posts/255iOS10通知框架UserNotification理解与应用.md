---
title: iOS10通知框架UserNotification理解与应用
date: 2016-09-18
categories: iOS10专题
tags: []
---
## iOS10通知框架UserNotification理解与应用

### 一、引言

        关于通知，无论与远程Push还是本地通知，以往的iOS系统暴漏给开发者的接口都是十分有限的，开发者只能对标题和内容进行简单的定义，至于UI展示和用户交互行为相关的部分，开发者开发起来都十分困难。至于本地通知，iOS10之前采用的是UILocationNotification类，远程通知有苹果服务器进行转发，本地通知和远程通知其回调的处理都是通过AppDelegate中的几个回调方法来完成。iOS10系统中，通知功能的增强是一大优化之处，iOS10中将通知功能整合成了一个框架UserNotification，其结构十分类似于iOS8中的UIWebView向WebKit框架整合的思路。并且UserNotification相比之前的通知功能更加强大，主要表现在如下几点：

1.通知处理代码可以从AppDelegate中剥离。

2.通知的注册，设置，处理更加结构化，更易于模块化开发。

3.UserNotification支持自定义通知音效和启动图。

4.UserNotification支持向通知内容中添加媒体附件，例如音频，视频。

5.UserNotification支持开发者定义多套通知模板。

6.UserNotification支持完全自定义的通知界面。

7.UserNotification支持自定义通知中的用户交互按钮。

8.通知的触发更加容易管理。

从上面列举的几点就可以看出，iOS10中的UsreNotification真的是一个大的改进，温故而知新，关于iOS之前版本本地通知和远程通知的相关内容请查看如下博客：

本地推送：[http://my.oschina.net/u/2340880/blog/405491](http://my.oschina.net/u/2340880/blog/405491)。

远程推送：[http://my.oschina.net/u/2340880/blog/413584](http://my.oschina.net/u/2340880/blog/413584)。

### 二、UserNotification概览

        学习一个新的框架或知识模块时，宏观上了解其体系，大体上掌握其结构是十分必要的，这更有利于我们对这个框架或模块的整体把握与理解。UserNotification框架中拆分定义了许多类、枚举和结构体，其中还定义了许多常量，类与类之间虽然关系复杂，但脉络十分清晰，把握住主线，层层分析，边很容易理解和应用UserNotification框架。

        下图中列举了UserNotification框架中所有核心的类：

![](https://static.oschina.net/uploads/space/2016/0917/211634_xIMM_2340880.png)

如图中关系所示，UserNotification框架中的核心类列举如下：

UNNotificationCenter：通知管理中心，单例，通知的注册，接收通知后的回调处理等，是UserNotification框架的核心。

UNNotification：通知对象，其中封装了通知请求。

UNNotificationSettings：通知相关设置。

UNNotificationCategory：通知模板。

UNNotificationAction：用于定义通知模板中的用户交互行为。

UNNotificationRequest：注册通知请求，其中定义了通知的内容和触发方式。

UNNotificationResponse：接收到通知后的回执。

UNNotificationContent：通知的具体内容。

UNNotificationTrigger：通知的触发器，由其子类具体定义。

UNNotificationAttachment：通知附件类，为通知内容添加媒体附件。

UNNotificationSound：定义通知音效。

UNPushNotificationTrigger：远程通知的触发器，UNNotificationTrigger子类。

UNTimeInervalNotificationTrigger：计时通知的触发器，UNNotificationTrigger子类。

UNCalendarNotificationTrigger：周期通知的触发器，UNNotificationTrigger子类。

UNLocationNotificationTrigger：地域通知的触发器，UNNotificationTrigger子类。

UNNotificationCenterDelegate：协议，其中方法用于监听通知状态。

### 三、进行通知用户权限申请与创建普通的本地通知

        要在iOS系统中使用通知，必须获取到用户权限，UserNotification框架中申请通知用户权限需要通过UNNotificationCenter来完成，示例如下：

```objectivec
//进行用户权限的申请
[[UNUserNotificationCenter currentNotificationCenter] requestAuthorizationWithOptions:UNAuthorizationOptionBadge|UNAuthorizationOptionSound|UNAuthorizationOptionAlert|UNAuthorizationOptionCarPlay completionHandler:^(BOOL granted, NSError * _Nullable error) {
      //在block中会传入布尔值granted，表示用户是否同意
      if (granted) {
            //如果用户权限申请成功，设置通知中心的代理
            [UNUserNotificationCenter currentNotificationCenter].delegate = self;
      }
}];
```

申请用户权限的方法中需要传入一个权限内容的参数，其枚举定义如下：

```objectivec
typedef NS_OPTIONS(NSUInteger, UNAuthorizationOptions) {
    //允许更新app上的通知数字
    UNAuthorizationOptionBadge   = (1 << 0),
    //允许通知声音
    UNAuthorizationOptionSound   = (1 << 1),
    //允许通知弹出警告
    UNAuthorizationOptionAlert   = (1 << 2),
    //允许车载设备接收通知
    UNAuthorizationOptionCarPlay = (1 << 3),
};
```

获取到用户权限后，使用UserNotification创建普通的通知，示例代码如下：

```objectivec
    //通知内容类
    UNMutableNotificationContent * content = [UNMutableNotificationContent new];
    //设置通知请求发送时 app图标上显示的数字
    content.badge = @2;
    //设置通知的内容
    content.body = @"这是iOS10的新通知内容：普通的iOS通知";
    //默认的通知提示音
    content.sound = [UNNotificationSound defaultSound];
    //设置通知的副标题
    content.subtitle = @"这里是副标题";
    //设置通知的标题
    content.title = @"这里是通知的标题";
    //设置从通知激活app时的launchImage图片
    content.launchImageName = @"lun";
    //设置5S之后执行
    UNTimeIntervalNotificationTrigger * trigger = [UNTimeIntervalNotificationTrigger triggerWithTimeInterval:5 repeats:NO];
    UNNotificationRequest * request = [UNNotificationRequest requestWithIdentifier:@"NotificationDefault" content:content trigger:trigger];
    //添加通知请求
    [[UNUserNotificationCenter currentNotificationCenter] addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
        
    }];
```

效果如下面图示：

![](https://static.oschina.net/uploads/space/2016/0917/221208_lU3E_2340880.png)               ![](https://static.oschina.net/uploads/space/2016/0917/221229_lWxR_2340880.png)

![](https://static.oschina.net/uploads/space/2016/0917/221247_qDOi_2340880.png)

### 四、通知音效类UNNotificationSound

        通知可以进行自定义的音效设置，其中方法如下：

```objectivec
//系统默认的音效
+ (instancetype)defaultSound;
//自定义的音频音效
/*
注意，音频文件必须在bundle中或者在Library/Sounds目录下
*/
+ (instancetype)soundNamed:(NSString *)name __WATCHOS_PROHIBITED;

```

### 五、通知触发器UNNotificationTrigger

        通知触发器可以理解为定义通知的发送时间，UNNotificationTrigger是触发器的基类，具体的触发器由它的四个子类实现，实际上，开发者在代码中可能会用到的触发器只有三种，UNPushNotificationTrigger远程推送触发器开发者不需要创建使用，远程通知有远程服务器触发，开发者只需要创建与本地通知有关的触发器进行使用。

#### 1.UNTimeIntervalNotificationTrigger

        UNTimeIntervalNotificationTrigger是计时触发器，开发者可以设置其在添加通知请求后一定时间发送。

```objectivec
//创建触发器 在timeInterval秒后触发 可以设置是否循环触发
+ (instancetype)triggerWithTimeInterval:(NSTimeInterval)timeInterval repeats:(BOOL)repeats;
//获取下次触发的时间点
- (nullable NSDate *)nextTriggerDate;
```

#### 2.UNCalendarNotificationTrigger

        UNCalendarNotificationTrigger是日历触发器，开发者可以设置其在某个时间点触发。

```objectivec
//创建触发器 设置触发时间 可以设置是否循环触发
+ (instancetype)triggerWithDateMatchingComponents:(NSDateComponents *)dateComponents repeats:(BOOL)repeats;
//下一次触发的时间点
- (nullable NSDate *)nextTriggerDate;
```

#### 3.UNLocationNotificationTrigger

        UNLocationNotificationTrigger是地域触发器，开发者可以设置当用户进入某一区域时触发。

```objectivec
//地域信息
@property (NS_NONATOMIC_IOSONLY, readonly, copy) CLRegion *region;
//创建触发器
+ (instancetype)triggerWithRegion:(CLRegion *)region repeats:(BOOL)repeats __WATCHOS_PROHIBITED;
```

### 六、为通知内容添加附件

        附件主要指的是媒体附件，例如图片，音频和视频，为通知内容添加附件需要使用UNNotificationAttachment类。示例代码如下：

```objectivec
    //创建图片附件
    UNNotificationAttachment * attach = [UNNotificationAttachment attachmentWithIdentifier:@"imageAttach" URL:[NSURL fileURLWithPath:[[NSBundle mainBundle] pathForResource:@"2" ofType:@"jpg"]] options:nil error:nil];
    UNMutableNotificationContent * content = [UNMutableNotificationContent new];
    //设置附件数组
    content.attachments = @[attach];
    content.badge = @1;
    content.body = @"这是iOS10的新通知内容：普通的iOS通知";
    //默认的通知提示音
    content.sound = [UNNotificationSound defaultSound];
    content.subtitle = @"这里是副标题";
    content.title = @"这里是通知的标题";
    //设置5S之后执行
    UNTimeIntervalNotificationTrigger * trigger = [UNTimeIntervalNotificationTrigger triggerWithTimeInterval:5 repeats:NO];
    UNNotificationRequest * request = [UNNotificationRequest requestWithIdentifier:@"NotificationDefaultImage" content:content trigger:trigger];
    [[UNUserNotificationCenter currentNotificationCenter] addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
        
    }];
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/0917/224208_aCSY_2340880.png)                 ![](https://static.oschina.net/uploads/space/2016/0917/224247_CpEZ_2340880.png)

需要注意，UNNotificationContent的附件数组虽然是一个数组，但是系统的通知模板只能展示其中的第一个附件，设置多个附件也不会有额外的效果，但是如果开发者进行通知模板UI的自定义，则此数组就可以派上用场了。音频附件界面如下：

![](https://static.oschina.net/uploads/space/2016/0917/230851_6oc4_2340880.png)

需要注意，添加附件的格式和大小都有一定的要求，如下表格所示：

![](https://static.oschina.net/uploads/space/2016/0917/231006_q7Y8_2340880.png)

创建通知内容附件UNNotificationAttachment实例的方法中有一个options配置字典，这个字典中可以进行配置的键值对如下：

```objectivec
//配置附件的类型的键 需要设置为NSString类型的值，如果不设置 则默认从扩展名中推断
extern NSString * const UNNotificationAttachmentOptionsTypeHintKey __IOS_AVAILABLE(10.0) __WATCHOS_AVAILABLE(3.0);
//配置是否隐藏缩略图的键 需要配置为NSNumber 0或者1
extern NSString * const UNNotificationAttachmentOptionsThumbnailHiddenKey __IOS_AVAILABLE(10.0) __WATCHOS_AVAILABLE(3.0);
//配置使用一个标准的矩形来对缩略图进行裁剪，需要配置为CGRectCreateDictionaryRepresentation(CGRect)创建的矩形引用
extern NSString * const UNNotificationAttachmentOptionsThumbnailClippingRectKey __IOS_AVAILABLE(10.0) __WATCHOS_AVAILABLE(3.0);
//使用视频中的某一帧作为缩略图 配置为NSNumber时间
extern NSString * const UNNotificationAttachmentOptionsThumbnailTimeKey __IOS_AVAILABLE(10.0) __WATCHOS_AVAILABLE(3.0);
```

### 七、定义通知模板UNNotificationCategory

        聊天类软件在iOS系统中，常常采用后台推送的方式推送新消息，用户可以在不进入应用程序的情况下，直接在左面回复通知推送过来的信息，这种功能就是通过UNNotificationCategory模板与UNNotificationAction用户活动来实现的。关于文本回复框，UserNotification框架中提供了UNTextInputNotificationAction类，其是UNNotificationAction的子类。示例代码如下：

```objectivec
    //创建用户活动
    /*
    options参数可选如下：
    //需要在解开锁屏下使用
    UNNotificationActionOptionAuthenticationRequired
    //是否指示有破坏性
    UNNotificationActionOptionDestructive
    //是否允许活动在后台启动app
    UNNotificationActionOptionForeground
    //无设置
    UNNotificationActionOptionNone
    */
    UNTextInputNotificationAction * action = [UNTextInputNotificationAction actionWithIdentifier:@"action" title:@"回复" options:UNNotificationActionOptionAuthenticationRequired textInputButtonTitle:@"活动" textInputPlaceholder:@"请输入回复内容"];
    //创建通知模板
    UNNotificationCategory * category = [UNNotificationCategory categoryWithIdentifier:@"myNotificationCategoryText" actions:@[action] intentIdentifiers:@[] options:UNNotificationCategoryOptionCustomDismissAction];
    UNMutableNotificationContent * content = [UNMutableNotificationContent new];
    content.badge = @1;
    content.body = @"这是iOS10的新通知内容：普通的iOS通知";
    //默认的通知提示音
    content.sound = [UNNotificationSound defaultSound];
    content.subtitle = @"这里是副标题";
    content.title = @"这里是通知的标题";
    //设置通知内容对应的模板 需要注意 这里的值要与对应模板id一致
    content.categoryIdentifier = @"myNotificationCategoryText";
    //设置5S之后执行
    UNTimeIntervalNotificationTrigger * trigger = [UNTimeIntervalNotificationTrigger triggerWithTimeInterval:5 repeats:NO];
      [[UNUserNotificationCenter currentNotificationCenter] setNotificationCategories:[NSSet setWithObjects:category, nil]];
    UNNotificationRequest * request = [UNNotificationRequest requestWithIdentifier:@"NotificationDefaultText" content:content trigger:trigger];
  
    [[UNUserNotificationCenter currentNotificationCenter] addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
        
    }];
```

需要注意，要使用模板，通知内容UNNotificationContent的categoryIdentifier要与UNNotificationCategory的id一致。效果如下：

![](https://static.oschina.net/uploads/space/2016/0917/234551_6zOM_2340880.png)

        也可以为通知模板添加多个自定义的用户交互按钮，示例如下：

```objectivec
UNNotificationAction * action = [UNNotificationAction actionWithIdentifier:@"action" title:@"活动标题1" options:UNNotificationActionOptionNone];
    UNNotificationAction * action2 = [UNNotificationAction actionWithIdentifier:@"action" title:@"活动标题2" options:UNNotificationActionOptionNone];
    UNNotificationAction * action3 = [UNNotificationAction actionWithIdentifier:@"action" title:@"活动标题3" options:UNNotificationActionOptionNone];
    UNNotificationAction * action4 = [UNNotificationAction actionWithIdentifier:@"action" title:@"活动标题4" options:UNNotificationActionOptionNone];
     UNNotificationCategory * category = [UNNotificationCategory categoryWithIdentifier:@"myNotificationCategoryBtn" actions:@[action,action2,action3,action4] intentIdentifiers:@[] options:UNNotificationCategoryOptionCustomDismissAction];
    UNMutableNotificationContent * content = [UNMutableNotificationContent new];
    content.badge = @1;
    content.body = @"这是iOS10的新通知内容：普通的iOS通知";
    //默认的通知提示音
    content.sound = [UNNotificationSound defaultSound];
    content.subtitle = @"这里是副标题";
    content.title = @"这里是通知的标题";
    content.categoryIdentifier = @"myNotificationCategoryBtn";
    //设置5S之后执行
    UNTimeIntervalNotificationTrigger * trigger = [UNTimeIntervalNotificationTrigger triggerWithTimeInterval:5 repeats:NO];
    UNNotificationRequest * request = [UNNotificationRequest requestWithIdentifier:@"NotificationDefault" content:content trigger:trigger];
    [[UNUserNotificationCenter currentNotificationCenter] setNotificationCategories:[NSSet setWithObjects:category, nil]];
    [[UNUserNotificationCenter currentNotificationCenter] addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
        
    }];

```

需要注意，系统模板最多支持添加4个用户交互按钮，如下图：

![](https://static.oschina.net/uploads/space/2016/0917/235115_49tT_2340880.png)

### 八、自定义通知模板UI

        通过前边的介绍，我们发现通过UserNotification框架开发者已经可以完成许多从来很难实现的效果。然而这都不是UserNotification框架最强大的地方，UserNotification框架最强大的地方在于其可以完全自定义通知的UI界面。

        完全自定义通知界面是通过iOS扩展来实现的，首先创建一个新的target，如下图：

![](https://static.oschina.net/uploads/space/2016/0917/235730_yGEj_2340880.png)

选择Notification Content，如下：

![](https://static.oschina.net/uploads/space/2016/0917/235756_yYly_2340880.png)

创建完成后，会发现工程中多了一个Notification Content的扩展，其中自带一个storyboard文件和一个NotificationViewController类，开发者可以在storyboard文件或者直接在Controller类中进行自定义界面的编写。

    需要注意，NotificationViewController自动遵守了UNNotificationContentExtension协议，这个协议专门用来处理自定义通知UI的内容展示，其中方法列举如下：

```objectivec
//接收到通知时会被调用
/*
开发者可以从notification对象中拿到附件等内容进行UI刷新
*/
- (void)didReceiveNotification:(UNNotification *)notification;
//当用户点击了通知中的用户交互按钮时会被调用
/*
response对象中有通知内容相关信息
在回调block块completion中，开发者可以传入一个UNNotificationContentExtensionResponseOption参数来告诉系统如何处理这次用户活动
UNNotificationContentExtensionResponseOption枚举中可选值如下：
typedef NS_ENUM(NSUInteger, UNNotificationContentExtensionResponseOption) {
    //不关闭当前通知界面
    UNNotificationContentExtensionResponseOptionDoNotDismiss,
    //关闭当前通知界面
    UNNotificationContentExtensionResponseOptionDismiss,
    //关闭当前通知界面并将用户活动传递给宿主app处理
    UNNotificationContentExtensionResponseOptionDismissAndForwardAction,
} __IOS_AVAILABLE(10_0) __TVOS_UNAVAILABLE __WATCHOS_UNAVAILABLE __OSX_UNAVAILABLE;
*/
- (void)didReceiveNotificationResponse:(UNNotificationResponse *)response completionHandler:(void (^)(UNNotificationContentExtensionResponseOption option))completion;
/*
这个属性作为get方法进行实现 这个方法用来返回一个通知界面要显示的媒体按钮
typedef NS_ENUM(NSUInteger, UNNotificationContentExtensionMediaPlayPauseButtonType) {
    //不显示媒体按钮
    UNNotificationContentExtensionMediaPlayPauseButtonTypeNone,
    //默认的媒体按钮 当点击按钮后 进行播放与暂停的切换 按钮始终显示
    UNNotificationContentExtensionMediaPlayPauseButtonTypeDefault,
    //Overlay风格 当点击按钮后，媒体播放，按钮隐藏 点击媒体后，播放暂停，按钮显示。
    UNNotificationContentExtensionMediaPlayPauseButtonTypeOverlay,
} __IOS_AVAILABLE(10_0) __TVOS_UNAVAILABLE __WATCHOS_UNAVAILABLE __OSX_UNAVAILABLE;
*/
@property (nonatomic, readonly, assign) UNNotificationContentExtensionMediaPlayPauseButtonType mediaPlayPauseButtonType;
//返回媒体按钮的位置
@property (nonatomic, readonly, assign) CGRect mediaPlayPauseButtonFrame;
//返回媒体按钮的颜色
@property (nonatomic, readonly, copy) UIColor *mediaPlayPauseButtonTintColor;
//点击播放按钮的回调
- (void)mediaPlay;
//点击暂停按钮的回调
- (void)mediaPause;
//媒体开始播放的回调
- (void)mediaPlayingStarted __IOS_AVAILABLE(10_0) __TVOS_UNAVAILABLE __WATCHOS_UNAVAILABLE __OSX_UNAVAILABLE;
//媒体开始暂停的回调
- (void)mediaPlayingPaused __IOS_AVAILABLE(10_0) __TVOS_UNAVAILABLE __WATCHOS_UNAVAILABLE __OSX_UNAVAILABLE;
```

需要注意，自定义的通知界面上虽然可以放按钮，可以放任何UI控件，但是其不能进行用户交互，唯一可以进行用户交互的方式是通过协议中的媒体按钮及其回调方法。

        定义好了通知UI模板，若要进行使用，还需要再Notification Content扩展中的info.plist文件的NSExtension字典的NSExtensionAttributes字典里进行一些配置，正常情况下，开发者需要进行配置的键有3个，分别如下：

UNNotificationExtensionCategory：设置模板的categoryId，用于与UNNotificationContent对应。

UNNotificationExtensionInitialContentSizeRatio：设置自定义通知界面的高度与宽度的比，宽度为固定宽度，在不同设备上有差别，开发者需要根据宽度计算出高度进行设置，系统根据这个比值来计算通知界面的高度。

UNNotificationExtensionDefaultContentHidden：是有隐藏系统默认的通知界面。

配置info.plist文件如下：

![](https://static.oschina.net/uploads/space/2016/0918/002737_D9It_2340880.png)

用如下的代码创建通知：

```objectivec
UNNotificationAction * action = [UNNotificationAction actionWithIdentifier:@"action" title:@"活动标题1" options:UNNotificationActionOptionNone];
    //根据id拿到自定义UI的模板
    UNNotificationCategory * category = [UNNotificationCategory categoryWithIdentifier:@"myNotificationCategoryH" actions:@[action] intentIdentifiers:@[] options:UNNotificationCategoryOptionCustomDismissAction];
    UNMutableNotificationContent * content = [UNMutableNotificationContent new];
    content.badge = @1;
    content.body = @"这是iOS10的新通知内容：普通的iOS通知";
    //默认的通知提示音
    content.sound = [UNNotificationSound defaultSound];
    content.subtitle = @"这里是副标题";
    content.title = @"这里是通知的标题";
    content.categoryIdentifier = @"myNotificationCategoryH";
    //设置5S之后执行
    UNTimeIntervalNotificationTrigger * trigger = [UNTimeIntervalNotificationTrigger triggerWithTimeInterval:5 repeats:NO];
    UNNotificationRequest * request = [UNNotificationRequest requestWithIdentifier:@"NotificationDefaultCustomUIH" content:content trigger:trigger];
    [[UNUserNotificationCenter currentNotificationCenter] setNotificationCategories:[NSSet setWithObjects:category, nil]];
    [[UNUserNotificationCenter currentNotificationCenter] addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
        
    }];
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/0918/003502_aSyA_2340880.png)

如果将UNNotificationExtensionDefaultContentHidden键值设置为0或者不设置，则不会隐藏系统默认的UI，如下：

![](https://static.oschina.net/uploads/space/2016/0918/003630_KpYF_2340880.png)

### 九、通知回调的处理

        UserNotification框架对于通知的回调处理，是通过UNUserNotificationCenterDelegate协议来实现的，这个协议中有两个方法，如下：

```objectivec
/*
这个方法在应用在前台，并且将要弹出通知时被调用，后台状态下弹通知不会调用这个方法
这个方法中的block块completionHandler()可以传入一个UNNotificationPresentationOptions类型的枚举
有个这个参数，开发者可以设置在前台状态下，依然可以弹出通知消息，枚举如下：
typedef NS_OPTIONS(NSUInteger, UNNotificationPresentationOptions) {
    //只修改app图标的消息数
    UNNotificationPresentationOptionBadge   = (1 << 0),
    //只提示通知音效
    UNNotificationPresentationOptionSound   = (1 << 1),
    //只弹出通知框
    UNNotificationPresentationOptionAlert   = (1 << 2),
} __IOS_AVAILABLE(10.0) __TVOS_AVAILABLE(10.0) __WATCHOS_AVAILABLE(3.0);
//什么都不做
static const UNNotificationPresentationOptions UNNotificationPresentationOptionNone 
*/
- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler __IOS_AVAILABLE(10.0) __TVOS_AVAILABLE(10.0) __WATCHOS_AVAILABLE(3.0);
/*
这个方法当接收到通知后，用户点击通知激活app时被调用，无论前台还是后台
*/
- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)())completionHandler __IOS_AVAILABLE(10.0) __WATCHOS_AVAILABLE(3.0) __TVOS_PROHIBITED;
```

### 十、UserNotification框架中其他零散知识

        前面所介绍的内容基本涵盖了UserNotification框架中所有的内容，在以后的应用开发中，开发者可以在通知方面发挥更大的想象力与创造力，给用户更加友好的体验。除了前边所介绍过的核心内容外，UserNotification框架中还有一些零散的类、枚举等。

#### 1.错误码描述

```objectivec
typedef NS_ENUM(NSInteger, UNErrorCode) {
    //通知不被允许
    UNErrorCodeNotificationsNotAllowed = 1,
    
    //附件无效url
    UNErrorCodeAttachmentInvalidURL = 100,
    //附件类型错误
    UNErrorCodeAttachmentUnrecognizedType,
    //附件大小错误
    UNErrorCodeAttachmentInvalidFileSize,
    //附件数据错误
    UNErrorCodeAttachmentNotInDataStore,
    UNErrorCodeAttachmentMoveIntoDataStoreFailed,
    UNErrorCodeAttachmentCorrupt,
    
    //时间无效
    UNErrorCodeNotificationInvalidNoDate = 1400,
    //无内容
    UNErrorCodeNotificationInvalidNoContent,
} __IOS_AVAILABLE(10.0) __TVOS_AVAILABLE(10.0) __WATCHOS_AVAILABLE(3.0);
```

#### 2.UNNotification类

```objectivec
@interface UNNotification : NSObject <NSCopying, NSSecureCoding>
//触发的时间
@property (nonatomic, readonly, copy) NSDate *date;
//内置的通知请求对象
@property (nonatomic, readonly, copy) UNNotificationRequest *request;

- (instancetype)init NS_UNAVAILABLE;

@end
```

#### 3.UNNotificationSettings类

        UNNotificationSettings类主要用来获取与通知相关的信息。

```objectivec
@interface UNNotificationSettings : NSObject <NSCopying, NSSecureCoding>
//用户权限状态
@property (NS_NONATOMIC_IOSONLY, readonly) UNAuthorizationStatus authorizationStatus;
//音效设置
@property (NS_NONATOMIC_IOSONLY, readonly) UNNotificationSetting soundSetting __TVOS_PROHIBITED;
//图标提醒设置
@property (NS_NONATOMIC_IOSONLY, readonly) UNNotificationSetting badgeSetting __WATCHOS_PROHIBITED;
//提醒框设置
@property (NS_NONATOMIC_IOSONLY, readonly) UNNotificationSetting alertSetting __TVOS_PROHIBITED;
//通知中心设置
@property (NS_NONATOMIC_IOSONLY, readonly) UNNotificationSetting notificationCenterSetting __TVOS_PROHIBITED;
//锁屏设置
@property (NS_NONATOMIC_IOSONLY, readonly) UNNotificationSetting lockScreenSetting __TVOS_PROHIBITED __WATCHOS_PROHIBITED;
//车载设备设置
@property (NS_NONATOMIC_IOSONLY, readonly) UNNotificationSetting carPlaySetting __TVOS_PROHIBITED __WATCHOS_PROHIBITED;
//提醒框风格
@property (NS_NONATOMIC_IOSONLY, readonly) UNAlertStyle alertStyle __TVOS_PROHIBITED __WATCHOS_PROHIBITED;

@end
```

UNNotificationSetting枚举如下：

```objectivec
typedef NS_ENUM(NSInteger, UNNotificationSetting) {
    //不支持
    UNNotificationSettingNotSupported  = 0,
    
    //不可用
    UNNotificationSettingDisabled,
    
    //可用
    UNNotificationSettingEnabled,
} 
```

UNAuthorizationStatus枚举如下：

```objectivec
typedef NS_ENUM(NSInteger, UNAuthorizationStatus) {
    //为做选择
    UNAuthorizationStatusNotDetermined = 0,
    
    // 用户拒绝
    UNAuthorizationStatusDenied,
    
    // 用户允许
    UNAuthorizationStatusAuthorized
}
```

UNAlertStyle枚举如下：

```objectivec
typedef NS_ENUM(NSInteger, UNAlertStyle) {
    //无
    UNAlertStyleNone = 0,
    //顶部Banner样式
    UNAlertStyleBanner,
    //警告框样式
    UNAlertStyleAlert,
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

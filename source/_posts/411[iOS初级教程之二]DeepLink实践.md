---
title: iOS初级教程之二:DeepLink实践
date: 2021-03-19
categories: iOS逻辑初窥
tags: []
---
# \[iOS初级教程之二\]DeepLink实践

## 一、唤醒iOS应用程序的几种方式

      唤醒应用是iOS开发中常见的技术，应用唤醒的方式有多种，概括下来，可以分为如下几类:

-   直接打开App
-   通知唤醒
-   scheme唤醒
-   Universal Links唤醒

      直接打开App是最直接的唤醒应用程序的方式，以iPhone为例，可以从主屏幕、搜索推荐、应用库等场景中进行打开。

![](https://oscimg.oschina.net/oscnet/up-b7c5e71d145638d8657899d6cd37060ab95.png)    ![](https://oscimg.oschina.net/oscnet/up-d34207cea51be9edc1604e847711b0df8c4.png)    ![](https://oscimg.oschina.net/oscnet/up-ac78a0457cf08d2d4896988ce214b470de2.png)

      当用户收到通知消息时，点击通知栏的通知消息也可以唤醒应用程序，通知分为本地通知和远程通知两种，从唤醒应用程序的作用上来说，这两种通知的逻辑是一致的。下面代码示例了发送本地通知：

```objectivec
- (void)addLocalNotification {
    [[UNUserNotificationCenter currentNotificationCenter] requestAuthorizationWithOptions:UNAuthorizationOptionAlert | UNAuthorizationOptionBadge | UNAuthorizationOptionSound completionHandler:^(BOOL granted, NSError * _Nullable error) {
        
    }];
    UNMutableNotificationContent *notificaiton = [[UNMutableNotificationContent alloc] init];
    notificaiton.title = @"本地通知";
    notificaiton.body = @"通知内容";
    notificaiton.badge = @(1);
    
    UNTimeIntervalNotificationTrigger *trriger = [UNTimeIntervalNotificationTrigger triggerWithTimeInterval:5 repeats:NO];
    
    UNNotificationRequest *request = [UNNotificationRequest requestWithIdentifier:@"requsetID" content:notificaiton trigger:trriger];
    
    [[UNUserNotificationCenter currentNotificationCenter] addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
            
    }];
}
```

      通过Scheme来唤醒应用程序也非常简单，在上一篇教程中，我们有介绍分享功能的集成，就有配置和使用过Scheme。要通过Scheme唤醒应用程序，首先需要添加URL Types，添加方法如下图所示：

![](https://oscimg.oschina.net/oscnet/up-f0aaadcd6018110de154a76bea7f69f0a4a.png)

上面我们定义了一个URL Scheme为deeplinkdemo，我们可以直接在浏览器中使用deeplinkdemo://开头的链接来唤醒此应用程序。

      Universal Link是一种更加强大的唤醒应用程序的方式，相关介绍官网如下：

[https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html)

Universal Link是iOS 9之后新增的应用跳转功能，其可以无缝的连接网站与App，不需通过浏览器做中转，即可以实现应用间的无缝切换。

      了解了上述的几种应用程序的唤醒方式，我们回到本教程的主题：DeepLink。DeepLink又称为深度超链，其是指应用以上技术进行应用程序的无缝调起、场景还原、营销唤醒等业务逻辑。下面，我们就介绍如何以友盟工具平台为基础来为应用程序接入智能超链功能。

## 二、U-Link SDK集成

      使用CocoaPods的方式来集成U-Link SDK非常方便，只需要在Podfile文件夹中添加如下依赖即可：

```ruby
pod 'UMCommon'
pod 'UMDevice'
```

如果项目没有使用CocoaPods进行管理，也可以选择手动引入的方式来集成SDK。可以在如下网址下载到最新版的SDK：

[https://developer.umeng.com/sdk/ios?acm=lb-zebra-609113-9110428.1003.4.8840408&scm=1003.4.lb-zebra-609113-9110428.OTHER\_16059174649951\_8840408](https://developer.umeng.com/sdk/ios?acm=lb-zebra-609113-9110428.1003.4.8840408&scm=1003.4.lb-zebra-609113-9110428.OTHER_16059174649951_8840408)

需要注意，如果采用了手动的方式引入SDK，需要在工程的Targets->BuildSettings->Other Linker Flags中添加-Objc参数，并且添加如下系统依赖库：

CoreTelephony.framework  
libz.tbd        
libsqlite.tbd    
SystemConfiguration.framework

      在工程中集成好了SDK后，我们还需要做一些简单的配置，首先需要在友盟后台创建应用程序，并在应用设置页面中获取到APP KEY参数，如下图所示：

![](https://oscimg.oschina.net/oscnet/up-b6f6001e2a6e35c3344f1b54bdc287ac4be.png)

这个APP KEY参数在进行SDK的初始化时会使用到。在友盟应用后台的Deeplink配置页面中，我们需要将应用的一些Scheme信息进行配置，如下图：

![](https://oscimg.oschina.net/oscnet/up-be56d66741b37187a7c98a4e64bf9f7c01f.png)

之后，我们便可以通过创建营销活动来使用智能超链的功能了，首先在友盟后台进行营销活动的创建，页面如下图：

![](https://oscimg.oschina.net/oscnet/up-c730b5bb01376300932d1749916c18962a2.png)

可以看到，在创建营销活动时，可以配置活动名称、活动描述、应用内跳转的路径以及相关参数，当用户通过此营销活动的链接唤醒应用程序时，我们可以根据这些参数来让应用程序定位到指令的页面，从而实现业务的无缝衔接。创建好了营销活动后，在此活动的详情页面即可获取到LinkID，在网页中我们使用此LinkID来唤醒应用。

## 三、Deeplink调试与参数接收

      创建好了营销活动也集成完成了SDK，下面我们可以尝试通过一个简单的HTML DEMO来调试智能超链。HTML示例代码如下：

```html
<!DOCTYPE html>
<html>
<head>
    <title>Demo</title>
    <meta charset="utf-8" />
    <script src="https://g.alicdn.com/jssdk/u-link/index.min.js"></script>
</head>
<body>
    <div>
        <h1>测试DeepLink跳转</h1>
        <button id="btn1">唤起 App</button>
    </div>
</body>

<script type="text/javascript">
    ULink.start({
        id: 'usr1ffnv829fu02h', /* 平台为每个应用分配的方案link ID，必填 */
        data: {
            custom:"customn1",
            custom2:"custom2"
        } /* 自定义参数，选填 */
    }).ready(function(ctx) { /* 初始化完成的回调函数 */
        document.getElementById('btn1').onclick = function(e){
          ctx.wakeup(); /* 用户点击某个按钮时启动app */
    };
});

</script>

<style type="text/css">
    div {
        text-align: center;
    }
    button {
        font-size: 60px;
    }
    h1 {
        font-size: 60px;
    }
</style>
</html>
```

如上代码所示，u-link是友盟智能超链在JavaScript端的SDK，调用ULink的start方法时，配置的对象id就是我们创建的营销活动的id，通过这种方式唤醒应用程序会自动将后台配置的参数传递过去，同时我们也可以在data中定义更多动态参数进行传递。

      对于Xcode工程，在应用启动是首先需要对U-Link的SDK进行初始化操作，如下：

```objectivec
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [UMConfigure initWithAppkey:@"602505af668f9e17b8aef059" channel:nil];
    return YES;
}
```

之后实现如下几个AppDelegate的回调来处理超链：

```objectivec
#import "AppDelegate.h"
#import <UMCommon/UMConfigure.h>
#import <UMCommon/MobClickLink.h>

@interface AppDelegate ()<MobClickLinkDelegate>

@end

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [UMConfigure initWithAppkey:@"602505af668f9e17b8aef059" channel:nil];
    return YES;
}

- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url {
    return [MobClickLink handleLinkURL:url delegate:self];
}

- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {
    return [MobClickLink handleLinkURL:url delegate:self];
}

- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray<id<UIUserActivityRestoring>> * _Nullable))restorationHandler {
    return [MobClickLink handleUniversalLink:userActivity delegate:self];
}

// 解析后的回调函数，这里可以拿到所有的参数与跳转路径信息
- (void)getLinkPath:(NSString *)path params:(NSDictionary *)params {
    NSLog(@"getLinkPath:%@, %@", path, params);
}


@end
```

需要注意，如果是新版Xcode创建的应用程序，需要在SceneDelegate类中实现上面的方法。

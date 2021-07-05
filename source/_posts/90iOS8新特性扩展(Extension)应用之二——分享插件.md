---
title: iOS8新特性扩展(Extension)应用之二——分享插件
date: 2015-07-30
categories: iOS逻辑初窥
tags: []
---
## iOS8新特性扩展(Extension)应用之二——分享插件

        在上一篇博客中，介绍了iOS8新特性扩展功能之一的Today功能：[http://my.oschina.net/u/2340880/blog/485533](http://my.oschina.net/u/2340880/blog/485533)，这里我们再介绍一下分享的扩展功能。

      在iOS8之前，除了一些主流的社交平台，例如苹果支持内容分享外，其他开发者的应用若要加入分享的功能，将会十分的复杂。在iOS8的新特性中，apple为我们准备了这样的扩展功能。

首先创建工程，在我们的工程中新建一个Target：

![](http://static.oschina.net/uploads/space/2015/0730/142100_pZm5_2340880.png)

之后，模板中会为我们创建一个controller类，这个controller用于控制我们的分享插件，里面内容：

```
@implementation ShareViewController
//这个函数用于判断分享内容的可用性，我们在其中获取分享的内容进行检查
- (BOOL)isContentValid {
    // Do validation of contentText and/or NSExtensionContext attachments here
    return YES;
}
//点击post按钮后出发的方法，我们可以在这里将分享的内容进行上传等操作
- (void)didSelectPost {
    // This is called after the user selects Post. Do the upload of contentText and/or NSExtensionContext attachments.
    
    // Inform the host that we're done, so it un-blocks its UI. Note: Alternatively you could call super's -didSelectPost, which will similarly complete the extension context.
    [self.extensionContext completeRequestReturningItems:@[] completionHandler:nil];
}
//这里用于设置分享插件的附件按钮
- (NSArray *)configurationItems {
    // To add configuration options via table cells at the bottom of the sheet, return an array of SLComposeSheetConfigurationItem here.
    return @[];
}

@end
```

除此之外，还有一些常用的属性：

\- (void)presentationAnimationDidFinish;

弹出视图动画结束后执行的方法

[@property](http://my.oschina.net/property) (readonly, NS\_NONATOMIC\_IOSONLY) NSString *contentText;

分享的内容文字

[@property](http://my.oschina.net/property) (copy, NS\_NONATOMIC\_IOSONLY) NSString *placeholder;

默认显示的提示文字

\- (void)didSelectCancel;

取消按钮执行的方法

我们在代码中如下添加后运行：

```
@implementation ShareViewController
-(NSString *)placeholder{
    return @"提示文字";
}
- (NSArray *)configurationItems {
    // To add configuration options via table cells at the bottom of the sheet, return an array of SLComposeSheetConfigurationItem here.
    SLComposeSheetConfigurationItem * item =[[SLComposeSheetConfigurationItem alloc]init];
    item.title=@"地点";
    item.value=@"城门";
    return @[item];
}
```

我们用系统的相册做测试，点击相片的分享按钮：

![](http://static.oschina.net/uploads/space/2015/0730/151324_NCmX_2340880.png)

点击MORE，添加我们的扩展插件。

![](http://static.oschina.net/uploads/space/2015/0730/151443_xqFp_2340880.png)

这时分享栏中多了一个我们的插件，点击效果如下：

![](http://static.oschina.net/uploads/space/2015/0730/151645_IsIy_2340880.png)

还有一点我们需要了解，在这个扩展的plist文件中，有这样一个键：NSExtensionAttributes，里面有一个NSExtensionActivationRule的字典，其中可以设置一些键值，对分享插件的属性进行控制。

![](http://static.oschina.net/uploads/space/2015/0730/152047_Y6qT_2340880.png)

这些键的写法在官方文档中的介绍如下：

![](http://static.oschina.net/uploads/space/2015/0730/152736_1KYh_2340880.png)

这些键的意义，文档中介绍的很清楚，我们可以根据需要进行设置。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

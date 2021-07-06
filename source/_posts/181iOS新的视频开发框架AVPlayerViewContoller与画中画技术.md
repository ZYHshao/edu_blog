---
title: iOS新的视频开发框架AVPlayerViewContoller与画中画技术
date: 2016-02-25
categories: iOS逻辑初窥
tags: []
---
## iOS新的视频开发框架AVPlayerViewContoller与画中画技术

### 一、引言

        前面有一篇博客探讨了iOS中视频播放的开发相关类和方法，那篇博客中主要讲解的是MeidaPlayer框架中的MPMoviePlayerController类和MPMoviePlayerViewController类。在iOS8中，iOS开发框架中引入了一个新的视频框架AVKit，其中提供了视频开发类AVPlayerViewController用于在应用中嵌入播放视频的控件。在iOS8中，这两个框架中的视频播放功能并无太大差异，基本都可以满足开发者的需求。iOS9系统后，iPad Air正式开始支持多任务与画中画的分屏功能，所谓画中画，即是用户可以将当前播放的视频缩小放在屏幕上同时进行其他应用程序的使用。这个革命性的功能将极大的方便用户的使用。于此同时，在iOS9中，MPMoviePlayerController与MPMoviePlayerViewController类也被完全易用，开发者使用AVPlayerViewController可以十分方便的实现视频播放的功能并在一些型号的iPad上集成画中画的功能。

### 二、AVPlayerViewController的使用与其中方法属性解析

        使用AVPlayerViewController首先需要引入两个框架，如下：

```
#import <AVKit/AVKit.h>
#import <AVFoundation/AVFoundation.h>
```

使用如下代码进行视频的播放：

```
    NSString * path = [[NSBundle mainBundle]pathForResource:@"iphone" ofType:@"mp4"];
    NSURL *url = [NSURL fileURLWithPath:path];
    AVPlayerViewController * play = [[AVPlayerViewController alloc]init];
    play.player = [[AVPlayer alloc]initWithURL:url];
    [self presentViewController:play animated:YES completion:nil];
```

运行工程，可以看到如下图所示的视频播放界面：

![](http://static.oschina.net/uploads/space/2016/0225/233200_WFeE_2340880.png)

AVPlayerViewController中还有如下属性和方法提供给开发者使用：

```
//是否显示视频播放控制控件
@property (nonatomic) BOOL showsPlaybackControls;
//设置视频播放界面的尺寸缩放选项
/*
可以设置的值及意义如下：
AVLayerVideoGravityResizeAspect   不进行比例缩放 以宽高中长的一边充满为基准
AVLayerVideoGravityResizeAspectFill 不进行比例缩放 以宽高中短的一边充满为基准
AVLayerVideoGravityResize     进行缩放充满屏幕
*/
@property (nonatomic, copy) NSString *videoGravity;
//获取是否已经准备好开始播放
@property (nonatomic, readonly, getter = isReadyForDisplay) BOOL readyForDisplay;
//获取视频播放界面的尺寸
@property (nonatomic, readonly) CGRect videoBounds;
//视频播放器的视图 自定义的控件可以添加在其上
@property (nonatomic, readonly, nullable) UIView *contentOverlayView;
//画中画代理 iOS9后可用
@property (nonatomic, weak, nullable) id <AVPlayerViewControllerDelegate> delegate NS_AVAILABLE_IOS(9_0);
//是否支持画中画 iOS9后可用 默认支持
@property (nonatomic) BOOL allowsPictureInPicturePlayback NS_AVAILABLE_IOS(9_0);
```

### 三、画中画编程技术应用

        AVPlayerViewController是默认支持画中画操作的，如上图所示，视频的播放界面右下角出现一个画中画的按钮，点击这个按钮当前播放的视频界面会缩小显示在屏幕角落，这时点击Home键回到主界面，或者切换到其他应用程序，视频播放不会中断。如下图所示：

![](http://static.oschina.net/uploads/space/2016/0225/234351_ETN2_2340880.png)

两指的捏合操作可以将缩小的视频播放窗口进行任意尺寸的放大，如果将视频窗口拖进屏幕的边界，视频窗口会被吸进边界，用户可以通过拖拽手势将其拉出，如下图：![](http://static.oschina.net/uploads/space/2016/0225/234744_bYsu_2340880.png)

AVPlayerViewControllerDelegate中的方法可以对用户画中画的操作进行监听：

```
//将要开始画中画时调用的方法
- (void)playerViewControllerWillStartPictureInPicture:(AVPlayerViewController *)playerViewController{
}
//已经开始画中画时调用的方法
- (void)playerViewControllerDidStartPictureInPicture:(AVPlayerViewController *)playerViewController{
}
//开始画中画失败调用的方法
- (void)playerViewController:(AVPlayerViewController *)playerViewController failedToStartPictureInPictureWithError:(NSError *)error{
}
//将要停止画中画时调用的方法
- (void)playerViewControllerWillStopPictureInPicture:(AVPlayerViewController *)playerViewController{
}
//已经停止画中画时调用的方法
- (void)playerViewControllerDidStopPictureInPicture:(AVPlayerViewController *)playerViewController{
}
//是否在开始画中画时自动将当前的播放界面dismiss掉 返回YES则自动dismiss 返回NO则不会自动dismiss
- (BOOL)playerViewControllerShouldAutomaticallyDismissAtPictureInPictureStart:(AVPlayerViewController *)playerViewController{
    return YES;
}
//用户点击还原按钮 从画中画模式还原回app内嵌模式时调用的方法
- (void)playerViewController:(AVPlayerViewController *)playerViewController restoreUserInterfaceForPictureInPictureStopWithCompletionHandler:(void (^)(BOOL restored))completionHandler{
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

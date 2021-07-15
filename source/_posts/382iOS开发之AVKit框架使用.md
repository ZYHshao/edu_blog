---
title: iOS开发之AVKit框架使用
date: 2018-10-11
categories: iOS逻辑初窥
tags: []
---
## iOS开发之AVKit框架使用

### 一、引言

    在iOS开发框架中，AVKit是一个非常上层，偏应用的框架，它是基于AVFoundation的一层视图层封装。其中相关文件和类都十分简单，本篇博客主要整理和总结AVKit中相关类的使用方法。

### 二、AVRoutePickerView

    AVRoutePickerView是iOS 11后新加入的类，AirPlay是iOS设备方便用户使用的一大特点。其作用是将当前手机播放的音频或者视频投送到其他外部设备上，例如支持AirPlay的电视，车载设备等。AVRoutePickerView只是一个按钮，其用来方便用户可以直接在应用程序内唤出AirPlay选择窗口。示例如下：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    AVRoutePickerView * view = [[AVRoutePickerView alloc]initWithFrame:CGRectMake(100, 100, 60, 60)];
    //活跃状态颜色
    view.activeTintColor = [UIColor redColor];
    //设置代理
    view.delegate = self;
    [self.view addSubview:view];
}
//AirPlay界面弹出时回调
- (void)routePickerViewWillBeginPresentingRoutes:(AVRoutePickerView *)routePickerView{
    NSLog(@"!!!!!!!!");
}
//AirPlay界面结束时回调
- (void)routePickerViewDidEndPresentingRoutes:(AVRoutePickerView *)routePickerView{
    NSLog(@"@@@@@@@@");
}
```

按钮和弹出界面效果如下：

![](https://oscimg.oschina.net/oscnet/c864c75a953fe04266f52c412aef9e81192.jpg)

![](https://oscimg.oschina.net/oscnet/942ba1b1c8f39228d484f399dcb779df185.jpg)

从上面的示例代码也可以看出，对于AVRoutePickerView，我们基本没有任何可以进行自定义的余地，从UI效果到按钮的触发方法全部由AVKit封装好了，它只是一个唤出系统功能的接口。

### 三、AVPlayerViewController

    AVPlayerViewController是对AVFoundation中的AVPlayer与AVPlayerLayer的封装，它是一个封装好的视图控制器，包含了视频的播放和控制功能。这个类在iOS8之后可用，解析如下：

```objectivec
@interface AVPlayerViewController : UIViewController
//视频播放器对象
@property (nonatomic, strong, nullable) AVPlayer *player;
//是否显示视频播放控制组件
@property (nonatomic) BOOL showsPlaybackControls;
//设置视频的填充方式
/*
//按比例缩放
AVF_EXPORT AVLayerVideoGravity const AVLayerVideoGravityResizeAspect NS_AVAILABLE(10_7, 4_0);
//按比例填充
AVF_EXPORT AVLayerVideoGravity const AVLayerVideoGravityResizeAspectFill NS_AVAILABLE(10_7, 4_0);
//充满
AVF_EXPORT AVLayerVideoGravity const AVLayerVideoGravityResize NS_AVAILABLE(10_7, 4_0);
*/
@property (nonatomic, copy) AVLayerVideoGravity videoGravity;
//视频的第一帧是否已经准备好了
@property (nonatomic, readonly, getter = isReadyForDisplay) BOOL readyForDisplay;
//获取视频的尺寸
@property (nonatomic, readonly) CGRect videoBounds;
//内容覆盖层 可以向其上添加子视图 会出现在视频层与控制层之间
@property (nonatomic, readonly, nullable) UIView *contentOverlayView;
//是否允许画中画  iOS9以上可用 ipad可用
@property (nonatomic) BOOL allowsPictureInPicturePlayback API_AVAILABLE(ios(9.0));
//是否对信息中心的播放器信息进行更新 默认为YES
@property (nonatomic) BOOL updatesNowPlayingInfoCenter API_AVAILABLE(ios(10.0));
//是否默认进行全屏播放
@property (nonatomic) BOOL entersFullScreenWhenPlaybackBegins API_AVAILABLE(ios(11.0));
//播放结束后 是否默认退出全屏
@property (nonatomic) BOOL exitsFullScreenWhenPlaybackEnds API_AVAILABLE(ios(11.0));
//代理
@property (nonatomic, weak, nullable) id <AVPlayerViewControllerDelegate> delegate API_AVAILABLE(ios(9.0));
@end
```

AVPlayerViewControllerDelegate解析如下：

```objectivec
//将要开始画中画时调用
- (void)playerViewControllerWillStartPictureInPicture:(AVPlayerViewController *)playerViewController;
//已经开始画中画时调用
- (void)playerViewControllerDidStartPictureInPicture:(AVPlayerViewController *)playerViewController;
//开启画中画失败调用
- (void)playerViewController:(AVPlayerViewController *)playerViewController failedToStartPictureInPictureWithError:(NSError *)error;
//将要结束画中画调用
- (void)playerViewControllerWillStopPictureInPicture:(AVPlayerViewController *)playerViewController;
//已经结束画中画调用
- (void)playerViewControllerDidStopPictureInPicture:(AVPlayerViewController *)playerViewController;
//是否自动关闭控制器当画中画开始时
- (BOOL)playerViewControllerShouldAutomaticallyDismissAtPictureInPictureStart:(AVPlayerViewController *)playerViewController;
//画中画结束后回复之前的用户界面
- (void)playerViewController:(AVPlayerViewController *)playerViewController restoreUserInterfaceForPictureInPictureStopWithCompletionHandler:(void (^)(BOOL restored))completionHandler;
```

### 四、AVPictureInPictureController

      AVPictureInPictureController是一个控制器，用来对画中画进行相关操作，解析如下：

```objectivec
@interface AVPictureInPictureController : NSObject
//获取当前设备是否支持画中画
+ (BOOL)isPictureInPictureSupported;
//画中画转换开始按钮图像
+ (UIImage *)pictureInPictureButtonStartImageCompatibleWithTraitCollection:(nullable UITraitCollection *)traitCollection;
//画中画转换结束按钮图像
+ (UIImage *)pictureInPictureButtonStopImageCompatibleWithTraitCollection:(nullable UITraitCollection *)traitCollection;
//构造方法
- (nullable instancetype)initWithPlayerLayer:(AVPlayerLayer *)playerLayer;
//播放器视图
@property (nonatomic, readonly) AVPlayerLayer *playerLayer;
//代理
@property (nonatomic, weak, nullable) id <AVPictureInPictureControllerDelegate> delegate;
//开始画中画
- (void)startPictureInPicture;
//结束画中画
- (void)stopPictureInPicture;
//画中画目前是否可用
@property (nonatomic, readonly, getter = isPictureInPicturePossible) BOOL pictureInPicturePossible;
//画中画是否激活
@property (nonatomic, readonly, getter = isPictureInPictureActive) BOOL pictureInPictureActive;
//是否支持画中画
@property (nonatomic, readonly, getter = isPictureInPictureSuspended) BOOL pictureInPictureSuspended;

```

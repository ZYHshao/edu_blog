---
title: iOS视频功能模块的开发
date: 2015-08-10
categories: iOS逻辑初窥
tags: []
---
## iOS视频功能模块的开发

### 一、使用MPMoviePlayerController进行视频播放

        MPMoviePlayerController是iOS中进行视频播放开发的一个控制类，里面涵盖了视频播放中大部分的需求功能，在使用这个框架时，需要导入头文件<MediaPlayer/MediaPlayer.h>。

#### 1、初始化方法

        MPMoviePlayerController可以播放网络视频，也可以播放本地视频，通过不同的URL来进行初始化，例如本地视频的初始化如下：

```
//视频文件路径
    NSString *path = [[NSBundle mainBundle] pathForResource:fileName ofType:@"mp4"];
    //视频URL
    NSURL *url = [NSURL fileURLWithPath:path];
    //视频播放对象
    MPMoviePlayerController * movie = [[MPMoviePlayerController alloc] initWithContentURL:url];
```

初始化和完成相关配置后，我们需要将MPMoviePlayerController对象的View添加在我们需要的UI视图上，这个控制器只提供的控制的相关功能，外部的UI并没有为我们提供好。

#### 2、相关属性与方法

[@property](http://my.oschina.net/property) (nonatomic, copy) NSURL *contentURL;

视频文件的url地址

[@property](http://my.oschina.net/property) (nonatomic, readonly) UIView *view;

播放器view，在使用之前，必须设置frame大小，然后将其添加在我们的UI视图上

[@property](http://my.oschina.net/property) (nonatomic, readonly) UIView *backgroundView;

播放器背景颜色

[@property](http://my.oschina.net/property) (nonatomic, readonly) MPMoviePlaybackState playbackState;

播放器的当前播放状态，枚举定义如下：

```
typedef NS_ENUM(NSInteger, MPMoviePlaybackState) {
    MPMoviePlaybackStateStopped,//停止播放
    MPMoviePlaybackStatePlaying,//正在播放
    MPMoviePlaybackStatePaused,//暂停播放
    MPMoviePlaybackStateInterrupted,//中断播放
    MPMoviePlaybackStateSeekingForward,//快进
    MPMoviePlaybackStateSeekingBackward//快退
};
```

[@property](http://my.oschina.net/property) (nonatomic, readonly) MPMovieLoadState loadState;

播放器的网络缓存状态，枚举定义如下：

```
typedef NS_OPTIONS(NSUInteger, MPMovieLoadState) {
    MPMovieLoadStateUnknown        = 0,//状态未知
    MPMovieLoadStatePlayable       = 1 << 0,//缓存数据足够开始播放，但是视频并没有缓存完全
    MPMovieLoadStatePlaythroughOK  = 1 << 1, //已经缓存完成，如果设置了自动播放，这时会自动播放
    MPMovieLoadStateStalled        = 1 << 2, //数据缓存已经停止，播放将暂停
};
```

@property (nonatomic) MPMovieControlStyle controlStyle;

播放器风格，枚举如下:

```
typedef NS_ENUM(NSInteger, MPMovieControlStyle) {
    MPMovieControlStyleNone,       // 无控制器
    MPMovieControlStyleEmbedded,   // 嵌入视频风格
    MPMovieControlStyleFullscreen, // 全屏播放风格
    
    MPMovieControlStyleDefault = MPMovieControlStyleEmbedded
};
```

@property (nonatomic) MPMovieRepeatMode repeatMode;

播放器的循环模式，枚举如下：

```
typedef NS_ENUM(NSInteger, MPMovieRepeatMode) {
    MPMovieRepeatModeNone,//播放结束后不循环
    MPMovieRepeatModeOne//循环
};
```

@property (nonatomic) BOOL shouldAutoplay;

是否开启自动播放

@property (nonatomic, getter=isFullscreen) BOOL fullscreen;

设置是否充满屏幕

\- (void)setFullscreen:(BOOL)fullscreen animated:(BOOL)animated;

设置是否充满屏幕，带动画效果

@property (nonatomic) MPMovieScalingMode scalingMode;

设置播放器的填充方式，枚举定义如下：

```
typedef NS_ENUM(NSInteger, MPMovieScalingMode) {
    MPMovieScalingModeNone,       // 无缩放
    MPMovieScalingModeAspectFit,  // 适应大小模式
    MPMovieScalingModeAspectFill, // 充满可视范围，可能会被裁剪
    MPMovieScalingModeFill        // 缩放到充满视图
};
```

@property (nonatomic, readonly) BOOL readyForDisplay NS\_AVAILABLE\_IOS(6_0);

返回YES说明数据栈已经缓存好数据，返回NO则没有缓存好

@property (nonatomic, readonly) MPMovieMediaTypeMask movieMediaTypes;

数据文件的格式，枚举如下：

```
typedef NS_OPTIONS(NSUInteger, MPMovieMediaTypeMask) {
    MPMovieMediaTypeMaskNone  = 0,//格式未知
    MPMovieMediaTypeMaskVideo = 1 << 0,//音频格式
    MPMovieMediaTypeMaskAudio = 1 << 1//视频格式
};
```

@property (nonatomic) MPMovieSourceType movieSourceType;

视频的数据类型，枚举如下：

```
typedef NS_ENUM(NSInteger, MPMovieSourceType) {
    MPMovieSourceTypeUnknown,//类型未知
    MPMovieSourceTypeFile,     // 文件类型
    MPMovieSourceTypeStreaming // 数据流
};
```

@property (nonatomic, readonly) NSTimeInterval duration;

视频文件的时长

@property (nonatomic, readonly) NSTimeInterval playableDuration;

缓存完成能够播放的时长

@property (nonatomic, readonly) CGSize naturalSize;

视频的原始大小

@property (nonatomic) NSTimeInterval initialPlaybackTime;

播放器开始播放的时间

@property (nonatomic) NSTimeInterval endPlaybackTime;

播放器结束播放的时间

@property (nonatomic) BOOL allowsAirPlay;

是否允许云端播放

\- (void)requestThumbnailImagesAtTimes:(NSArray *)playbackTimes timeOption:(MPMovieTimeOption)optio;

获取视频某一些时间点的缩略图，参数枚举如下，生成缩略图的数据回调在后面的通知中详说：

```
typedef NS_ENUM(NSInteger, MPMovieTimeOption) {
    MPMovieTimeOptionNearestKeyFrame,//使用最近的关键帧生成缩略图
    MPMovieTimeOptionExact//使用精确的当前帧生成缩略图
};
```

与播放控制相关的方法如下：

```
//调用这个方法进行播放视频的准备工作
- (void)prepareToPlay;
//获取播放器的准备工作是否就绪
@property(nonatomic, readonly) BOOL isPreparedToPlay;
//调用此方法进行视频的播放
- (void)play;
//调用此方法进行视频播放的暂停操作
- (void)pause;
//调用此方法停止视频播放
- (void)stop;
//当前视频已播放的时间
@property(nonatomic) NSTimeInterval currentPlaybackTime;
//当前视频的播放速度
@property(nonatomic) float currentPlaybackRate;
//调用此方法进行快进操作
- (void)beginSeekingForward;
//调用此方法进行快退操作
- (void)beginSeekingBackward;
//调用此方法结束快进或者快退操作
- (void)endSeeking;
```

#### 3、系统相关通知

        MPMoviePlayerController的系统回调并没有采用代理的设计模式，而是采用的系统发通知，我们注册观察者，接收我们需要的通知。举例几种常用通知如下：

NSString \* const MPMoviePlayerScalingModeDidChangeNotification;

播放器缩放产生改变时发送的通知

NSString \* const MPMoviePlayerPlaybackDidFinishNotification;

播放结束时发送的通知

NSString \* const MPMoviePlayerPlaybackStateDidChangeNotification;

播放状态改变时发送的通知

NSString \* const MPMoviePlayerLoadStateDidChangeNotification;

缓冲状态改变时发送的通知

NSString \* const MPMoviePlayerNowPlayingMovieDidChangeNotification;

当前播放的视频改变时发送的通知

NSString \* const MPMoviePlayerWillEnterFullscreenNotification;

将要进入全屏模式时发送的通知

NSString \* const MPMoviePlayerDidEnterFullscreenNotification;

已经进入全屏时发送的通知

NSString \* const MPMoviePlayerWillExitFullscreenNotification;

将要退出全屏时发送的通知

NSString \* const MPMoviePlayerDidExitFullscreenNotification;

已经退出全屏时发送的通知

NSString \* const MPMoviePlayerThumbnailImageRequestDidFinishNotification;

获取缩略图完成时发送的通知

### 二、MPMoviePlayerViewController视频视图控制器

        如果你很熟悉MVC，你可能会觉得MPMoviePlayerController的设计模式非常蹩脚，强行要求你将控制器的视图分离出来加在另外的UI上，徒增的代码逻辑的混乱，那么你想的没错，MPMoviePlayerViewController可能就是为了解决这个问题。

        MPMoviePlayerViewController将视图封装在了一起，其中有一个成员对象是MPMoviePlayerController类型，类似C++中的has-a逻辑，我们只需要对MPMoviePlayerViewController进行的简单的初始化后，对其中MPMoviePlayerController进行其他配置，之后通过模态跳转切换控制器即可。

        方法如下：

\- (instancetype)initWithContentURL:(NSURL *)contentURL;

初始化方法，和上面类似

@property (nonatomic, readonly) MPMoviePlayerController *moviePlayer;

播放器对象

\- (void)presentMoviePlayerViewControllerAnimated:(MPMoviePlayerViewController *)moviePlayerViewController;

\- (void)dismissMoviePlayerViewControllerAnimated;

viewController的模态跳转方法，也可以通过导航push与pop

代码示例如下：

```
@interface ViewController2 ()
@property(nonatomic,strong)MPMoviePlayerController * movie;
@property(nonatomic,strong)MPMoviePlayerViewController * viewController;
@end

@implementation ViewController2

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    [self playMovie:@"111"];
}
-(void)playMovie:(NSString *)fileName{
    //视频文件路径
    NSString *path = [[NSBundle mainBundle] pathForResource:fileName ofType:@"mp4"];
    //视频URL
    NSURL *url = [NSURL fileURLWithPath:path];
    //视频播放对象
    _viewController = [[MPMoviePlayerViewController alloc]initWithContentURL:url];
    _movie=_viewController.moviePlayer;
    // 注册一个播放结束的通知
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(myMovieFinishedCallback:)
                                                 name:MPMoviePlayerPlaybackDidFinishNotification
                                               object:_movie];
    _movie.fullscreen=YES;
}
-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{
    [_movie play];
    [self presentViewController:_viewController animated:YES completion:nil];
}

-(void)myMovieFinishedCallback:(NSNotification*)notify
{
    //视频播放对象
    MPMoviePlayerController* theMovie = [notify object];
    //销毁播放通知
    [[NSNotificationCenter defaultCenter] removeObserver:self
                                                    name:MPMoviePlayerPlaybackDidFinishNotification
                                                  object:theMovie];
    [theMovie.view removeFromSuperview];
    
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

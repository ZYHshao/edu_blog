---
title: iOS录屏框架ReplayKit的应用总结
date: 2020-05-12
categories: iOS逻辑初窥
tags: []
---
## iOS录屏框架ReplayKit的应用总结

      ReplayKit是iOS自带的一个屏幕录制的框架，其支持应用程序对当前应用内页面进行录屏，并将最终的视频保存到系统相册中。ReplayKit在iOS 9之后引入，其接口简介，可以非常方便的为应用添加录屏功能。需要注意，在某些iOS 12系统上，开启录屏可能会失败(通常需要重启设备解决)。

      在ReplayKit框架中，有两个非常重要的类，分别是RPScreenRecorder类与RPPreviewViewController类。RPScreenRecorder是录屏核心功能类，RPPreviewViewController是录屏结束后的预览控制器类。

      下面，列举了一个简单的录屏示例代码：

```objectivec
@interface ViewController () <RPScreenRecorderDelegate, RPPreviewViewControllerDelegate>

@end

@implementation ViewController

- (void)viewDidLoad {
    self.view.backgroundColor = [UIColor whiteColor];
    [super viewDidLoad];
    // 获取录屏功能是否可用
    NSLog(@"%d", [RPScreenRecorder sharedRecorder].available);
    // 设置录屏代理
    [RPScreenRecorder sharedRecorder].delegate = self;
    
    UIButton *btn = [UIButton buttonWithType:UIButtonTypeSystem];
    [btn setTitle:@"Start" forState:UIControlStateNormal];
    [btn addTarget:self action:@selector(start) forControlEvents:UIControlEventTouchUpInside];
    btn.frame = CGRectMake(100, 100, 100, 30);
    [self.view addSubview:btn];
    
    UIButton *btn2 = [UIButton buttonWithType:UIButtonTypeSystem];
    [btn2 setTitle:@"End" forState:UIControlStateNormal];
    [btn2 addTarget:self action:@selector(stop) forControlEvents:UIControlEventTouchUpInside];
    btn2.frame = CGRectMake(100, 150, 100, 30);
    [self.view addSubview:btn2];
}

- (void)stop {
    // 结束录屏 
    [[RPScreenRecorder sharedRecorder] stopRecordingWithHandler:^(RPPreviewViewController * _Nullable previewViewController, NSError * _Nullable error) {
        NSLog(@"stopRecordingWithHandler");
        if (!error) {
            previewViewController.previewControllerDelegate = self;
            // 到视频预览控制器
            [self presentViewController:previewViewController animated:YES completion:nil];
        }
    }];
}

// 开始录屏
- (void)start {
    [[RPScreenRecorder sharedRecorder] startRecordingWithHandler:^(NSError * _Nullable error) {
        NSLog(@"startRecordingWithHandlerError:%@", error);
    }];
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    self.view.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
}

// 录屏结束的回调
- (void)screenRecorder:(RPScreenRecorder *)screenRecorder didStopRecordingWithPreviewViewController:(nullable RPPreviewViewController *)previewViewController error:(nullable NSError *)error {
    NSLog(@"didStopRecordingWithPreviewViewControllerError:%@", error);
}

// 录屏可用性改变的回调
- (void)screenRecorderDidChangeAvailability:(RPScreenRecorder *)screenRecorder {
    NSLog(@"screenRecorderDidChangeAvailability:%d", screenRecorder.available);
}

// 录屏预览控制器的代理回调
- (void)previewControllerDidFinish:(RPPreviewViewController *)previewController {
    [previewController dismissViewControllerAnimated:YES completion:nil];
}

- (void)previewController:(RPPreviewViewController *)previewController didFinishWithActivityTypes:(NSSet <NSString *> *)activityTypes {
    NSLog(@"didFinishWithActivityTypes %@", activityTypes);
}

@end
```

其中，RPScreenRecorder类中提供了丰富的接口可以使用，列举如下：

```objectivec
@interface RPScreenRecorder : NSObject
// 获取单例
+ (RPScreenRecorder *)sharedRecorder;
// iOS 10 之前使用，可以通知是否开启麦克风
- (void)startRecordingWithMicrophoneEnabled:(BOOL)microphoneEnabled handler:(nullable void(^)(NSError * _Nullable error))handler;
// iOS 10 之后使用 开启录屏
- (void)startRecordingWithHandler:(nullable void(^)(NSError * _Nullable error))handler;
// 结束录屏
- (void)stopRecordingWithHandler:(nullable void(^)(RPPreviewViewController * _Nullable previewViewController, NSError * _Nullable error))handler;
// 放弃录屏
- (void)discardRecordingWithHandler:(void(^)(void))handler;
// 开启录屏，可以获取到视频流数据 iOS 11后可用
- (void)startCaptureWithHandler:(nullable void(^)(CMSampleBufferRef sampleBuffer, RPSampleBufferType bufferType, NSError * _Nullable error))captureHandler completionHandler:(nullable void(^)(NSError * _Nullable error))completionHandler;
// 结束视频流捕获
- (void)stopCaptureWithHandler:(nullable void(^)(NSError * _Nullable error))handler;
// 设置代理对象
@property (nonatomic, weak, nullable) id<RPScreenRecorderDelegate> delegate;
// 录屏功能是否可用
@property (nonatomic, readonly, getter=isAvailable) BOOL available;
// 是否正在录制中
@property (nonatomic, readonly, getter=isRecording) BOOL recording;
// 是否使用麦克风
@property (nonatomic, getter=isMicrophoneEnabled) BOOL microphoneEnabled;
// 是否可以使用相机
@property (nonatomic, getter=isCameraEnabled) BOOL cameraEnabled;
// 设置摄像头模式  前置/后置
@property (nonatomic) RPCameraPosition cameraPosition;
// 相机预览视图
@property (nonatomic, readonly, nullable) UIView *cameraPreviewView;
@end
```

在录屏的时候，也支持调用系统的麦克风和摄像头共同完成录制。RPScreenRecorderDelegate协议中定义了一些回调方法，如下：

```objectivec
// 停止录屏后的回调 iOS 10 之前使用
- (void)screenRecorder:(RPScreenRecorder *)screenRecorder didStopRecordingWithError:(NSError *)error previewViewController:(nullable RPPreviewViewController *)previewViewController;
// 停止录屏后的回调 iOS 10 之后使用
- (void)screenRecorder:(RPScreenRecorder *)screenRecorder didStopRecordingWithPreviewViewController:(nullable RPPreviewViewController *)previewViewController error:(nullable NSError *)error;
// 录屏权限更改的回调
- (void)screenRecorderDidChangeAvailability:(RPScreenRecorder *)screenRecorder;
```

RPPreviewViewController类是视频预览控制器类，这个控制器没有暴露太多的属性给开发者使用，其中预览模式可以选择分享模式或编辑模式，除此之外，其中还提供了一个代理协议给开发者进行使用，用来对用户的操作进行处理，如下：

```objectivec
@protocol RPPreviewViewControllerDelegate <NSObject>
@optional
// 预览结束
- (void)previewControllerDidFinish:(RPPreviewViewController *)previewController;
// 用户行为回调
- (void)previewController:(RPPreviewViewController *)previewController didFinishWithActivityTypes:(NSSet <NSString *> *)activityTypes;
@end
```

---
title: iOS高质量的动画实现解决方案——Lottie
date: 2018-02-11
categories: iOS逻辑初窥
tags: []
---
## iOS高质量的动画实现解决方案——Lottie

    真心的认为Lottie是一款十分优秀且实用的动画开发库，不只对于iOS和android原生开发者来说其让复杂动画的实现几乎没有成本，对于设计师来说，它的所见即所得，不需导出帧图像等优势也十分明显。本篇博客主要以iOS平台为例，简单介绍和总结Lottie动画库的使用方式。

### 一、几个有用链接

Lottie官网：[https://airbnb.design/lottie/](https://airbnb.design/lottie/)。

LottieFiles：[https://www.lottiefiles.com/](https://www.lottiefiles.com/)。

LottieFiles是一个在线的测试Lottie动画的网站，并且其上面也提供了许多常用的Lottie动画组件。

### 二、一个简单的小Demo

    先来看一个简单的小例子，我在LottieFiles上找了一个骑行动画的JSON文件，此文件的下载地址如下：

[https://www.lottiefiles.com/download/1385](https://www.lottiefiles.com/download/1385)

这是一个比较炫酷的骑行动画，试想一下，如果使用GIF或帧动画来实现，需要素材的大小可能要远远超过136k。将下载的JSON文件添加到iOS项目中，之后就像使用图片一样的来使用它即可，代码如下：

```objectivec
#import <Lottie/Lottie.h>
@interface ViewController ()

@property(nonatomic,strong)LOTAnimationView * animationView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.animationView = [LOTAnimationView animationNamed:@"go.json"];
    [self.view addSubview:self.animationView];
    self.animationView.frame = CGRectMake(20, 30, 280, 200);
    self.animationView.loopAnimation = YES;
    [self.animationView play];
}

@end
```

动画效果如下图：

![](https://static.oschina.net/uploads/space/2018/0211/111000_YbZ2_2340880.gif)

无论是从流畅度还是性能上，动画效果都要比GIF图片好许多。

### 三、对Lottie库的应用解析

    首先LOTAnimationView类是显示Lottie动画的视图类，从源代码中看它是继承自LOTView，不要慌，这个LOTView并不是什么稀奇古怪的类，它其实就是为了代码统一，是UIView或NSView的别名而已。  如果你将动画直接拖入到主工程下面，那么可以直接使用动画JSON文件名来进行动画的创建，方法如下：

```objectivec
//直接从mainBundle中加载素材
+ (nonnull instancetype)animationNamed:(nonnull NSString *)animationName NS_SWIFT_NAME(init(name:));
```

你也可以从自定义的Bundle或者使用其他方式来加载JSON文件：

```objectivec
//从自定义的Bundle加载动画
+ (nonnull instancetype)animationNamed:(nonnull NSString *)animationName inBundle:(nonnull NSBundle *)bundle NS_SWIFT_NAME(init(name:bundle:));
//直接从JSON字典加载动画
+ (nonnull instancetype)animationFromJSON:(nonnull NSDictionary *)animationJSON NS_SWIFT_NAME(init(json:));
//直接通过JSON文件加载动画
+ (nonnull instancetype)animationWithFilePath:(nonnull NSString *)filePath NS_SWIFT_NAME(init(filePath:));
+ (nonnull instancetype)animationFromJSON:(nullable NSDictionary *)animationJSON inBundle:(nullable NSBundle *)bundle NS_SWIFT_NAME(init(json:bundle:));
//从URL加载
- (nonnull instancetype)initWithContentsOfURL:(nonnull NSURL *)url;

```

其实无论上面哪种方式加载动画，都是通过LOTComposition组件类实例化的，你也可以直接通过这个类来构建动画视图：

```objectivec
//常用的构造方法
+ (nullable instancetype)animationNamed:(nonnull NSString *)animationName NS_SWIFT_NAME(init(name:));
+ (nullable instancetype)animationNamed:(nonnull NSString *)animationName
                              inBundle:(nonnull NSBundle *)bundle NS_SWIFT_NAME(init(name:bundle:));
+ (nullable instancetype)animationWithFilePath:(nonnull NSString *)filePath NS_SWIFT_NAME(init(filePath:));
+ (nonnull instancetype)animationFromJSON:(nonnull NSDictionary *)animationJSON NS_SWIFT_NAME(init(json:));
+ (nonnull instancetype)animationFromJSON:(nullable NSDictionary *)animationJSON
                                 inBundle:(nullable NSBundle *)bundle NS_SWIFT_NAME(init(json:bundle:));
- (instancetype _Nonnull)initWithJSON:(NSDictionary * _Nullable)jsonDictionary
                      withAssetBundle:(NSBundle * _Nullable)bundle;
```

JSON文件中包含的信息非常丰富，会与LOTComposition实例进行映射，例如动画的时长，起始帧和结束帧，宽高尺寸等。

    构造出LOTAnimationView实例后，需要调用方法进行动画的播放，下面列出了LOTAnimationView中的常用属性与方法：

```objectivec
//获取动画是否正在播放
@property (nonatomic, readonly) BOOL isAnimationPlaying;
//设置动画是否循环播放
@property (nonatomic, assign) BOOL loopAnimation;
//设置动画是否自动逆序播放
@property (nonatomic, assign) BOOL autoReverseAnimation;
//设置是否缓存
@property (nonatomic, assign) BOOL cacheEnable;
//动画完成的回调block
@property (nonatomic, copy, nullable) LOTAnimationCompletionBlock completionBlock;
//组件实例
@property (nonatomic, strong, nullable) LOTComposition *sceneModel;
//从指定的进度位置播放动画
- (void)playToProgress:(CGFloat)toProgress
        withCompletion:(nullable LOTAnimationCompletionBlock)completion;
//播放指定区间内的动画
- (void)playFromProgress:(CGFloat)fromStartProgress
              toProgress:(CGFloat)toEndProgress
          withCompletion:(nullable LOTAnimationCompletionBlock)completion;
//播放到动画的某一帧
- (void)playToFrame:(nonnull NSNumber *)toFrame
     withCompletion:(nullable LOTAnimationCompletionBlock)completion;
//播放指定帧区间内的动画
- (void)playFromFrame:(nonnull NSNumber *)fromStartFrame
              toFrame:(nonnull NSNumber *)toEndFrame
       withCompletion:(nullable LOTAnimationCompletionBlock)completion;
//播放动画 可以设置回调
- (void)playWithCompletion:(nullable LOTAnimationCompletionBlock)completion;
//直接播放动画
- (void)play;
//暂停播放
- (void)pause;
//停止播放
- (void)stop;
//设置当前帧
- (void)setProgressWithFrame:(nonnull NSNumber *)currentFrame;
//设置某一帧对应的动画属性值
- (void)setValue:(nonnull id)value
      forKeypath:(nonnull NSString *)keypath
         atFrame:(nullable NSNumber *)frame;
```

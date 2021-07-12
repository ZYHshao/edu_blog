---
title: iOS10语音识别框架SpeechFramework应用
date: 2016-09-25
categories: iOS10专题
tags: []
---
## iOS10语音识别框架SpeechFramework应用

### 一、引言

        iOS10系统是一个较有突破性的系统，其在Message，Notification等方面都开放了很多实用性的开发接口。本篇博客将主要探讨iOS10中新引入的SpeechFramework框架。有个这个框架，开发者可以十分容易的为自己的App添加语音识别功能，不需要再依赖于其他第三方的语音识别服务，并且，Apple的Siri应用的强大也证明了Apple的语音服务是足够强大的，不通过第三方，也大大增强了用户的安全性。

### 二、SpeechFramework框架中的重要类

        SpeechFramework框架比较轻量级，其中的类并不十分冗杂，在学习SpeechFramework框架前，我们需要对其中类与类与类之间的关系有个大致的熟悉了解。

SFSpeechRecognizer：这个类是语音识别的操作类，用于语音识别用户权限的申请，语言环境的设置，语音模式的设置以及向Apple服务发送语音识别的请求。

SFSpeechRecognitionTask：这个类是语音识别服务请求任务类，每一个语音识别请求都可以抽象为一个SFSpeechRecognitionTask实例，其中SFSpeechRecognitionTaskDelegate协议中约定了许多请求任务过程中的监听方法。

SFSpeechRecognitionRequest:语音识别请求类，需要通过其子类来进行实例化。

SFSpeechURLRecognitionRequest：通过音频URL来创建语音识别请求。

SFSpeechAudioBufferRecognitionRequest:通过音频流来创建语音识别请求。

SFSpeechRecognitionResult：语音识别请求结果类。

SFTranscription：语音转换后的信息类。

SFTranscriptionSegment：语音转换中的音频节点类。

        了解了上述类的作用于其之间的联系，使用SpeechFramework框架将十分容易。

### 三、申请用户语音识别权限与进行语音识别请求

        开发者若要在自己的App中使用语音识别功能，需要获取用户的同意。首先需要在工程的Info.plist文件中添加一个Privacy-Speech Recognition Usage Description键，其实需要对应一个String类型的值，这个值将会在系统获取权限的警告框中显示，Info.plist文件如下图所示：

![](https://static.oschina.net/uploads/space/2016/0925/194243_bTmq_2340880.png)

使用SFSpeechRecognize类的requestAuthorization方法来进行用户权限的申请，用户的反馈结果会在这个方法的回调block中传入，如下：

```objectivec
  //申请用户语音识别权限
  [SFSpeechRecognizer requestAuthorization:^(SFSpeechRecognizerAuthorizationStatus status) {     
  }];
```

SFSpeechRecognizerAuthorzationStatus枚举中定义了用户的反馈结果，如下：

```objectivec
typedef NS_ENUM(NSInteger, SFSpeechRecognizerAuthorizationStatus) {
    //结果未知 用户尚未进行选择
    SFSpeechRecognizerAuthorizationStatusNotDetermined,
    //用户拒绝授权语音识别
    SFSpeechRecognizerAuthorizationStatusDenied,
    //设备不支持语音识别功能
    SFSpeechRecognizerAuthorizationStatusRestricted,
    //用户授权语音识别
    SFSpeechRecognizerAuthorizationStatusAuthorized,
};
```

        如果申请用户语音识别权限成功，开发者可以通过SFSpeechRecognizer操作类来进行语音识别请求，示例如下：

```objectivec
    //创建语音识别操作类对象
    SFSpeechRecognizer * rec = [[SFSpeechRecognizer alloc]init];
    //通过一个音频路径创建音频识别请求
    SFSpeechRecognitionRequest * request = [[SFSpeechURLRecognitionRequest alloc]initWithURL:[[NSBundle mainBundle] URLForResource:@"7011" withExtension:@"m4a"]];
    //进行请求
    [rec recognitionTaskWithRequest:request resultHandler:^(SFSpeechRecognitionResult * _Nullable result, NSError * _Nullable error) {
        //打印语音识别的结果字符串
        NSLog(@"%@",result.bestTranscription.formattedString);
    }];
```

### 四、深入SFSpeechRecognizer类

        SFSpeechRecognizer类的主要作用是申请权限，配置参数与进行语音识别请求。其中比较重要的属性与方法如下：

```objectivec
//获取当前用户权限状态
+ (SFSpeechRecognizerAuthorizationStatus)authorizationStatus;
//申请语音识别用户权限
+ (void)requestAuthorization:(void(^)(SFSpeechRecognizerAuthorizationStatus status))handler;
//获取所支持的所有语言环境
+ (NSSet<NSLocale *> *)supportedLocales;
//初始化方法 需要注意 这个初始化方法将默认以设备当前的语言环境作为语音识别的语言环境
- (nullable instancetype)init;
//初始化方法 设置一个特定的语言环境
- (nullable instancetype)initWithLocale:(NSLocale *)locale NS_DESIGNATED_INITIALIZER;
//语音识别是否可用
@property (nonatomic, readonly, getter=isAvailable) BOOL available;
//语音识别操作类协议代理
@property (nonatomic, weak) id<SFSpeechRecognizerDelegate> delegate;
//设置语音识别的配置参数 需要注意 在每个语音识别请求中也有这样一个属性 这里设置将作为默认值
//如果SFSpeechRecognitionRequest对象中也进行了设置 则会覆盖这里的值
/*
typedef NS_ENUM(NSInteger, SFSpeechRecognitionTaskHint) {
    SFSpeechRecognitionTaskHintUnspecified = 0,     // 无定义
    SFSpeechRecognitionTaskHintDictation = 1,       // 正常的听写风格
    SFSpeechRecognitionTaskHintSearch = 2,          // 搜索风格
    SFSpeechRecognitionTaskHintConfirmation = 3,    // 短语风格
};
*/
@property (nonatomic) SFSpeechRecognitionTaskHint defaultTaskHint;
//使用回调Block的方式进行语音识别请求 请求结果会在Block中传入
- (SFSpeechRecognitionTask *)recognitionTaskWithRequest:(SFSpeechRecognitionRequest *)request
                                          resultHandler:(void (^)(SFSpeechRecognitionResult * __nullable result, NSError * __nullable error))resultHandler;
//使用代理回调的方式进行语音识别请求
- (SFSpeechRecognitionTask *)recognitionTaskWithRequest:(SFSpeechRecognitionRequest *)request
                                               delegate:(id <SFSpeechRecognitionTaskDelegate>)delegate;
//设置请求所占用的任务队列
@property (nonatomic, strong) NSOperationQueue *queue;
```

SFSpeechRecognizerDelegate协议中只约定了一个方法，如下:

```objectivec
//当语音识别操作可用性发生改变时会被调用
- (void)speechRecognizer:(SFSpeechRecognizer *)speechRecognizer availabilityDidChange:(BOOL)available;

```

        通过Block回调的方式进行语音识别请求十分简单，如果使用代理回调的方式，开发者需要实现SFSpeechRecognitionTaskDelegate协议中的相关方法，如下：

```objectivec
//当开始检测音频源中的语音时首先调用此方法
- (void)speechRecognitionDidDetectSpeech:(SFSpeechRecognitionTask *)task;
//当识别出一条可用的信息后 会调用
/*
需要注意，apple的语音识别服务会根据提供的音频源识别出多个可能的结果 每有一条结果可用 都会调用此方法
*/
- (void)speechRecognitionTask:(SFSpeechRecognitionTask *)task didHypothesizeTranscription:(SFTranscription *)transcription;
//当识别完成所有可用的结果后调用
- (void)speechRecognitionTask:(SFSpeechRecognitionTask *)task didFinishRecognition:(SFSpeechRecognitionResult *)recognitionResult;
//当不再接受音频输入时调用 即开始处理语音识别任务时调用
- (void)speechRecognitionTaskFinishedReadingAudio:(SFSpeechRecognitionTask *)task;
//当语音识别任务被取消时调用
- (void)speechRecognitionTaskWasCancelled:(SFSpeechRecognitionTask *)task;
//语音识别任务完成时被调用
- (void)speechRecognitionTask:(SFSpeechRecognitionTask *)task didFinishSuccessfully:(BOOL)successfully;
```

        SFSpeechRecognitionTask类中封装了属性和方法如下：

```objectivec
//此任务的当前状态
/*
typedef NS_ENUM(NSInteger, SFSpeechRecognitionTaskState) {
    SFSpeechRecognitionTaskStateStarting = 0,       // 任务开始
    SFSpeechRecognitionTaskStateRunning = 1,        // 任务正在运行
    SFSpeechRecognitionTaskStateFinishing = 2,      // 不在进行音频读入 即将返回识别结果
    SFSpeechRecognitionTaskStateCanceling = 3,      // 任务取消
    SFSpeechRecognitionTaskStateCompleted = 4,      // 所有结果返回完成
};
*/
@property (nonatomic, readonly) SFSpeechRecognitionTaskState state;
//音频输入是否完成
@property (nonatomic, readonly, getter=isFinishing) BOOL finishing;
//手动完成音频输入 不再接收音频
- (void)finish;
//任务是否被取消
@property (nonatomic, readonly, getter=isCancelled) BOOL cancelled;
//手动取消任务
- (void)cancel;
```

        关于音频识别请求类，除了可以使用SFSpeechURLRecognitionRequest类来进行创建外，还可以使用SFSpeechAudioBufferRecognitionRequest类来进行创建：

```objectivec
@interface SFSpeechAudioBufferRecognitionRequest : SFSpeechRecognitionRequest

@property (nonatomic, readonly) AVAudioFormat *nativeAudioFormat;
//拼接音频流
- (void)appendAudioPCMBuffer:(AVAudioPCMBuffer *)audioPCMBuffer;
- (void)appendAudioSampleBuffer:(CMSampleBufferRef)sampleBuffer;
//完成输入
- (void)endAudio;

@end
```

### 五、语音识别结果类SFSpeechRecognitionResult

        SFSpeechRecognitionResult类是语音识别结果的封装，其中包含了许多套平行的识别信息，其每一份识别信息都有可信度属性来描述其准确程度。SFSpeechRecognitionResult类中属性如下：

```objectivec
//识别到的多套语音转换信息数组 其会按照准确度进行排序
@property (nonatomic, readonly, copy) NSArray<SFTranscription *> *transcriptions;
//准确性最高的识别实例
@property (nonatomic, readonly, copy) SFTranscription *bestTranscription;
//是否已经完成 如果YES 则所有所有识别信息都已经获取完成
@property (nonatomic, readonly, getter=isFinal) BOOL final;

```

SFSpeechRecognitionResult类只是语音识别结果的一个封装，真正的识别信息定义在SFTranscription类中，SFTranscription类中属性如下：

```objectivec
//完整的语音识别准换后的文本信息字符串
@property (nonatomic, readonly, copy) NSString *formattedString;
//语音识别节点数组
@property (nonatomic, readonly, copy) NSArray<SFTranscriptionSegment *> *segments;
```

当对一句完整的话进行识别时，Apple的语音识别服务实际上会把这句语音拆分成若干个音频节点，每个节点可能为一个单词，SFTranscription类中的segments属性就存放这些节点。SFTranscriptionSegment类中定义的属性如下：

```objectivec
//当前节点识别后的文本信息
@property (nonatomic, readonly, copy) NSString *substring;
//当前节点识别后的文本信息在整体识别语句中的位置
@property (nonatomic, readonly) NSRange substringRange;
//当前节点的音频时间戳
@property (nonatomic, readonly) NSTimeInterval timestamp;
//当前节点音频的持续时间
@property (nonatomic, readonly) NSTimeInterval duration;
//可信度/准确度 0-1之间
@property (nonatomic, readonly) float confidence;
//关于此节点的其他可能的识别结果 
@property (nonatomic, readonly) NSArray<NSString *> *alternativeSubstrings;
```

温馨提示：SpeechFramework框架在模拟器上运行会出现异常情况，无法进行语音识别请求。会报出kAFAssistantErrorDomain的错误，还望有知道解决方案的朋友，给些建议，Thanks。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

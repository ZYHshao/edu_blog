---
title: iOS MachineLearning 系列（13）—— 语音与音频相关的AI能力
date: 2023-05-08
categories: 从机器学习到AI
tags: []
---
# iOS MachineLearning 系列（13）—— 语音与音频相关的AI能力

在语音分析方面，iOS中提供了原生的Speech框架，这个框架可以实时的将语音解析成文本。这个能力非常强大，使用它我们可以实现类似实时翻译的功能。对于非语音的音频，也有一些原生的AI能力可以使用，例如分析语音的类型。SoundAnalysis框架能够识别300多种声音，我们也可以使用自己训练的模型来处理定制化的音频识别需求。

## 1 - 进行语音识别

使用Speech框架来进行语音识别非常简单，并且其支持多种语言。使用此功能前，我们需要先请求用户授予权限。在Info.plist文件中新增如下key:

```
NSSpeechRecognitionUsageDescription
```

此key设置的值为字符串文案，会在使用Speech框架时弹窗展示。

需要注意，Speech框架提供的并非是完全依赖本地的AI能力，其需要连接Apple的服务器来实现功能，因此在使用时要确保网络的正常。

首先需要定义个识别器对象，如下：

```swift
let recognizer = SFSpeechRecognizer(locale: Locale(identifier: "zh-Hants"))
```

其中locale参数设置所识别为的语言。

之后需要创建一个语音识别请求，并发起识别任务，如下：

```swift
// 这里使用本地的语音文件
let path = Bundle.main.path(forResource: "12168", ofType: ".wav")
let url = URL(fileURLWithPath: path!)
// 创建请求对象
let request = SFSpeechURLRecognitionRequest(url: url)

let label = UILabel(frame: CGRect(x: 0, y: 100, width: view.frame.width, height: 400))
label.numberOfLines = 0
view.addSubview(label)
// 发起请求任务
recognizer?.recognitionTask(with: request, resultHandler: { result, error in
    print(result?.bestTranscription.formattedString, error)
    label.text = result?.bestTranscription.formattedString
})
```

运行上面的代码，如果提供的音频文件是正常的语音文件，即可看到识别效果。上面的结果回调会根据音频的长度来多次回调，每次都会根据上下文进行之前识别结果的矫正。

Speech框架不仅支持语音文件的识别，也支持实时进行语音数据流的识别。只需要创建不同的Request类即可。我们可以先来看下语音分析请求的父类SFSpeechRecognitionRequest：

```swift
open class SFSpeechRecognitionRequest : NSObject {
    // 设置语音识别的类型
    open var taskHint: SFSpeechRecognitionTaskHint
    // 设置是否在解析过程中返回中间值，默认true，如果设置false则只有当整个语音文件解析完成再返回结果
    open var shouldReportPartialResults: Bool
    // 设置一组自定义的短语，这些短语可能不在词汇表中，针对场景的加强识别的准确性
    open var contextualStrings: [String]
    // 是否进行纯本地的解析，这种场景下不会发送语音到apple服务器，但是会降低准确性，某些语言不可用
    open var requiresOnDeviceRecognition: Bool
    // 设置识别结果中是否增加标点，iOS16之后可用
    open var addsPunctuation: Bool
}
```

其中taskHint属性用来设置识别类型，此枚举定义如下：

```swift
public enum SFSpeechRecognitionTaskHint : Int, @unchecked Sendable {
    case unspecified = 0 // 未指定明确类型
    case dictation = 1 // 一般的听写风格
    case search = 2 // 搜索请求风格
    case confirmation = 3 // 短语
}

```

要对语音文件进行分析，使用SFSpeechURLRecognitionRequest子类，如果要实时识别语音流，则使用SFSpeechAudioBufferRecognitionRequest子类即可，SFSpeechAudioBufferRecognitionRequest定义如下：

```swift
open class SFSpeechAudioBufferRecognitionRequest : SFSpeechRecognitionRequest {
    // 利于识别的音频格式
    open var nativeAudioFormat: AVAudioFormat { get }
    // 添加AVAudioPCMBuffer数据
    open func append(_ audioPCMBuffer: AVAudioPCMBuffer)
    // 添加CMSampleBuffer数据
    open func appendAudioSampleBuffer(_ sampleBuffer: CMSampleBuffer)
    // 调用此方法标识语音流数据添加完成
    open func endAudio()
}
```

SFSpeechRecognizer类用来发起语音识别请求，定义如下：

```swift
open class SFSpeechRecognizer : NSObject {
    // 所支持的语言
    open class func supportedLocales() -> Set<Locale>
    // 用户授权状态
    open class func authorizationStatus() -> SFSpeechRecognizerAuthorizationStatus
    // 请求用户授权
    open class func requestAuthorization(_ handler: @escaping (SFSpeechRecognizerAuthorizationStatus) -> Void)
    // 构造方法，使用当前系统语言
    public convenience init?()
    // 构造方法，设置语言
    public init?(locale: Locale)
    // 功能是否可用
    open var isAvailable: Bool { get }
    // 使用的语言
    open var locale: Locale { get }
    // 获取是否支持纯本地的请求
    open var supportsOnDeviceRecognition: Bool
    // 代理
    weak open var delegate: SFSpeechRecognizerDelegate?
    // 设置发起请求默认的类型
    open var defaultTaskHint: SFSpeechRecognitionTaskHint
    // 发起请求任务
    open func recognitionTask(with request: SFSpeechRecognitionRequest, resultHandler: @escaping (SFSpeechRecognitionResult?, Error?) -> Void) -> SFSpeechRecognitionTask
    open func recognitionTask(with request: SFSpeechRecognitionRequest, delegate: SFSpeechRecognitionTaskDelegate) -> SFSpeechRecognitionTask
    // 回调所使用的队列
    open var queue: OperationQueue
}

// SFSpeechRecognizerDelegate协议
public protocol SFSpeechRecognizerDelegate : NSObjectProtocol {
    // 可用性变化时回调
    optional func speechRecognizer(_ speechRecognizer: SFSpeechRecognizer, availabilityDidChange available: Bool)
}

```

如果使用闭包的方式来发起请求，则结果会在闭包回调中给到，如果采用代理的方式，则会通过代理回调返回。SFSpeechRecognitionTaskDelegate协议如下：

```swift
public protocol SFSpeechRecognitionTaskDelegate : NSObjectProtocol {
    // 首次检测到语音时调用
    optional func speechRecognitionDidDetectSpeech(_ task: SFSpeechRecognitionTask)
    // 每次有中间结果时调用
    optional func speechRecognitionTask(_ task: SFSpeechRecognitionTask, didHypothesizeTranscription transcription: SFTranscription)
    // 最终识别完成时调用
    optional func speechRecognitionTask(_ task: SFSpeechRecognitionTask, didFinishRecognition recognitionResult: SFSpeechRecognitionResult)
    // 识别任务结束后调用
    optional func speechRecognitionTaskFinishedReadingAudio(_ task: SFSpeechRecognitionTask)
    // 识别任务取消时调用
    optional func speechRecognitionTaskWasCancelled(_ task: SFSpeechRecognitionTask)
    // 识别任务完整成功后调用
    optional func speechRecognitionTask(_ task: SFSpeechRecognitionTask, didFinishSuccessfully successfully: Bool)
}
```

其中SFTranscription类用来描述识别中间结果，如下：

```swift
open class SFTranscription : NSObject, NSCopying, NSSecureCoding {
    // 格式化后的字符串
    open var formattedString: String { get }
    // 片段数组
    open var segments: [SFTranscriptionSegment] { get }
    // 语音速度，每秒单词数
    open var speakingRate: Double { get }
    // 单词间的平均停顿
    open var averagePauseDuration: TimeInterval { get }
}
```

其中SFTranscriptionSegment是具体的词组，里面封装了更多详细的信息：

```swift
open class SFTranscriptionSegment : NSObject, NSCopying, NSSecureCoding {
    // 词组
    open var substring: String { get }
    // 在原字符串中的位置
    open var substringRange: NSRange { get }
    // 在原音频中的时间点
    open var timestamp: TimeInterval { get }
    // 在音频中的时长
    open var duration: TimeInterval { get }
    // 测量的可信度，0到1，越大表示越可信
    open var confidence: Float { get }
    // 对此音频片段的更多可能结果
    open var alternativeSubstrings: [String] { get }
    // 发声特性对象
    open var voiceAnalytics: SFVoiceAnalytics? { get }
}
```

SFSpeechRecognitionResult类描述了分析的结果，实际上是一组SFTranscription的聚合。定义如下：

```swift
open class SFSpeechRecognitionResult : NSObject, NSCopying, NSSecureCoding {
    // 最完美的结果片段（如果分析结果，此为完整的）
    @NSCopying open var bestTranscription: SFTranscription { get }
    // 所有分析结果（根据可信度来排序）
    open var transcriptions: [SFTranscription] { get }
    // 是否分析结束
    open var isFinal: Bool { get }
    // 音频元数据信息
    open var speechRecognitionMetadata: SFSpeechRecognitionMetadata? { get }
}
```

SFSpeechRecognitionMetadata中封装了语音的基础数据：

```swift
open class SFSpeechRecognitionMetadata : NSObject, NSCopying, NSSecureCoding {
    // 语音速度，每分钟单词数
    open var speakingRate: Double { get }
    // 平均语速，每个单词平均秒数
    open var averagePauseDuration: TimeInterval { get }
    // 语音在音频中的开始时间
    open var speechStartTimestamp: TimeInterval { get }
    // 语音的持续时间
    open var speechDuration: TimeInterval { get }
    // 音频分析数据
    open var voiceAnalytics: SFVoiceAnalytics? { get }
}

open class SFVoiceAnalytics : NSObject, NSCopying, NSSecureCoding {
    // 人声稳定性
    @NSCopying open var jitter: SFAcousticFeature { get }
    @NSCopying open var shimmer: SFAcousticFeature { get }
    // 人声高低
    @NSCopying open var pitch: SFAcousticFeature { get }
    // 语音概率
    @NSCopying open var voicing: SFAcousticFeature { get }
}
```

发起语音请求后，会返回一个SFSpeechRecognitionTask对象，此任务对象可以对当次分析过程进行控制，如下：

```swift
open class SFSpeechRecognitionTask : NSObject {
    // 当前任务状态
    open var state: SFSpeechRecognitionTaskState { get }
    // 是否完成
    open var isFinishing: Bool { get }
    // 手动完成任务
    open func finish()
    // 是否取消
    open var isCancelled: Bool { get }
    // 手动取消任务
    open func cancel()
    // 异常数据
    open var error: Error? { get }
}

// 任务状态枚举定义如下
public enum SFSpeechRecognitionTaskState : Int, @unchecked Sendable {
    case starting = 0 // 开始
    case running = 1  // 运行中
    case finishing = 2 // 结束
    case canceling = 3 // 取消
    case completed = 4 //完成
}
```

## 2 - 音频类别识别

iOS内置API的音频分析能力可以方便的对音频进行分类，例如人声，乐器声等等。内置的SoundAnalysis框架能够分析识别300余种音效，当然其也支持使用自定义的模型来进行分析，本篇文章将只涉及到API的使用。

SNAudioFileAnalyzer类是音频分析的处理类，例如：

```swift
let analyzer = try! SNAudioFileAnalyzer(url: URL(fileURLWithPath: Bundle.main.path(forResource: "12168", ofType: ".wav")!))
```

SNAudioFileAnalyzer在实例化时会包装一个音频文件地址，后续将对此音频进行分析。首先需要创建一个分析请求，如下：

```swift
let request = try! SNClassifySoundRequest(classifierIdentifier: .version1)
```

其参数设置使用的算法版本。

通过如下方法来向SNAudioFileAnalyzer实例中添加一个请求，并设置请求过程的监听：

```swift
try! analyzer.add(request, withObserver: self)
```

之后调用analyze方法来触发请求的执行：

```swift
analyzer.analyze()
```

对请求过程的监听对象需要实现SNResultsObserving协议，如下：

```swift
public protocol SNResultsObserving : NSObjectProtocol {
    // 分析到请求结果时调用的回调，一段音频分析中会持续调用
    func request(_ request: SNRequest, didProduce result: SNResult)
    // 请求失败
    optional func request(_ request: SNRequest, didFailWithError error: Error)
    // 请求完成
    optional func requestDidComplete(_ request: SNRequest)
}
```

请求的结果数据为SNResult，这个是基础协议，真正将返回的对象是SNClassificationResult类型的，定义如下：

```swift
open class SNClassificationResult : NSObject, SNResult {
    // 分析出的类别
    open var classifications: [SNClassification] { get }
    // 在音频中的时间范围
    open var timeRange: CMTimeRange { get }
    // 返回指定标识符的类别
    open func classification(forIdentifier identifier: String) -> SNClassification?
}

open class SNClassification : NSObject {
    // 当前类别的标识符
    open var identifier: String { get }
    // 可信度
    open var confidence: Double { get }
}
```

SoundAnalysis框架本身比较简单，我们再来看下分析请求类，SNRequest是请求类的基类，为了便于后续framework的升级，我们使用SNClassifySoundRequest类创建请求：

```swift
open class SNClassifySoundRequest : NSObject, SNRequest {
    // 模型对固定大小的音频块进行分析时的重叠量，影响分析准确性
    open var overlapFactor: Double
    // 缓冲窗口的持续时间
    open var windowDuration: CMTime
    // 设置一组类别，分析结果将从此组中选择
    open var knownClassifications: [String] { get }
    // 使用自定义的模型进行分析
    public init(mlModel: MLModel)
    // 使用内置模型进行分析
    public init(classifierIdentifier: SNClassifierIdentifier)
}
```

最后Analyzer相关的类主要用来对请求进行控制，并决定要分析的音频。SoundAnalysis框架支持直接对音频文件进行分析，也支持对音频数据流进行分析，使用的类如下：

```swift
// 分析音频文件
open class SNAudioFileAnalyzer : NSObject {
    // 构造方法
    public init(url: URL) throws
    // 添加一个请求和对应的监听实例
    open func add(_ request: SNRequest, withObserver observer: SNResultsObserving) throws
    // 移除一个请求
    open func remove(_ request: SNRequest)
    // 移除所有请求
    open func removeAllRequests()
    // 开启同步分析（会阻塞线程）
    open func analyze()
    // 异步进行分析
    open func analyze(completionHandler: @escaping (Bool) -> Void)
    // 异步分析
    open func analyze() async -> Bool
    // 取消分析任务
    open func cancelAnalysis()
}

// 分析音频流
open class SNAudioStreamAnalyzer : NSObject {
    // 构造方法
    public init(format: AVAudioFormat)
    open func add(_ request: SNRequest, withObserver observer: SNResultsObserving) throws
    // 请求控制方法
    open func remove(_ request: SNRequest)
    open func removeAllRequests()
    // 对数据流做分析
    open func analyze(_ audioBuffer: AVAudioBuffer, atAudioFramePosition audioFramePosition: AVAudioFramePosition)
    // 完成数据流分析
    open func completeAnalysis()
}
```

完整的示例代码可以在如下地址找到：

[https://github.com/ZYHshao/MachineLearnDemo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FZYHshao%2FMachineLearnDemo)
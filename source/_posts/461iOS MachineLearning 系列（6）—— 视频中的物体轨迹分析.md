---
title: iOS MachineLearning 系列（6）—— 视频中的物体轨迹分析
date: 2023-04-27
categories: 从机器学习到AI
tags: []
---
# iOS MachineLearning 系列（6）—— 视频中的物体轨迹分析

轨迹分析是比物体追踪更上层的一种应用。Vision框架中提供了检测视频中多个物体的运动轨迹等能力，在健身，体育类应用中非常有用。

轨迹检测需要一系列的运动状态来分析，因此这类的请求是有状态的，有状态的请求可以被句柄多次调用，其会自动记录之前的状态，从而进行轨迹路径分析。需要注意，在进行轨迹检测时，要保证摄像机的相对静止，镜头的移动可能会影响检测的准确性。

在日常生活中，我们可以使用轨迹检测来进行投球的矫正，球类落点的推测等等。

## 1 - 解析视频中的物体飞行轨迹

轨迹检测需要保存状态，因此其传入的图像分析参数需要为包含CMTime信息的CMSampleBuffer数据。对于一个视频文件，我们首先要做的是将其中的图像帧解析出来，即获取到CMSampleBuffer数据。示例代码如下：

```swift
func detectTrajectories() {
    // 视频资源url
    let videoURL = URL(fileURLWithPath: Bundle.main.path(forResource: "video2", ofType: ".mov")!)
    // 读取视频资源
    let asset =  AVAsset(url: videoURL)
    guard let videoTrack = asset.tracks(withMediaType: .video).first else { return }
    // 获取帧率
    let frameRate = videoTrack.nominalFrameRate
    // 获取总时长
    let frameDuration = CMTime(seconds: 1 / Double(frameRate), preferredTimescale: CMTimeScale(NSEC_PER_SEC))
    // 解析参数
    let assetReaderOutputSettings: [String: Any] = [
        kCVPixelBufferPixelFormatTypeKey as String: kCVPixelFormatType_32BGRA
    ]
    // 解析输出类实例
    let assetReaderOutput = AVAssetReaderTrackOutput(track: videoTrack, outputSettings: assetReaderOutputSettings)
    // 创建视频reader实例
    let assetReader = try! AVAssetReader(asset: asset)
    // 添加输出对象
    assetReader.add(assetReaderOutput)
    // 开始解析
    if assetReader.startReading() {
        // 读取帧
        while let sampleBuffer = assetReaderOutput.copyNextSampleBuffer() {
            autoreleasepool {
                if CMSampleBufferDataIsReady(sampleBuffer) {
                    let timestamp = CMSampleBufferGetPresentationTimeStamp(sampleBuffer)
                    // 进行轨迹分析
                    processFrame(sampleBuffer, atTime: timestamp, withDuration:frameDuration)
                }
            }
        }
    }
}

```

processFram方法进行轨迹分析，实现如下：

```swift
func processFrame(_ sampleBuffer: CMSampleBuffer, atTime time : CMTime, withDuration duration : CMTime) {
    // 创建句柄
    let handler = VNImageRequestHandler(cmSampleBuffer: sampleBuffer, orientation: .up)
    // 发起分析请求
    try? handler.perform([request])
}

```

request对象的构建如下：

```swift
lazy var request: VNDetectTrajectoriesRequest = {
    let req = VNDetectTrajectoriesRequest(frameAnalysisSpacing:.zero, trajectoryLength: 10) { result, error in
        if let error {
            print(error)
        }
        self.handleResult(request: result as! VNDetectTrajectoriesRequest)
    }
    return req
}()

```

这里的参数后面会详细解释。

在示例中，我们可以添加一个AVPlayer来播放原视频，然后将分析出的轨迹绘制到视频对应的位置上进行对比。handleResult方法示例如下：

```swift
func handleResult(request: VNDetectTrajectoriesRequest) {
    for res in request.results ?? [] {
        // 校正后的轨迹点
        let points = res.projectedPoints
        for p in points {
            DispatchQueue.main.async {
                let v = UIView()
                // 视频宽高比
                let scale = self.image.size.width / self.image.size.height
                let width = self.view.frame.width
                let height = width / scale
                let size = CGSize(width: width, height:height)
                v.backgroundColor = .red
                // 播放器充满页面，居中播放视频的y轴偏移
                let offsetY = self.view.frame.height / 2 - height / 2
                v.frame = CGRect(x: p.x * size.width, y: (1 - p.y) * size.height + offsetY, width: 4, height: 4)
                self.view.addSubview(v)
            }
        }
    }
}

```

轨迹分析效果如下所示：

![](https://oscimg.oschina.net/oscnet/up-62e3ed3552ec82dca4d8567167da3eecd73.gif)

## 2 - VNDetectTrajectoriesRequest与VNTrajectoryObservation 类

VNDetectTrajectoriesRequest类一种有状态的分析请求类，继承自VNStatefulRequest，VNDetectTrajectoriesRequest定义如下：

```swift
open class VNDetectTrajectoriesRequest : VNStatefulRequest {
    // 构造方法
    // frameAnalysisSpacing参数设置采样间隔  
    // trajectoryLength设置确定一条轨迹的点数 最小为5
    public init(frameAnalysisSpacing: CMTime, trajectoryLength: Int, completionHandler: VNRequestCompletionHandler? = nil)
    // 轨迹点数
    open var trajectoryLength: Int { get }
    // 设置要检测的对象的最小半径
    open var objectMinimumNormalizedRadius: Float
    open var minimumObjectSize: Float

    // 设置要检测对象的最大半径
    open var objectMaximumNormalizedRadius: Float
    open var maximumObjectSize: Float
    
    // 检测的目标帧的时间
    open var targetFrameTime: CMTime

    // 分析结果
    open var results: [VNTrajectoryObservation]? { get }
}

```

VNTrajectoryObservatio类是轨迹分析的结果类，其内封装了组成轨迹的点。定义如下：

```swift
open class VNTrajectoryObservation : VNObservation {
    // 检测出的未处理前的原始点
    open var detectedPoints: [VNPoint] { get }
    // 矫正后的轨迹点
    open var projectedPoints: [VNPoint] { get }
    // 描述轨迹的抛物线方程
    open var equationCoefficients: simd_float3 { get }
    // 测量的物体的半径平均值
    open var movingAverageRadius: CGFloat { get }
}

```

其中equationCoefficients属性是模拟出的抛物线方程，即下面的公式：

> y = a*_x^2 + b_*x + c

simd_float3结构中会封装a，b和c的值。
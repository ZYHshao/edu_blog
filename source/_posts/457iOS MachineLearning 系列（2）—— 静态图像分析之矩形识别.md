---
title: iOS MachineLearning 系列（2）—— 静态图像分析之矩形识别
date: 2023-04-18
categories: 从机器学习到AI
tags: []
---
# iOS MachineLearning 系列（2）—— 静态图像分析之矩形识别

本系列文章将完整的介绍iOS中Machine Learning相关技术的应用。本篇文章开始，我们将先介绍一些与Machine Learning相关的API的应用。使用这些API可以快速方便的实现很多如图像识别，分析等复杂功能，且不会增加应用安装包的体积。

本篇将首先介绍如何分析出静态图片中的矩形区域。矩形区域的是被非常重要，其通常用来对要分析的图片进行预处理，例如通过矩形分析截取其中的二维码，条形码部分后再进行精准的识别。

## 1 - 矩形分析示例

与视觉相关的大部分AI能力都封装在Vision框架中，本文要介绍的是通过发起矩形分析请求来分析图片，得到分析结果后将分析出来的矩形区域绘制回原图像上。

首先定义一些属性：

```swift
// 要分析的图片资源
let image = UIImage(named: "image2")!
lazy var imageView = UIImageView(image: image)

// 绘制的矩形区域
var boxViews: [UIView] = []

// 图像分析请求句柄
lazy var imageRequestHandler = VNImageRequestHandler(cgImage: image.cgImage!,
                                                orientation: .up,
                                                options: [:])

// 图像分析请求实例
private lazy var rectangleDetectionRequest: VNDetectRectanglesRequest = {
    let rectDetectRequest = VNDetectRectanglesRequest { request, error in
        DispatchQueue.main.async {
            self.drawTask(request: request as! VNDetectRectanglesRequest)
        }
    }
    // 自定义一些配置项
    // 设置要分析的最大结果个数（矩形个数）
    rectDetectRequest.maximumObservations = 0
    // 设置最低接受的可信值
    rectDetectRequest.minimumConfidence = 0
    // 设置最小接受的纵横比
    rectDetectRequest.minimumAspectRatio = 0.1
    return rectDetectRequest
}()
```

其中VNDetectRectanglesRequest即是核心的图片分析请求类，VNImageRequestHandler是请求句柄，用来发起请求。后面我们会详细介绍。在开始请求分析之前，我们还需要定义个方法，用来进行矩形区域绘制：

```swift
private func drawTask(request: VNDetectRectanglesRequest) {
    // 将之前绘制的删除
    boxViews.forEach { v in
        v.removeFromSuperview()
    }
    // 遍历分析结果
    for result in request.results ?? [] {
        var box = result.boundingBox
        // 坐标系转换
        box.origin.y = 1 - box.origin.y - box.size.height
        let v = UIView()
        v.backgroundColor = .clear
        v.layer.borderColor = UIColor.black.cgColor
        v.layer.borderWidth = 1
        imageView.addSubview(v)
        let size = imageView.frame.size
        v.frame = CGRect(x: box.origin.x * size.width, y: box.origin.y * size.height, width: box.size.width * size.width, height: box.size.height * size.height)
    }
}
```

需要注意，Vision框架中的坐标系与CoreGraphics框架中的坐标系是一致的，其以左下角点为（0， 0）点，在UIKit框架中则是以左上角点为（0，0）点，记得进行坐标系的转换。

最后，使用下面的代码来发起请求，静态图像的分析将会是一个耗时的过程，因此建议在非主线程中进行：

```swift
DispatchQueue.global(qos: .userInitiated).async {
    do {
        // 发起分析请求
        try self.imageRequestHandler.perform([self.rectangleDetectionRequest])
    } catch let error as NSError {
        print("Failed to perform image request: \(error)")
        return
    }
}
```

分析的结果会在定义VNDetectRectanglesRequest时传入的回调中返回。

你可以用几张图片来实验下检测效果，如下图：

![](https://oscimg.oschina.net/oscnet/up-f53521d500087a848365283685172494d1a.png)

![](https://oscimg.oschina.net/oscnet/up-7cf831af13a5478f07b846830ce89e96057.png)

上面图片中的黑色边框就是我们检测出的结果绘制的。

## 2 - 关于VNDetectRectanglesRequest类

VNDetectRectanglesRequest类用来对核心的分析请求进行定义，并且设置结果回调。VNDetectRectanglesRequest类是专门创建矩形区域识别的请求类，继承自VNImageBasedRequest，VNImageBasedRequest类是静态图像分析请求的基类，继承自VNRequest类。

我们先来看VNRequest类：

```swift
@available(iOS 11.0, *)
open class VNRequest : NSObject, NSCopying {
    // 构造方法，无处理回调
    public convenience init()

    // 构造方法其中回调参数定义如下
    // (VNRequest, Error?) -> Void
    // VNRequest为当前实例本身 error是异常（如果有）
    public init(completionHandler: VNRequestCompletionHandler? = nil)

    // 是否开启后台线程模式，此模式会占用更少的内存，CPU，GPU资源，给用户更好的渲染体验，但是会以耗时为代价
    open var preferBackgroundProcessing: Bool

    // 是否允许使用GPU进行加速
    open var usesCPUOnly: Bool

    // 分析结果列表，VNObservation是结果基类，不同的子类实现不同的功能
    open var results: [VNObservation]? { get }
    
    // 处理回调，此回调中会传入当前Request对象，通过内部的results拿到结果
    open var completionHandler: VNRequestCompletionHandler? { get }
    
    // 进行分析的特定算法版本
    open var revision: Int
    // 所支持的算法版本集合
    open class var supportedRevisions: IndexSet { get }
    // 默认的版本
    open class var defaultRevision: Int { get }
    // 当前使用的算法版本
    open class var currentRevision: Int { get }
    // 取消分析请求
    open func cancel()
}
```

在VNRequest类中封装了一组VNObservation对象，当成功的完成了图像分析任务后，结果会被封装成VNObservation对象，不同的分析任务对应的结果对象也不同，VNObservation是这些结果的基类，其中封装了基础的信息，如下：

```swift
@available(iOS 11.0, *)
open class VNObservation : NSObject, NSCopying, NSSecureCoding, VNRequestRevisionProviding {
    // 唯一标识id
    open var uuid: UUID { get }

    // 此结果的可信度，取值0到1之间
    open var confidence: VNConfidence { get }

    // 此结果的有效时间
    @available(iOS 14.0, *)
    open var timeRange: CMTimeRange { get }
}
```

VNImageBasedReques类是VNRequest的一个子类，其是静态图片分析请求类的基类，其中只封装了一个属性：

```swift
@available(iOS 11.0, *)
open class VNImageBasedRequest : VNRequest {
    // 矩形被标准化处理后的尺寸，默认为{{ 0, 0 }, { 1, 1 }}
    open var regionOfInterest: CGRect
}
```

regionOfInterest属性非常有用，其默认会把我们要处理的图像标准化为单位矩形，返回的结果中的坐标是以此单位矩形为标准的。

最后，我们再来看下VNDetectRectanglesRequest类，这个类即使我们进行矩形区域识别的请求配置类，如下：

```swift
@available(iOS 11.0, *)
open class VNDetectRectanglesRequest : VNImageBasedRequest {
    // 设置检测接受的矩形最小的纵横比 VNAspectRatio是Float类型的别名，取值0-1之间
    open var minimumAspectRatio: VNAspectRatio
    
    // 设置检测所接受的最大的纵横比，取值0-1之间
    open var maximumAspectRatio: VNAspectRatio

    // 设置矩形角度可以偏离90度的最大角度，取值0-45之间
    open var quadratureTolerance: VNDegrees
    
    // 设置允许检测到的最小的矩形尺寸，设置为相对原图像比例值0-1之间
    open var minimumSize: Float
    
    // 设置能够接受的最小可信度，0到1之间，小于此可信度的检测结果不会被返回
    open var minimumConfidence: VNConfidence

    // 设置允许检测出的最多结果数，默认为1，设置为0表示不限制，但是Vision框架目前最多支持16
    open var maximumObservations: Int
    
    // 结果数组
    open var results: [VNRectangleObservation]? { get }
}
```

需要注意，设置最大最小纵横比时，会总是以长的一边作为纵，短的一边作为横。

## 3 - 关于VNRectangleObservation类

VNRectangleObservatio是矩形区域分析请求的结果类，继承自VNDetectedObjectObservation类，VNDetectedObjectObservation类是VNObservation的子类，其通常与对象的识别有关，其封装了与识别相关的属性，如下：

```swift
@available(iOS 11.0, *)
open class VNDetectedObjectObservation : VNObservation {
    // 检测出的区域，注意原点在左下角
    open var boundingBox: CGRect { get }
    // 缓冲区的图像数据
    open var globalSegmentationMask: VNPixelBufferObservation? { get }
}
```

VNRectangleObservation类则封装了与矩形相关的属性数据：

```swift
@available(iOS 11.0, *)
open class VNRectangleObservation : VNDetectedObjectObservation {
    // 左上角位置
    open var topLeft: CGPoint { get }
    // 右上角位置
    open var topRight: CGPoint { get }
    // 左下角位置
    open var bottomLeft: CGPoint { get }
    // 右下角位置
    open var bottomRight: CGPoint { get }
}
```

理解了请求配置类与分析结果类的用法，剩下的就是请求句柄了。

## 4 - 关于VNImageRequestHandler类

VNImageRequestHandler类是请求句柄类，更通俗的说，其为分析请求提供了图像数据源，并触发请求。其支持的构造方法如下：

```swift
@available(iOS 11.0, *)
open class VNImageRequestHandler : NSObject {
    // 构造方法
    public init(cvPixelBuffer pixelBuffer: CVPixelBuffer, options: [VNImageOption : Any] = [:])
    public init(cvPixelBuffer pixelBuffer: CVPixelBuffer, orientation: CGImagePropertyOrientation, options: [VNImageOption : Any] = [:])
    public init(cgImage image: CGImage, options: [VNImageOption : Any] = [:])
    public init(cgImage image: CGImage, orientation: CGImagePropertyOrientation, options: [VNImageOption : Any] = [:])
    public init(ciImage image: CIImage, options: [VNImageOption : Any] = [:])
    public init(ciImage image: CIImage, orientation: CGImagePropertyOrientation, options: [VNImageOption : Any] = [:])
    public init(url imageURL: URL, options: [VNImageOption : Any] = [:])
    public init(url imageURL: URL, orientation: CGImagePropertyOrientation, options: [VNImageOption : Any] = [:])
    public init(data imageData: Data, options: [VNImageOption : Any] = [:])
    public init(data imageData: Data, orientation: CGImagePropertyOrientation, options: [VNImageOption : Any] = [:])
    public init(cmSampleBuffer sampleBuffer: CMSampleBuffer, options: [VNImageOption : Any] = [:])
    public init(cmSampleBuffer sampleBuffer: CMSampleBuffer, orientation: CGImagePropertyOrientation, options: [VNImageOption : Any] = [:])
}
```

VNImageRequestHandler类的构造方法很多，但归根结底是要提供三部分内容：

1.  图片数据源。
2.  图片的方向。
3.  额外参数。

其中，图片的数据源可以从二进制数据加载，可以从网络加载，可以从CoreImage或CoreGraphics框架的图片对象加载等等，这里不多赘述。

图片的方向需要在构造句柄实例对象时进行提供，枚举如下：

```swift
@frozen public enum CGImagePropertyOrientation : UInt32, @unchecked Sendable {

    
    case up = 1 // 正向 

    case upMirrored = 2 // 水平镜像

    case down = 3 // 180度旋转

    case downMirrored = 4 // 竖直镜像

    case leftMirrored = 5 // 顺时针旋转90度后镜像

    case right = 6 // 顺时针旋转90度

    case rightMirrored = 7 // 逆时针旋转90度后镜像

    case left = 8 // 逆时针旋转90度
}
```

额外参数可以配置为一个字典对象，提供更多图片数据，支持配置的字段如下：

properties：此键可配置为一个属性字典，参考CGImageSourceCopyPropertiesAtIndex。

cameraIntrinsics：相机内部数据配置。

ciContex：CIContext配置。

最后，调用VNImageRequestHandler类的如下方法即可开始静态图像处理：

```swift
open func perform(_ requests: [VNRequest]) throws
```

同一个图像句柄可以同时发起多种图像处理请求。

注：本文所介绍的示例代码可在如下仓库获取：

[https://github.com/ZYHshao/MachineLearnDemo](https://github.com/ZYHshao/MachineLearnDemo)

> 专注技术，懂的热爱，愿意分享，做个朋友
> 
> QQ：316045346
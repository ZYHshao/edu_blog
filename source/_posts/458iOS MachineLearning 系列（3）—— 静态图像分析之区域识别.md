---
title: iOS MachineLearning 系列（3）—— 静态图像分析之区域识别
date: 2023-04-22
categories: 从机器学习到AI
tags: []
---
# iOS MachineLearning 系列（3）—— 静态图像分析之区域识别

本系列的前一篇文章介绍了如何使用iOS中自带的API对图片中的矩形区域进行分析。在图像静态分析方面，矩形区域分析是非常基础的部分。API还提供了更多面向应用的分析能力，如文本区域分析，条形码二维码的分析，人脸区域分析，人体分析等。本篇文章主要介绍这些分析API的应用。关于矩形识别的基础文章，链接如下：

[https://my.oschina.net/u/2340880/blog/8671152](https://my.oschina.net/u/2340880/blog/8671152)

## 1 - 文本区域分析

文本区域分析相比矩形区域分析更加上层，其API接口也更加简单。分析请求的创建示例如下：

```swift
private lazy var textDetectionRequest: VNDetectTextRectanglesRequest = {
    let textDetectRequest = VNDetectTextRectanglesRequest { request, error in
        DispatchQueue.main.async {
            self.drawTask(request: request as! VNDetectTextRectanglesRequest)
        }
    }
    // 是否报告字符边框区域
    textDetectRequest.reportCharacterBoxes = true
    return textDetectRequest
}()
```

其请求的发起方式，回调结果的处理与矩形分析一文中介绍的一致，这里就不再赘述。唯一不同的是，其分析的结果中新增了characterBoxes属性，用来获取每个字符的所在区域。

文本区域识别效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-3090f1a99c656a9d19cdd4931894a5b31e7.png)

## 2 - 条形码二维码识别

条形码和二维码在生活中非常常见，Vision框架中提供的API不仅支持条码区域的检测，还可以直接将条码的内容识别出来。

条码分析请求使用VNDetectBarcodesRequest类创建，如下：

```swift
open class VNDetectBarcodesRequest : VNImageBasedRequest {
    // 类属性，获取所支持的条码类型
    open class var supportedSymbologies: [VNBarcodeSymbology] { get }
    // 设置分析时要支持的条码类型
    open var symbologies: [VNBarcodeSymbology]
    // 结果列表
    open var results: [VNBarcodeObservation]? { get }
}
```

如果我们不对symbologies属性进行设置，则默认会尝试识别所有支持的类型。示例代码如下：

```swift
private lazy var barCodeDetectionRequest: VNDetectBarcodesRequest = {
    let barCodeDetectRequest = VNDetectBarcodesRequest {[weak self] request, error in
        guard let self else {return}
        DispatchQueue.main.async {
            self.drawTask(request: request as! VNDetectBarcodesRequest)
        }
    }
    barCodeDetectRequest.revision = VNDetectBarcodesRequestRevision1
    return barCodeDetectRequest
}()
```

需要注意，实测需要将分析所使用的算法版本revision设置为VNDetectBarcodesRequestRevision1。默认使用的版本可能无法分析出结果。

条码分析的结果类VNBarcodeObservation中会封装条码的相关数据，如下：

```swift
open class VNBarcodeObservation : VNRectangleObservation {
    // 当前条码的类型
    open var symbology: VNBarcodeSymbology { get }
    // 条码的描述对象，不同类型的条码会有不同的子类实现
    open var barcodeDescriptor: CIBarcodeDescriptor? { get }
    // 条码内容
    open var payloadStringValue: String? { get }
}
```

VNBarcodeObservation类也是继承自VNRectangleObservation类的，因此其也可以分析出条码所在的区域，需要注意，对于条形码来说其只能分析出条码的位置，对于二维码来说，其可以准确的识别出二维码的区域，如下图所示：

![](https://oscimg.oschina.net/oscnet/up-941384f0b46f09489de8a687de22b608419.png)

![](https://oscimg.oschina.net/oscnet/up-1b8ff45db111639e4d8e8b55d87f03e1756.png)

注：互联网上有很多可以生成条码的工具，例如：

[https://www.idcd.com/tool/barcode/encode](https://www.idcd.com/tool/barcode/encode)

## 3 - 轮廓检测

相比前面两种图像分析能力，轮廓检测的能力要更加复杂也更加强大一些。其可以通过图片的对比度差异来对内容轮廓进行分析。轮廓分析使用VNDetectContoursRequest类来创建请求。此类主要功能列举如下：

```swift
open class VNDetectContoursRequest : VNImageBasedRequest {
    // 轮廓检测时的对比度设置，取值0-3之间，此值越大，检测结果越精确（对于高对比度图片）
    open var contrastAdjustment: Float
    // 作为对比度分界的像素，取值0-1之间，默认0.5，会取居中值
    open var contrastPivot: NSNumber?
    // 设置检测时是否是检测暗色对象，默认为true，即认为背景色浅。设置为false则会在暗色图中检测明亮的对象轮廓
    open var detectsDarkOnLight: Bool
    // 设置检测图片时的缩放，轮廓检测会将图片进行压缩，此值取值范围为[64..NSUIntegerMax]，取最大值时表示使用原图
    open var maximumImageDimension: Int
    // 结果数组
    open var results: [VNContoursObservation]? { get }
}
```

其检测结果VNContoursObservation类中封装了轮廓的路径信息，在进行轮廓检测时，最外层的轮廓可能有很多内层轮廓组成，这些信息也封装在此类中。如下：

```swift
open class VNContoursObservation : VNObservation {
    // 内部轮廓个数
    open var contourCount: Int { get }
    // 获取指定的轮廓对象
    open func contour(at contourIndex: Int) throws -> VNContour
    // 顶级轮廓个数
    open var topLevelContourCount: Int { get }
    // 顶级轮廓数组
    open var topLevelContours: [VNContour] { get }
    // 根据indexPath获取轮廓对象
    open func contour(at indexPath: IndexPath) throws -> VNContour
    // 路径，会包含内部所有轮廓
    open var normalizedPath: CGPath { get }
}
```

需要注意，其返回的CGPath路径依然是以单位矩形为参照的，我们要将其绘制出来，需要对其进行转换，转换其实非常简单，现对其进行方法，并进行x轴方向的镜像反转，之后向下进行平移一个标准单位即可。示例如下：

```swift
private func drawTask(request: VNDetectContoursRequest) {
    boxViews.forEach { v in
        v.removeFromSuperview()
    }
    for result in request.results ?? [] {
        let oriPath = result.normalizedPath
        var transform = CGAffineTransform.identity.scaledBy(x: imageView.frame.width, y: -imageView.frame.height).translatedBy(x: 0, y: -1)
        let layer = CAShapeLayer()
        let path = oriPath.copy(using: &transform)
        layer.bounds = self.imageView.bounds
        layer.anchorPoint = CGPoint(x: 0, y: 0)
        imageView.layer.addSublayer(layer)
        layer.path = path
        layer.strokeColor = UIColor.blue.cgColor
        layer.backgroundColor = UIColor.white.cgColor
        layer.fillColor = UIColor.gray.cgColor
        layer.lineWidth = 1
    }
}
```

原图与绘制的轮廓图如下所示：

原图：

![](https://oscimg.oschina.net/oscnet/up-dc1398141a420ba81920ba62e3299f88319.png)

轮廓：

![](https://oscimg.oschina.net/oscnet/up-e9a75fb7f6b267b0270c7606b481b15c49e.png)

可以通过VNContoursObservation对象来获取其内的所有轮廓对象，VNContour定义如下：

```swift
open class VNContour : NSObject, NSCopying, VNRequestRevisionProviding {
    // indexPath
    open var indexPath: IndexPath { get }
    // 子轮廓个数
    open var childContourCount: Int { get }
    // 子轮廓对象数组
    open var childContours: [VNContour] { get }
    // 通过index获取子轮廓
    open func childContour(at childContourIndex: Int) throws -> VNContour
    // 描述轮廓的点数
    open var pointCount: Int { get }
    // 轮廓路径
    open var normalizedPath: CGPath { get }
    // 轮廓的纵横比
    open var aspectRatio: Float { get }
    // 简化的多边形轮廓，参数设置简化的阈值
    open func polygonApproximation(epsilon: Float) throws -> VNContour
}
```

理论上说，我们对所有的子轮廓进行绘制，也能得到一样的路径图像，例如：

```swift
private func drawTask(request: VNDetectContoursRequest) {
    boxViews.forEach { v in
        v.removeFromSuperview()
    }
    for result in request.results ?? [] {
        for i in 0 ..< result.contourCount {
            let contour = try! result.contour(at: i)
            var transform = CGAffineTransform.identity.scaledBy(x: imageView.frame.width, y: -imageView.frame.height).translatedBy(x: 0, y: -1)
            let layer = CAShapeLayer()
            let path = contour.normalizedPath.copy(using: &transform)
            layer.bounds = self.imageView.bounds
            layer.anchorPoint = CGPoint(x: 0, y: 0)
            imageView.layer.addSublayer(layer)
            layer.path = path
            layer.strokeColor = UIColor.blue.cgColor
            layer.backgroundColor = UIColor.clear.cgColor
            layer.fillColor = UIColor.clear.cgColor
            layer.lineWidth = 1
        }
    }
}
```

效果如下图：

![](https://oscimg.oschina.net/oscnet/up-90f52ae0cec78c332d643ff4924c4fe7fb0.png)

## 4 - 文档区域识别

文档识别可以分析出图片中的文本段落，使用VNDetectDocumentSegmentationRequest来创建分析请求，VNDetectDocumentSegmentationRequest没有额外特殊的属性，其分析结果为一组VNRectangleObservation对象，可以获取到文档所在的矩形区域。这里不再过多解说。

## 5 - 人脸区域识别

人脸识别在生活中也有着很广泛的应用，在进行人脸对比识别等高级处理前，我们通常需要将人脸的区域先提取出来，Vision框架中也提供了人脸区域识别的接口，使用VNDetectFaceRectanglesRequest类来创建请求即可。VNDetectFaceRectanglesRequest类本身比较加单，继承自VNImageBasedRequest类，无需进行额外的配置即可使用，其分析的结果为一组VNFaceObservation对象，分析效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-a57d29b95c0f2729d725bae6bfd4f509089.png)

VNFaceObservation类本身是继承自VNDetectedObjectObservation类的，因此我们可以直接获取到人脸的区域。VNFaceObservation中还有许多其他有用的信息：

```swift
open class VNFaceObservation : VNDetectedObjectObservation {
    // 面部特征对象
    open var landmarks: VNFaceLandmarks2D? { get }
    // 人脸在z轴的旋转度数，取值为-PI到PI之间
    open var roll: NSNumber? { get }
    // 人脸在y轴的旋转度数，取值为-PI/2到PI/2之间
    open var yaw: NSNumber? { get }
    // 人脸在x轴的旋转度数，取值为-PI/2到PI/2之间
    open var pitch: NSNumber? { get }
}
```

通过roll，yaw和pitch这3个属性，我们可以获取到人脸在空间中的角度相关信息。landmarks属性则比较复杂，其封装了人脸的特征点。并且VNDetectFaceRectanglesRequest请求是不会分析面部特征的，此属性会为nil，关于面部特征，我们后续介绍。

人脸特征分析请求使用VNDetectFaceLandmarksRequest创建，其返回的结果中会有landmarks数据，示例代码如下：

```swift
private func drawTask(request: VNDetectFaceLandmarksRequest) {
    boxViews.forEach { v in
        v.removeFromSuperview()
    }
    for result in request.results ?? [] {
        
        var box = result.boundingBox
        // 坐标系转换
        box.origin.y = 1 - box.origin.y - box.size.height
        let v = UIView()
        v.backgroundColor = .clear
        v.layer.borderColor = UIColor.red.cgColor
        v.layer.borderWidth = 2
        imageView.addSubview(v)
        let size = imageView.frame.size
        v.frame = CGRect(x: box.origin.x * size.width, y: box.origin.y * size.height, width: box.size.width * size.width, height: box.size.height * size.height)
        
        // 进行特征绘制
        let landmarks = result.landmarks
        // 拿到所有特征点
        let allPoints = landmarks?.allPoints?.normalizedPoints
        
        let faceRect = result.boundingBox
        // 进行绘制
        for point in allPoints ?? [] {
            //faceRect的宽高是个比例，我们对应转换成View上的人脸区域宽高
            let rectWidth = imageView.frame.width * faceRect.width
            let rectHeight = imageView.frame.height * faceRect.height
            // 进行坐标转换
            // 特征点的x坐标为人脸区域的比例，
            // 1. point.x * rectWidth 得到在人脸区域内的x位置
            // 2. + faceRect.minX * imageView.frame.width 得到在View上的x坐标
            // 3. point.y * rectHeight + faceRect.minY * imageView.frame.height获得Y坐标
            // 4. imageView.frame.height -  的作用是y坐标进行翻转
            let tempPoint = CGPoint(x: point.x * rectWidth + faceRect.minX * imageView.frame.width, y: imageView.frame.height - (point.y * rectHeight + faceRect.minY * imageView.frame.height))
            let subV = UIView()
            subV.backgroundColor = .red
            subV.frame = CGRect(x: tempPoint.x - 2, y: tempPoint.y - 2, width: 4, height: 4)
            imageView.addSubview(subV)
        }
    }
}
```

VNFaceLandmarks2D中封装了很多特征信息，上面的示例代码会将所有的特征点进行绘制，我们也可以根据需要取部分特征点：

```swift
open class VNFaceLandmarks2D : VNFaceLandmarks {
    // 所有特征点
    open var allPoints: VNFaceLandmarkRegion2D? { get }
    // 只包含面部轮廓的特征点
    open var faceContour: VNFaceLandmarkRegion2D? { get }
    // 左眼位置的特征点
    open var leftEye: VNFaceLandmarkRegion2D? { get }
    // 右眼位置的特征点
    open var rightEye: VNFaceLandmarkRegion2D? { get }
    // 左眉特征点
    open var leftEyebrow: VNFaceLandmarkRegion2D? { get }
    // 右眉特征点
    open var rightEyebrow: VNFaceLandmarkRegion2D? { get }
    // 鼻子特征点
    open var nose: VNFaceLandmarkRegion2D? { get }
    // 鼻尖特征点
    open var noseCrest: VNFaceLandmarkRegion2D? { get }
    // 中间特征点
    open var medianLine: VNFaceLandmarkRegion2D? { get }
    // 外唇特征点
    open var outerLips: VNFaceLandmarkRegion2D? { get }
    // 内唇特征点
    open var innerLips: VNFaceLandmarkRegion2D? { get }
    // 左瞳孔特征点
    open var leftPupil: VNFaceLandmarkRegion2D? { get }
    // 右瞳孔特征点
    open var rightPupil: VNFaceLandmarkRegion2D? { get }
}
```

VNFaceLandmarkRegion2D类中具体封装了特征点位置信息，需要注意，特征点的坐标是相对人脸区域的比例值，要进行转换。

**主要提示：特征检测在模拟器上可能不能正常工作，可以使用真机测试。**

默认人脸特征分析会返回76个特征点，我们可以通过设置VNDetectFaceLandmarksRequest请求实例的constellation属性来修改使用的检测算法，枚举如下：

```swift
public enum VNRequestFaceLandmarksConstellation : UInt, @unchecked Sendable {
    case constellationNotDefined = 0
    // 使用65个特征点的算法
    case constellation65Points = 1
    // 使用73个特征点的算法
    case constellation76Points = 2
}
```

效果如下图：

![](https://oscimg.oschina.net/oscnet/up-c685c2f1c704b912ccef0602b610bff8b94.png)

Vision框架的静态区域分析中与人脸分析相关的还有一种，使用VNDetectFaceCaptureQualityRequest请求可以分析当前捕获到的人脸的质量，使用此请求分析的结果中会包含如下属性：

```swift
extension VNFaceObservation {
    // 人脸捕获的质量
    @nonobjc public var faceCaptureQuality: Float? { get }
}

```

faceCaptureQualit值越接近1，捕获的人脸效果越好。

## 6 - 水平线识别

VNDetectHorizonReques用来创建水平线分析请求，其可以分析出图片中的水平线位置。此请求本身比较简单，其返回的结果对象为VNHorizonObservation，如下：

```swift
open class VNHorizonObservation : VNObservation {
    // 角度
    open var angle: CGFloat { get }
}
```

分析结果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-64dd93cfaeaf3f4b1b7a9d1c904f7695145.png)

## 7 - 人体相关识别

人体姿势识别也是Vision框架非常强大的一个功能，其可以将静态图像中人体的关键节点分析出来，通过这些关键节点，我们可以对人体当前的姿势进行推断。在运动矫正，健康检查等应用中应用广泛。人体姿势识别请求使用VNDetectHumanBodyPoseRequest类创建，如下：

```swift
open class VNDetectHumanBodyPoseRequest : VNImageBasedRequest {
    // 获取所支持检查的关键节点
    open class func supportedJointNames(forRevision revision: Int) throws -> [VNHumanBodyPoseObservation.JointName]
    // 获取所支持检查的关键节组
    open class func supportedJointsGroupNames(forRevision revision: Int) throws -> [VNHumanBodyPoseObservation.JointsGroupName]
    // 分析结果
    open var results: [VNHumanBodyPoseObservation]? { get }
}
```

VNHumanBodyPoseObservatio分析结果类中封装的有各个关键节点的坐标信息，如下：

```swift
open class VNHumanBodyPoseObservation : VNRecognizedPointsObservation {
    // 可用的节点名
    open var availableJointNames: [VNHumanBodyPoseObservation.JointName] { get }
    // 可用的节点组名
    open var availableJointsGroupNames: [VNHumanBodyPoseObservation.JointsGroupName] { get }
    // 获取某个节点坐标
    open func recognizedPoint(_ jointName: VNHumanBodyPoseObservation.JointName) throws -> VNRecognizedPoint
    // 获取某个节点组
    open func recognizedPoints(_ jointsGroupName: VNHumanBodyPoseObservation.JointsGroupName) throws -> [VNHumanBodyPoseObservation.JointName : VNRecognizedPoint]
}
```

下面示例代码演示了如何对身体姿势节点进行解析：

```swift
private func drawTask(request: VNDetectHumanBodyPoseRequest) {
    boxViews.forEach { v in
        v.removeFromSuperview()
    }
    for result in request.results ?? [] {
        for point in result.availableJointNames {
            if let p = try? result.recognizedPoint(point) {
                let v = UIView(frame: CGRect(x: p.x * imageView.bounds.width - 2, y: (1 - p.y) * imageView.bounds.height - 2.0, width: 4, height: 4))
                imageView.addSubview(v)
                v.backgroundColor = .red
            }
        }
    }
}
```

效果如下图：

![](https://oscimg.oschina.net/oscnet/up-b18855d8efe967f4b00a239bd8b766ee9d9.png)

所有支持的身体节点名和节点组名列举如下：

```swift
// 身体节点
extension VNHumanBodyPoseObservation.JointName {
    // 鼻子节点
    public static let nose: VNHumanBodyPoseObservation.JointName
    // 左眼节点
    public static let leftEye: VNHumanBodyPoseObservation.JointName
    // 右眼节点
    public static let rightEye: VNHumanBodyPoseObservation.JointName
    // 左耳节点
    public static let leftEar: VNHumanBodyPoseObservation.JointName
    // 右耳节点
    public static let rightEar: VNHumanBodyPoseObservation.JointName
    // 左肩节点
    public static let leftShoulder: VNHumanBodyPoseObservation.JointName
    // 右肩节点
    public static let rightShoulder: VNHumanBodyPoseObservation.JointName
    // 脖子节点
    public static let neck: VNHumanBodyPoseObservation.JointName
    // 左肘节点
    public static let leftElbow: VNHumanBodyPoseObservation.JointName
    // 右肘节点
    public static let rightElbow: VNHumanBodyPoseObservation.JointName
    // 左腕节点
    public static let leftWrist: VNHumanBodyPoseObservation.JointName
    // 右腕节点
    public static let rightWrist: VNHumanBodyPoseObservation.JointName
    // 左髋节点
    public static let leftHip: VNHumanBodyPoseObservation.JointName
    // 右髋节点
    public static let rightHip: VNHumanBodyPoseObservation.JointName
    // 身体节点
    public static let root: VNHumanBodyPoseObservation.JointName
    // 左膝节点
    public static let leftKnee: VNHumanBodyPoseObservation.JointName
    // 右膝节点
    public static let rightKnee: VNHumanBodyPoseObservation.JointName
    // 左踝节点
    public static let leftAnkle: VNHumanBodyPoseObservation.JointName
    // 右踝节点
    public static let rightAnkle: VNHumanBodyPoseObservation.JointName
}
// 节点组
extension VNHumanBodyPoseObservation.JointsGroupName {
    // 面部节点组
    public static let face: VNHumanBodyPoseObservation.JointsGroupName
    // 躯干节点组
    public static let torso: VNHumanBodyPoseObservation.JointsGroupName 
    // 左臂节点组
    public static let leftArm: VNHumanBodyPoseObservation.JointsGroupName 
     // 右臂节点组
    public static let rightArm: VNHumanBodyPoseObservation.JointsGroupName 
    // 左腿节点组
    public static let leftLeg: VNHumanBodyPoseObservation.JointsGroupName 
    // 右腿节点组
    public static let rightLeg: VNHumanBodyPoseObservation.JointsGroupName 
    // 所有节点
    public static let all: VNHumanBodyPoseObservation.JointsGroupName
}
```

与人体姿势识别类似，VNDetectHumanHandPoseRequest用来对手势进行识别，VNDetectHumanHandPoseRequest定义如下：

```swift
open class VNDetectHumanHandPoseRequest : VNImageBasedRequest {
    // 支持的手势节点
    open class func supportedJointNames(forRevision revision: Int) throws -> [VNHumanHandPoseObservation.JointName]
    // 支持的手势节点组
    open class func supportedJointsGroupNames(forRevision revision: Int) throws -> [VNHumanHandPoseObservation.JointsGroupName]
    // 设置最大支持的检测人手数量，默认2，最大6
    open var maximumHandCount: Int
    // 识别结果
    open var results: [VNHumanHandPoseObservation]? { get }
}
```

VNHumanHandPoseObservation类的定义如下：

```swift
open class VNHumanHandPoseObservation : VNRecognizedPointsObservation {
    // 可用的节点名
    open var availableJointNames: [VNHumanHandPoseObservation.JointName] { get }
    // 可用的节点组名
    open var availableJointsGroupNames: [VNHumanHandPoseObservation.JointsGroupName] { get }
    // 获取坐标点
    open func recognizedPoint(_ jointName: VNHumanHandPoseObservation.JointName) throws -> VNRecognizedPoint
    open func recognizedPoints(_ jointsGroupName: VNHumanHandPoseObservation.JointsGroupName) throws -> [VNHumanHandPoseObservation.JointName : VNRecognizedPoint]
    // 获取手性
    open var chirality: VNChirality { get }
}
```

chiralit属性用来识别左右手，枚举如下：

```swift
@frozen public enum VNChirality : Int, @unchecked Sendable {
    // 未知
    case unknown = 0
    // 左手
    case left = -1
    // 右手
    case right = 1
}
```

在手势识别中，可用的节点名列举如下：

```swift
extension VNHumanHandPoseObservation.JointName {
    // 手腕节点
    public static let wrist: VNHumanHandPoseObservation.JointName
    // 拇指关节节点
    public static let thumbCMC: VNHumanHandPoseObservation.JointName
    public static let thumbMP: VNHumanHandPoseObservation.JointName
    public static let thumbIP: VNHumanHandPoseObservation.JointName
    public static let thumbTip: VNHumanHandPoseObservation.JointName

    // 食指关节节点
    public static let indexMCP: VNHumanHandPoseObservation.JointName
    public static let indexPIP: VNHumanHandPoseObservation.JointName
    public static let indexDIP: VNHumanHandPoseObservation.JointName
    public static let indexTip: VNHumanHandPoseObservation.JointName

    // 中指关节节点
    public static let middleMCP: VNHumanHandPoseObservation.JointName
    public static let middlePIP: VNHumanHandPoseObservation.JointName
    public static let middleDIP: VNHumanHandPoseObservation.JointName
    public static let middleTip: VNHumanHandPoseObservation.JointName

    // 无名指关节节点
    public static let ringMCP: VNHumanHandPoseObservation.JointName
    public static let ringPIP: VNHumanHandPoseObservation.JointName
    public static let ringDIP: VNHumanHandPoseObservation.JointName
    public static let ringTip: VNHumanHandPoseObservation.JointName

    // 小指关节节点
    public static let littleMCP: VNHumanHandPoseObservation.JointName
    public static let littlePIP: VNHumanHandPoseObservation.JointName
    public static let littleDIP: VNHumanHandPoseObservation.JointName
    public static let littleTip: VNHumanHandPoseObservation.JointName
}

extension VNHumanHandPoseObservation.JointsGroupName {
    // 拇指
    public static let thumb: VNHumanHandPoseObservation.JointsGroupName
    // 食指
    public static let indexFinger: VNHumanHandPoseObservation.JointsGroupName
    // 中指
    public static let middleFinger: VNHumanHandPoseObservation.JointsGroupName
    // 无名指
    public static let ringFinger: VNHumanHandPoseObservation.JointsGroupName
    // 小指
    public static let littleFinger: VNHumanHandPoseObservation.JointsGroupName
    // 全部
    public static let all: VNHumanHandPoseObservation.JointsGroupName
}
```

效果如下图：

![](https://oscimg.oschina.net/oscnet/up-ad80dd6039f6dc72a9cb3eb9e0b4684933b.png)

如果我们只需要识别人体的躯干部位，则使用VNDetectHumanRectanglesRequest会非常方便，VNDetectHumanRectanglesRequest定义如下：

```swift
open class VNDetectHumanRectanglesRequest : VNImageBasedRequest {
    // 设置是否仅仅检测上半身，默认为true
    open var upperBodyOnly: Bool
    // 分析结果
    open var results: [VNHumanObservation]? { get }
}
```

人体躯干识别的结果用法与矩形识别类似，效果如下：

![](https://oscimg.oschina.net/oscnet/up-b223d4620e89caa6bf3f82ed2e627849f62.png)

**需要注意：人体姿势识别和手势识别的API在模拟器上可能无法正常的工作。**

本篇文章，我们介绍了许多关于静态图像区域分析和识别的API，这些接口功能强大，且设计的非常简洁。文本中所涉及到的代码，都可以在如下Demo中找到：

[https://github.com/ZYHshao/MachineLearnDemo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FZYHshao%2FMachineLearnDemo)

> 专注技术，懂的热爱，愿意分享，做个朋友
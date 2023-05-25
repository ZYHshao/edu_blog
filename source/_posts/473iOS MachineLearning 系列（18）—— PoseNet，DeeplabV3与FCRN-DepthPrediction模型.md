---
title: iOS MachineLearning 系列（18）—— PoseNet，DeeplabV3与FCRN-DepthPrediction模型
date: 2023-05-25
categories: 从机器学习到AI
tags: []
---
# iOS MachineLearning 系列（18）—— PoseNet，DeeplabV3与FCRN-DepthPrediction模型

本篇文章将再介绍三个官方的CoreML模型：PoseNet，DeeplabV3和FCRN-DepthPrediction。

PoseNet是人体姿势分析模型，可以识别图片中的人体部分，然后以17个基准点来描述人体的姿势。关于人体姿势的识别，其实Vision框架本来就有此能力，本文主要介绍使用自定义的模型如何实现。Vision框架的用法可以参见如下文章：

[https://my.oschina.net/u/2340880/blog/8681809](https://my.oschina.net/u/2340880/blog/8681809)

DeeplabV3是一个图像分割模型，主要用来实现抠图需求，Vision框架中也有对应的能力，可参加如下文章：

[https://my.oschina.net/u/2340880/blog/8695980](https://my.oschina.net/u/2340880/blog/8695980)

FCRN-DepthPrediction模型用来进行图片景深的预测，这是Vision框架所不具备的能力。

在此地址可以下载到这三个模型：

[https://developer.apple.com/machine-learning/models/](https://developer.apple.com/machine-learning/models/)

## 1 - PoseNet模型

PoseNet模型可以检测17个人体的关键部位或关节，通过这些关键点来构建出完整的人体姿势。

![](https://oscimg.oschina.net/oscnet/up-9c894f7d63116016c44bd351cd7b0e16769.png)

PoseNet最大的模型在6MB左右，相比Vision框架提供的姿势识别，直接使用模型来做会比较麻烦，但是Vision框架也有局限性，其姿势识别的API是在iOS 14之后引入的，如果要支持更低的版本，还是需要我们自己来实现。

首先观察下PoseNet模型在Xcode中的介绍，我们主要专注在输入输出部分：

![](https://oscimg.oschina.net/oscnet/up-5f74c1788492c055d37be19aa5fd025cb24.png)

其中输入部分比较简单，为图片数据。输出部分我们需要关注的是offsets和heatmap两个。其中offsets存储了特征点的位置信息，heatmap存储了特征点的可信度信息。

从模型介绍可以看出，这是一个17\*33\*33的多维数组。其中最外层的一维17分别对应了人体的17个特征位置，每个特征位置对应一个33\*33的矩阵，这就好比，把原始图片横纵等距的画上32条分割线，将图片在水平方向和竖直方向上各均分成33个部分，这里存储的数据即是此人体特征点在当前图片部分上的可信度是多少。换句话说，我们其实就是遍历第一维的数据（17个特征点），然后分别找到每个特征点可信度最大的区块位置即可。要找到特征点的具体位置，需要使用返回的offsets数据，此属性是一个34\*33\*33的多维数组，也很好理解，对第一维来说，前17个元素表示的是每个特征点的y方向偏移，后17个元素表示的是每个特征点的x方向的偏移。每个元素都是一个33\*33的矩阵，我们需要查找第一步找到的可信度最高的区块内的偏移。首先封装一些数据模型类如下：

```swift
// 人体特征点类
class Joint {
    // 特征点名字枚举，这里的顺序和模型多维数组中的顺序是一致的
    enum Name: Int, CaseIterable {
        case nose
        case leftEye
        case rightEye
        case leftEar
        case rightEar
        case leftShoulder
        case rightShoulder
        case leftElbow
        case rightElbow
        case leftWrist
        case rightWrist
        case leftHip
        case rightHip
        case leftKnee
        case rightKnee
        case leftAnkle
        case rightAnkle
    }
    // 名字
    let name: Name
    // 可信度
    var confidence: Double
    // 所在图片上的位置
    var position: CGPoint
    
    init(name: Name,
         position: CGPoint = .zero,
         confidence: Double = 0) {
        self.name = name
        self.position = position
        self.confidence = confidence
    }
}

class PoseModel {
    // 共17个特征点
    let jointCount = 17
    // 此模型会将图片分割成33*33的区块
    let xBlocks = 33
    let yBlicks = 33
    
    // 所有特征组成的字典
    var joints: [Joint.Name: Joint] = [
        .nose: Joint(name: .nose),
        .leftEye: Joint(name: .leftEye),
        .leftEar: Joint(name: .leftEar),
        .leftShoulder: Joint(name: .leftShoulder),
        .leftElbow: Joint(name: .leftElbow),
        .leftWrist: Joint(name: .leftWrist),
        .leftHip: Joint(name: .leftHip),
        .leftKnee: Joint(name: .leftKnee),
        .leftAnkle: Joint(name: .leftAnkle),
        .rightEye: Joint(name: .rightEye),
        .rightEar: Joint(name: .rightEar),
        .rightShoulder: Joint(name: .rightShoulder),
        .rightElbow: Joint(name: .rightElbow),
        .rightWrist: Joint(name: .rightWrist),
        .rightHip: Joint(name: .rightHip),
        .rightKnee: Joint(name: .rightKnee),
        .rightAnkle: Joint(name: .rightAnkle)
    ]
    
    var data: PoseNetMobileNet100S16FP16Output
    // 每个区块的宽高
    var blockWidth: Double
    var blockHeight: Double
    // 使用此输出模型来解析
    init(data: PoseNetMobileNet100S16FP16Output, width: Double, height: Double) {
        self.data = data
        self.blockWidth = width
        self.blockHeight = height
        // 对每个节点进行解析
        joints.values.forEach { joint in
            cofigure(joint: joint)
        }
        
        joints.values.forEach { joint in
            // 打印每个特征点的可信度
            print(joint.name, joint.confidence)
        }
    }
    
    func cofigure(joint: Joint) {
        // 解析出可信度最高的block 记录可信度和区块位置
        var bastConfidence = 0.0
        var bastBlockX = 0
        var bastBlockY = 0
        for x in 0 ..< xBlocks {
            for y in 0 ..< yBlicks {
                let multiArrayIndex = [NSNumber(integerLiteral: joint.name.rawValue), NSNumber(integerLiteral: y), NSNumber(integerLiteral: x)]
                let confidence = data.heatmap[multiArrayIndex].doubleValue
                if confidence > bastConfidence  {
                    bastConfidence = confidence
                    bastBlockX = x
                    bastBlockY = y
                }
            }
        }
        joint.confidence = bastConfidence
        
        // 获取详细的坐标位置，offsets多维数组中存放的是对应特征点的偏移位置，前17个为y偏移，后17个为x偏移。
        let yOffsetIndex = [NSNumber(integerLiteral: joint.name.rawValue), NSNumber(integerLiteral: bastBlockY), NSNumber(integerLiteral: bastBlockX)]
        let xOffsetIndex = [NSNumber(integerLiteral: joint.name.rawValue + jointCount), NSNumber(integerLiteral: bastBlockY), NSNumber(integerLiteral: bastBlockX)]
        let offsetX = data.offsets[xOffsetIndex].doubleValue
        let offsetY = data.offsets[yOffsetIndex].doubleValue
        // 通过偏移量加上区块的起始位置来确定最终位置
        joint.position = CGPoint(x: Double(bastBlockX) * blockWidth + offsetX, y: Double(bastBlockY) * blockHeight + offsetY)
    }
}
```

可以看到，整个解析的过程还是比较复杂，一旦搞定了解析，其使用逻辑就非常简单了，示例如下：

```swift
class PoseModelViewController: UIViewController {

    let image = UIImage(named: "man")!
    lazy var imageView: UIImageView = {
        let v = UIImageView()
        v.image = image
        return v
    }()
    
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
       
        let scale = image.size.width / image.size.height
        
        imageView.frame = CGRect(x: 0, y: 100, width: view.frame.width, height: view.frame.width / scale)
        view.addSubview(imageView)
        // 这些模型都是Xcode自动生成的
        let model = try! PoseNetMobileNet100S16FP16(configuration: MLModelConfiguration())
        let input = try! PoseNetMobileNet100S16FP16Input(imageWith: image.cgImage!)
        let output = try! model.prediction(input: input)
        handleOutPut(outPut: output)
    }
    
    func handleOutPut(outPut: PoseNetMobileNet100S16FP16Output) {
        let pose = PoseModel(data: outPut, width: Double(imageView.frame.width) / 33, height: Double(imageView.frame.height) / 33)
        for joint in pose.joints.values {
            let v = UIView()
            v.frame = CGRect(x: joint.position.x - 3, y: joint.position.y - 3, width: 6, height: 6)
            v.backgroundColor = .red
            v.layer.cornerRadius = 3
            imageView.addSubview(v)
        }
    }
}
```

效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-0e6b50e62e5b219881e29247607853f8e25.png)

## 2 - DeeplabV3模型

DeeplabV3模型用来检测物体的轮廓，简单来说，其是一个用来进行抠图应用的模型。

同样DeeplabV3模型的使用也不像Vision框架那么方便，其模型介绍如下：

![](https://oscimg.oschina.net/oscnet/up-ef518a0c0a943a8b7578b7cdc0a422b4b90.png)

我们只关注其输入和输出，可以看到，此模型会将输入的图片格式化成513\*513的点阵，输出的也是一个513\*513的二维点阵，当这些点的取值要么是0要么是1，我们转换到原图按照0和1的排布进行有色和无色的渲染即可得到蒙层图。使用示例如下：

```swift
import UIKit
import CoreML

class SegModelViewController: UIViewController {


    let image = UIImage(named: "seg.jpeg")!
    lazy var imageView: UIImageView = {
        let v = UIImageView()
        v.image = image
        return v
    }()
    
    lazy var imageView2: UIImageView = {
        let v = UIImageView()
        v.image = image
        return v
    }()
    
    
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
       
        let scale = image.size.width / image.size.height
        
        imageView.frame = CGRect(x: 0, y: 100, width: view.frame.width, height: view.frame.width / scale)
        view.addSubview(imageView)
        
        imageView2.frame = CGRect(x: 0, y: 100 + imageView.frame.height, width: view.frame.width, height: view.frame.width / scale)
        view.addSubview(imageView2)
        // 创建模型
        let model = try! DeepLabV3(configuration: MLModelConfiguration())
        // 创建输入数据结构
        let input = try! DeepLabV3Input(imageWith: image.cgImage!)
        // 进行处理
        let output = try! model.prediction(input: input)
        // 对输出做解析
        handleOutPut(outPut: output)
    }
    
    func handleOutPut(outPut: DeepLabV3Output) {
        let width = imageView2.frame.width / 513
        let height = imageView2.frame.height / 513
        let shape = CAShapeLayer()
        shape.frame = CGRect(x: 0, y: 0, width: imageView2.frame.width, height: imageView2.frame.height)
        shape.anchorPoint = CGPoint(x: 0.5, y: 0.5)
        imageView2.layer.addSublayer(shape)
        var path = CGMutablePath()
        // 对513*513的点阵进行行列遍历
        for line in 0 ..< 513 {
            for column in 0 ..< 513 {
                // 从模型的返回结果中来取值
                let black = outPut.semanticPredictions[[NSNumber(integerLiteral: line), NSNumber(integerLiteral: column)]]
                if black == 0 {
                    path.addRect(CGRect(x: CGFloat(column) * width, y: CGFloat(line) * height, width: width, height: height))
                }
            }
        }
        shape.fillColor = UIColor.black.cgColor
        shape.path = path
    }

}

```

![](https://oscimg.oschina.net/oscnet/up-a2b29468ebfc657e04b5a3a387d77a073cb.png)

## 3 - FCRN-DepthPredictio模型

 FCRN模型进行图片的景物深度预测，即对于平面的图片，此模型可以预测出其中物体离我们的远近。iOS的Vision框架中并没有提供类似的功能接口，因此如果需要分析景深，使用FCRN模型是不错的选择。其实，景深分析在增强现实方面有着很重要的应用。首先，我们还是先看此模型的输入输出：

![](https://oscimg.oschina.net/oscnet/up-a60ed5aefeb7c88483da34ce782477b5e48.png)

可以看到，其讲输入一张图片，并将图片分割成128*160的点阵，将每个点的预测深度返回，其返回的结果越大表示离我们越近，越小则表示离我们越远。完整的示例代码如下：

```swift
import UIKit
import CoreML

class DeppModelViewController: UIViewController {
    
    let image = UIImage(named: "deep2")!
    lazy var imageView: UIImageView = {
        let v = UIImageView()
        v.image = image
        return v
    }()
    
    lazy var imageView2: UIImageView = {
        let v = UIImageView()
        v.image = image
        return v
    }()
    

    override func viewDidLoad() {
        super.viewDidLoad()

        view.backgroundColor = .white
       
        let scale = image.size.width / image.size.height
        
        imageView.frame = CGRect(x: 0, y: 100, width: view.frame.width, height: view.frame.width / scale)
        view.addSubview(imageView)
        
        imageView2.frame = CGRect(x: 0, y: 100 + imageView.frame.height, width: view.frame.width, height: view.frame.width / scale)
        view.addSubview(imageView2)
        // 使用模型进行分析
        let model = try! FCRN(configuration: MLModelConfiguration())
        let input = try! FCRNInput(imageWith: image.cgImage!)
        let output = try! model.prediction(input: input)
        handleOutPut(outPut: output)
    }
    

    func handleOutPut(outPut: FCRNOutput) {
        let width = imageView2.frame.width / 160
        let height = imageView2.frame.height / 128
        // 遍历点阵，根据景深来绘制不同的颜色
        for line in 0 ..< 128 {
            for column in 0 ..< 160 {
                let black = outPut.depthmap[[NSNumber(integerLiteral: 0), NSNumber(integerLiteral: line), NSNumber(integerLiteral: column)]].doubleValue
                let v = UIView()
                v.frame = CGRect(x: CGFloat(column) * width, y: CGFloat(line) * height, width: width, height: height)
                // 这里除2是为了将数据映射到0-1之间
                v.backgroundColor = UIColor(red: black / 2, green: black / 2, blue: black / 2, alpha: 1)
                imageView2.addSubview(v)
            }
        }
    }
}
```

![](https://oscimg.oschina.net/oscnet/up-adc6cab724f7d7df69a251b5e8ee44d28e7.png)

图中颜色越深的区域表示离我们越近。

FCRN模型虽然补充了iOS Vision框架的能力，但其模型大小超过200M，也将会对应用的包体积造成额外的负担。

到此，我们将官方推荐的一些CoreML模型中与视觉有关的就介绍完了，在官方的CoreML模型库中，还提供了一个与文本处理相关的模型，我们将在后续文章中介绍。

本中所涉及到的代码，都可以在如下 Demo 中找到：

[https://github.com/ZYHshao/MachineLearnDemo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FZYHshao%2FMachineLearnDemo)
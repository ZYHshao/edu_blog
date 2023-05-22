---
title: iOS MachineLearning 系列（17）—— 几个常用的对象识别 CoreML 模型
date: 2023-05-22
categories: 从机器学习到AI
tags: []
---
# iOS MachineLearning 系列（17）—— 几个常用的对象识别 CoreML 模型

上一篇文章中，我们介绍了几个官方的图片分类的模型，图片分类模型的应用场景在于将图片中最主要的事物进行识别，在已有的词库中找到最可能得事物。而对象识别则要更高级一些。再之前的文章，我们介绍过可以使用官方提供的API来进行矩形识别，文本识别，二维码识别以及人脸识别等，这类识别功能的特点是我们不仅可以将图片中的物体位置和尺寸分析出来，还可以对其进行类别的分类。

本篇文章，我们将介绍使用官方提供的两个CoreML模型来进行对象识别，这两个模型可以很好的补充系统API的不足，助力业务场景功能的开发。

## 1 - TOLOv3模型

TOLOv3是一个快速识别物体的神经网络模型。关于物体识别，Vision框架提供的有VNRecognizedObjectObservation，此类即描述了识别出的结果。使用自定义的CoreML模型进行对象识别，也会使用到此类。前面我们在使用图片分类的模型时，可以直接使用Xcode帮我们生成的代码，TOLOv3模型则不再适用此方式，我们需要使用Vision框架的接口来实现功能。

其实过程也比较简单，首先加载模型：

```swift
let url = URL(fileURLWithPath: Bundle.main.path(forResource: "YOLOv3", ofType: "mlmodelc")!)
lazy var visionModel = try! VNCoreMLModel(for: MLModel(contentsOf: url))

```

这里需要注意，我们拖入Xcode工程中的模型名字为YOLOv3.mlmodel，但是经过Xcode的编译，实际在IPA包中的模型文件的后缀为mlmodelc，这里一定要注意不要写错。

下面创建一个图片分析请求的处理类：

```swift
// 图像分析请求处理类
lazy var imageRequestHandler = VNImageRequestHandler(cgImage: image.cgImage!,
                                                    orientation: .up,
                                                    options: [:])

```

这一步我们应该相当熟悉了，前面文章介绍Vision框架时，几乎每个示例都会使用到这个类。之后来创建识别请求：

```swift
let objectRecognition = VNCoreMLRequest(model: visionModel, completionHandler: { (request, error) in
    DispatchQueue.main.async(execute: {
        // perform all the UI updates on the main queue
        if let results = request.results {
            self.handleOutPut(outPut: results as! [VNRecognizedObjectObservation])
        }
    })
})

```

这里使用了Vision框架中的VNCoreMLRequest类，这个类通过CoreML模型来创建请求。最后发起请求即可：

```swift
try? imageRequestHandler.perform([objectRecognition])

```

关于请求的结果的处理，其实就是对VNRecognizedObjectObservation数据的处理，示例如下：

```swift
func handleOutPut(outPut: [VNRecognizedObjectObservation]) {
    for objectObservation in outPut {
        // 获取可信度最高的结果
        let topLabelObservation = objectObservation.labels[0]
        // 获取位置和尺寸
        var box = objectObservation.boundingBox
        box.origin.y = 1 - box.origin.y - box.size.height
        let size = imageView.frame.size
        let v = UIView()
        v.frame = CGRect(x: box.origin.x * size.width, y: box.origin.y * size.height, width: box.size.width * size.width, height: box.size.height * size.height)
        v.backgroundColor = .clear
        v.layer.borderColor = UIColor.red.cgColor
        v.layer.borderWidth = 2
        imageView.addSubview(v)
        
        let label = UILabel()
        label.font = .boldSystemFont(ofSize: 18)
        label.text = "\(topLabelObservation.identifier):" + String(format: "%0.2f", topLabelObservation.confidence)
        label.frame = CGRect(x: v.frame.origin.x, y: v.frame.origin.y - 18, width: 200, height: 18)
        label.textColor = .red
        imageView.addSubview(label)
    }
}

```

代码运行效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-fd9743d06cb25e2cd206fdb3b5f47ffadbd.png)

可以看到，这个识别模型对交通工具和信号灯的识别度很高，其用来做辅助驾驶或导航会非常好用。

## 2 - TOLOv3Tiny模型

TOLOv3模型很好用，但其尺寸也很大，32位的模型大小为248.4MB，最算最小的8位模型也要62.2MB。更多时候将这么大的一个模型引入应用中是要付出成本的，最直观的成本就是应用的包体积会增大，增加用户的下载难度。TOLOv3还提供了一个精简版的模型TOLOv3Tiny，从功能上来说，其与TOLOv3所能支持的识别的词库一样，但其只有35.4MB，并且最小的8位模型只有8.9MB。其用法与TOLOv3一样，对同一张图片，其识别效果如下：

![](https://oscimg.oschina.net/oscnet/up-23b4c1452819438e36d2304757359d68c4d.png)

可以看到，相比起来，精简版的模型对物体的边界分析能力要略差一些。识别的精度也更低一些。

这两个模型都可以在如下地址下载：

[https://developer.apple.com/machine-learning/models/](https://developer.apple.com/machine-learning/models/)

示例代码可在此Demo中查看：

[https://github.com/ZYHshao/MachineLearnDemo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FZYHshao%2FMachineLearnDemo)
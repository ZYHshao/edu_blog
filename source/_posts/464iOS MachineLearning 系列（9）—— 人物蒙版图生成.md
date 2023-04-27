---
title: iOS MachineLearning 系列（9）—— 人物蒙版图生成
date: 2023-04-29
categories: 从机器学习到AI
tags: []
---
# iOS MachineLearning 系列（9）—— 人物蒙版图生成

人物蒙版图能力是Vision框架在iOS 15中新增的功能，这个功能可以将图片中的人物按照轮廓生成无光蒙版。无光蒙版在实际业务中非常有用，使用此蒙版可以方便的将人物从图片中提取出来，然后和其他的背景图进行合成。

## 1 - 人物蒙版的提取

首先，人物蒙版的提取非常简单，使用VNGeneratePersonSegmentationRequest创建蒙版分析请求，如下：

```swift
private lazy var personRequest: VNGeneratePersonSegmentationRequest = {
    let request = VNGeneratePersonSegmentationRequest { request, error in
        DispatchQueue.main.async {
            self.drawTask(request: request as! VNGeneratePersonSegmentationRequest)
        }
    }
    return request
}()
```

使用VNImageRequestHandler来触发请求即可：

```swift
// 要分析的图片资源
let image = UIImage(named: "image7")!

// 图像分析请求
lazy var imageRequestHandler = VNImageRequestHandler(ciImage: CIImage(cgImage: image.cgImage!),
                                                orientation: .up,
                                                options: [:])
```

请求的结果VNPixelBufferObservation中会封装蒙版图片CVPixelBuffer数据，如下：

```swift
open class VNPixelBufferObservation : VNObservation {
    // 分析出的蒙版数据
    open var pixelBuffer: CVPixelBuffer { get }
    // 分析所使用的模型
    open var featureName: String? { get }
}
```

处理请求结果如下：

```
private func drawTask(request: VNGeneratePersonSegmentationRequest) {

    for result in request.results ?? [] {
        // 创建CIImage实例
        let ciImage = CIImage(cvPixelBuffer: result.pixelBuffer)
        // 默认返回的蒙版为黑白两色，这里讲所有黑色替换成透明色
        let filter = CIFilter(name: "CIMaskToAlpha", parameters: [kCIInputImageKey: ciImage])
        guard let outputImage = filter?.outputImage else { return }
        let context = CIContext(options: nil)
        let cgImage = context.createCGImage(outputImage, from: outputImage.extent)
        let uiImage = UIImage(cgImage: cgImage!)
        
        let v = UIImageView(image: uiImage)
        v.frame = imageView.frame
        v.backgroundColor = .black
        v.frame.origin.y += imageView.frame.height
        v.image = uiImage
        view.addSubview(v)
    }
}
```

效果如下图：

![](https://oscimg.oschina.net/oscnet/up-fb0af3c03eb6227022ed4b36784d9d373b7.jpg)

## 2 - 进行人物提取

获取到了蒙版，提取人物将很简单，修改上述代码如下：

```swift
private func drawTask(request: VNGeneratePersonSegmentationRequest) {

    for result in request.results ?? [] {
        print(result.pixelBuffer)
        let ciImage = CIImage(cvPixelBuffer: result.pixelBuffer)
        
        let filter = CIFilter(name: "CIMaskToAlpha", parameters: [kCIInputImageKey: ciImage])
        
        guard let outputImage = filter?.outputImage else { return }
        let context = CIContext(options: nil)
        let cgImage = context.createCGImage(outputImage, from: outputImage.extent)
        let uiImage = UIImage(cgImage: cgImage!)
        
        
        let v = UIImageView(image: uiImage)
        v.frame = imageView.frame
        v.backgroundColor = .blue
        v.frame.origin.y += imageView.frame.height

        v.image = uiImage
        view.addSubview(v)
        
        
        
        let v2 = UIImageView(image: image)
        v2.frame = v.frame
        v2.frame.origin.y += v.frame.height
        
        let mask =  UIImageView(image: uiImage)
        mask.frame = CGRect(x: 0, y: 0, width: v2.frame.size.width, height: v2.frame.size.height)
        mask.backgroundColor = .clear
        // 使用蒙版截取
        v2.mask = mask
        view.addSubview(v2)
    }
}
```

效果如下：

![](https://oscimg.oschina.net/oscnet/up-b395ab3aae53f667578dfac78344e2ccf6c.jpg)

完整的示例代码可以在如下地址找到：

[https://github.com/ZYHshao/MachineLearnDemo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FZYHshao%2FMachineLearnDemo)

> VNGeneratePersonSegmentationRequest结合CIFilter的使用，可以方便的进行背景合成。
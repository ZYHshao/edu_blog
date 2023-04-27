---
title: iOS MachineLearning 系列（7）—— 图片相似度分析
date: 2023-04-28
categories: 从机器学习到AI
tags: []
---
# iOS MachineLearning 系列（7）—— 图片相似度分析

图片相似度分析是Vision框架中提供的高级功能。其本质是计算图片的特征值，通过特征值的比较来计算出图片特征差距，从而可以获取到图片的相似程度。在实际应用中，图片的相似度分析有着广泛的应用。如人脸对比识别，相似物品的搜索和识别等。

进行图片相似度计算前，首先需要对图片的特征值进行分析。使用VNGenerateImageFeaturePrintRequest类创建图片特征分析请求。定义如下：

```swift
open class VNGenerateImageFeaturePrintRequest : VNImageBasedRequest {
    // 图片的裁切和缩放配置
    open var imageCropAndScaleOption: VNImageCropAndScaleOption
    // 结果列表
    open var results: [VNFeaturePrintObservation]? { get }
}

```

VNFeaturePrintObservatio结果实例中封装了特征数据：

```swift
open class VNFeaturePrintObservation : VNObservation {
    // 特征类型
    open var elementType: VNElementType { get }
    // 特征元素数量
    open var elementCount: Int { get }
    // 特征数据
    open var data: Data { get }
    // 进行差距比较
    open func computeDistance(_ outDistance: UnsafeMutablePointer<Float>, to featurePrint: VNFeaturePrintObservation) throws
}

```

其中computeDistance方法即用来进行两个分析结果的特征差距计算。对于完全一样的图片，计算的差距为0，差距越大，表明图片的相似度越小。

下面提供了完整的Demo代码：

```swift
import UIKit
import Vision

class ImageFeatureViewController: UIViewController {
    
    let image1 = UIImage(named: "cat1")!
    let image2 = UIImage(named: "cat2")!
    let image3 = UIImage(named: "dog1")!
    let image4 = UIImage(named: "dog2")!
    lazy var imageView1 = UIImageView(image: image1)
    lazy var imageView2 = UIImageView(image: image2)
    lazy var imageView3 = UIImageView(image: image3)
    lazy var imageView4 = UIImageView(image: image4)
    

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        
        view.addSubview(imageView1)
        view.addSubview(imageView2)
        view.addSubview(imageView3)
        view.addSubview(imageView4)
        
        let width = (self.view.frame.width - 60) / 2
        let h1 = width / (image1.size.width / image1.size.height)
        let h2 = width / (image2.size.width / image2.size.height)
        let h3 = width / (image3.size.width / image3.size.height)
        let h4 = width / (image4.size.width / image4.size.height)
        
        imageView1.frame = CGRect(x: 20, y: 100, width: width, height: h1)
        imageView2.frame = CGRect(x: width + 40, y: 100, width: width, height: h2)
        imageView3.frame = CGRect(x: 20, y: max(h1, h2) + 120, width: width, height: h3)
        imageView4.frame = CGRect(x: width + 40, y: max(h1, h2) + 120, width: width, height: h4)
        // 进行特征值分析
        sendRequest(image: image1, number: 1)
        sendRequest(image: image2, number: 2)
        sendRequest(image: image3, number: 3)
        sendRequest(image: image4, number: 4)           
    }
    
    func sendRequest(image: UIImage, number: Int) {
        let handler = VNImageRequestHandler(cgImage: image.cgImage!, orientation: .up)
        let request = VNGenerateImageFeaturePrintRequest { result, error in
            guard error == nil else {
                print(error!)
                return
            }
            let r = result as! VNGenerateImageFeaturePrintRequest
            if number == 1 {
                self.result1 = r.results?.first
            }
            if number == 2 {
                self.result2 = r.results?.first
            }
            if number == 3 {
                self.result3 = r.results?.first
            }
            if number == 4 {
                self.result4 = r.results?.first
            }
            if let result1 = self.result1, let result2 = self.result2, let result3 = self.result3, let result4 = self.result4 {
                // 进行相似性对比
                var distance12 = Float(0)
                try! result1.computeDistance(&distance12, to: result2)
                var distance13 = Float(0)
                try! result1.computeDistance(&distance13, to: result3)
                var distance34 = Float(0)
                try! result3.computeDistance(&distance34, to: result4)
                print("图1与图2相似差距:", distance12)
                print("图1与图3相似差距:", distance13)
                print("图3与图4相似差距:", distance34)
                DispatchQueue.main.async {
                    let l = UILabel()
                    l.text = "图1与图2相似差距:\(distance12)\n图1与图3相似差距:\(distance13)\n图3与图4相似差距:\(distance34)"
                    l.font = .boldSystemFont(ofSize: 22)
                    self.view.addSubview(l)
                    l.frame = CGRect(x: 0, y: max(self.imageView3.frame.height, self.imageView4.frame.height) + self.imageView3.frame.origin.y, width: self.view.frame.width, height: 100)
                    l.numberOfLines = 0
                }
            }
        }
        try? handler.perform([request])
    }
    
    var result1: VNFeaturePrintObservation?
    var result2: VNFeaturePrintObservation?
    var result3: VNFeaturePrintObservation?
    var result4: VNFeaturePrintObservation?

}

```

![](https://oscimg.oschina.net/oscnet/up-a8151fbcc0e927e1c934b9f7c93ce1cd404.jpg)

可以看到，上面两只猫的相似差距为12，猫和狗的相似差距为26，两只狗的相似差距为8。

完整的示例代码可以在如下地址找到：

[https://github.com/ZYHshao/MachineLearnDemo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FZYHshao%2FMachineLearnDemo)
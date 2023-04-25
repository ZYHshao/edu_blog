---
title: iOS MachineLearning 系列（4）—— 静态图像分析之物体识别与分类
date: 2023-04-25
categories: 从机器学习到AI
tags: []
---
# iOS MachineLearning 系列（4）—— 静态图像分析之物体识别与分类

本系列的前几篇文件，详细了介绍了Vision框架中关于静态图片区域识别的内容。本篇文章，我们将着重介绍静态图片中物体的识别与分类。物体识别和分类也是Machine Learning领域重要的应用。通过大量的图片数据进行训练后，模型可以轻易的分析出图片的属性以及图片中物体的属性。

## 1 - 文字识别

文字识别是应用非常广泛的一种图片识别技术。在Vision框架中，使用VNRecognizeTextRequest来进行文字识别，并且其支持多种语言，且有不错的识别精度。VNRecognizeTextRequest的创建示例如下：

```swift
private lazy var recognizeTextRequest: VNRecognizeTextRequest = {
    let textDetectionRequest = VNRecognizeTextRequest { request, error in
        DispatchQueue.main.async {
            self.drawTask(request: request as! VNRecognizeTextRequest)
        }
    }
    // 设置语言
    textDetectionRequest.recognitionLanguages = ["zh-Hans"]
    // 设置识别级别 accurate为最精准 fast为最快速
    textDetectionRequest.recognitionLevel = .accurate
    // 设置是否使用语言矫正
    textDetectionRequest.usesLanguageCorrection = true
    // 获取所支持的语言
    let set = try? textDetectionRequest.supportedRecognitionLanguages()
    print(set)
    return textDetectionRequest
}()

```

可以通过对VNRecognizeTextRequest实例进行配置来调整识别精度，识别的语言，是否进行矫正的选项，VNRecognizeTextRequest类的定义如下：

```swift
open class VNRecognizeTextRequest : VNImageBasedRequest, VNRequestProgressProviding {
    // 所支持的语言列表
    open class func supportedRecognitionLanguages(for recognitionLevel: VNRequestTextRecognitionLevel, revision requestRevision: Int) throws -> [String]
    open func supportedRecognitionLanguages() throws -> [String]
    // 识别过程中所使用的语言
    open var recognitionLanguages: [String]
    // 自定义的词汇，在识别单词时，自定义的词汇优先级会高于默认词典
    open var customWords: [String]
    // 识别等级，精度优先会更加消耗性能
    //  accurate: 精度优先 fast: 速度优先
    open var recognitionLevel: VNRequestTextRecognitionLevel
    // 设置是否使用自动矫正，自动矫正会更加消耗性能
    open var usesLanguageCorrection: Bool
    // 设置是否自动识别语言类型，当不确定输入的语种时，可以设置其自动识别，会更消耗性能
    open var automaticallyDetectsLanguage: Bool
    // 设置可识别文本的最小高度（为相对原图的比例值）
    open var minimumTextHeight: Float
    // 结果数组
    open var results: [VNRecognizedTextObservation]? { get }
}

```

VNRecognizeTextRequest的识别结果为VNRecognizedTextObservation类，此类也是继承自VNRectangleObservation的，因此我们也同时可以获取到所识别的文本所在原图的位置。VNRecognizedTextObservation类的定义如下：

```swift
open class VNRecognizedTextObservation : VNRectangleObservation {
    // 获取候选结果
    open func topCandidates(_ maxCandidateCount: Int) -> [VNRecognizedText]
}

```

topCandidate会返回一组候选结果，其参数设置最多返回的候选结果个数，需要注意此参数所支持的最大值为10。候选结果是指对于同一段文字，可能会识别出多个相似的结果，最终识别的文本结果VNRecognizedText类的定义如下：

```swift
open class VNRecognizedText : NSObject, NSCopying, NSSecureCoding, VNRequestRevisionProviding {
    // 识别出的文本字符串
    open var string: String { get }
    // 本次识别结果的可信度（0-1之间）
    open var confidence: VNConfidence { get }
}

```

对于confidence可信度属性来说，越接近1，可信度越高。

下图演示了照片中文本的识别效果：

![](https://oscimg.oschina.net/oscnet/up-4a8bf9a717a883f7ccafc4bc98778d2c1ba.png)

可以看到，Vision对于中文印刷体的识别能力还是比较准确的。

目前，所支持识别的语种列举如下：

```
en-US：美式英语
fr-FR：法语
it-IT：意大利语
de-DE：德语
es-ES：西班牙语
pt-BR：葡萄牙语
zh-Hans：简体中文
zh-Hant：繁体中文
yue-Hans：粤语简体
yue-Hant：粤语繁体
ko-KR：韩语
ja-JP：日语
ru-RU：俄语
uk-UA：乌克兰语

```

## 2 - 动物识别

虽说是动物识别，但其实目前的API仅仅支持猫和狗的识别。使用VNRecognizeAnimalsRequest类来创建动物识别请求：

```swift
open class VNRecognizeAnimalsRequest : VNImageBasedRequest {
    // 获取所支持识别的动物种类
    open class func knownAnimalIdentifiers(forRevision requestRevision: Int) throws -> [VNAnimalIdentifier]
    open func supportedIdentifiers() throws -> [VNAnimalIdentifier]
    // 结果列表
    open var results: [VNRecognizedObjectObservation]? { get }
}

```

识别的结果VNRecognizedObjectObservation类也是继承自VNDetectedObjectObservation，其会包装所识别的动物所在图片中的区域，且VNRecognizedObjectObservation类中会封装一组VNClassificationObservation对象，如下：

```swift
open class VNRecognizedObjectObservation : VNDetectedObjectObservation {
    // 识别的动物标签
    open var labels: [VNClassificationObservation] { get }
}

```

VNClassificationObservatio类即表示识别出的物体具体的标签，定义如下：

```swift
open class VNClassificationObservation : VNObservation {
    // 标签字符串
    open var identifier: String { get }
}

```

对于VNRecognizeAnimalsRequest请求来说，此标签的值可能为Cat或Dog。识别效果如下图：

![](https://oscimg.oschina.net/oscnet/up-ca643b1bf157b3ff0d4ac67c19c5e29fa28.png)

## 3 - 图片物体分类

图片物体分类是指对静态图片继续分析，将其中可能存在的物体分析出来。使用VNClassifyImageRequest创建图片物体分析请求。此类非常简单，没有太多需要配置的，定义如下：

```swift
open class VNClassifyImageRequest : VNImageBasedRequest {
    // 获取支持识别的物体
    open class func knownClassifications(forRevision requestRevision: Int) throws -> [VNClassificationObservation]
    open func supportedIdentifiers() throws -> [String]
    // 结果数组
    open var results: [VNClassificationObservation]? { get }
}

```

VNClassifyImageRequest所支持识别的物体种类非常多，有千余种，这里就不再列举。其识别后的结果也是VNClassificationObservation类，其内部的identifier表示所识别出的物体的标签。

需要注意，对于略微复杂的图片来说，识别的结果可能非常多，我们需要根据需求来设置一个可信度的阈值，只有达到此可信度的才被采用，例如：

```swift
private func drawTask(request: VNClassifyImageRequest) {
    boxViews.forEach { v in
        v.removeFromSuperview()
    }
    for result in request.results ?? []  where result.confidence > 0.8 {  
        // 解析出文本
        textView.text = textView.text.appending(result.identifier + "\n")
    }
}

```

识别效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-bb2044f667dd81f2eeb7dadc601b2cc4212.jpg)

可以看到，我们选择了大于0.8可信度的结果，所识别出的关键字有：建筑，加工木材，动物，哺乳动物，犬类，狗，博美。（不知为何对猫的识别度很差）

本中所涉及到的代码，都可以在如下 Demo 中找到：

[https://github.com/ZYHshao/MachineLearnDemo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FZYHshao%2FMachineLearnDemo)

> 到此，我们已经将静态图片的分析做了详尽的介绍，相信很多AI能力都是开发中会使用到的。本系列后面文章，将介绍对象追踪的相关API的用法。
---
title: iOS MachineLearning 系列（15）—— 可进行个性化更新的CoreML模型
date: 2023-05-11
categories: 从机器学习到AI
tags: []
---
# iOS MachineLearning 系列（15）—— 可进行个性化更新的CoreML模型

上一篇文章，介绍了使用官方提供的CoreML模型来实现手写数字识别。其实，更多时候我们需要一个更加个性化的模型，对于手写图像来说，每个人的写法可能风格各异，如果可以在用户的使用过程中不断的更新模型，适应更加个性化的场景，就更完美了。幸运的是，CoreML正提供了这样的功能。我们可以创建一个可更新的模型来实现个性化Learning，同样，本文暂无设计模型的训练，我们通过官方的UpdatableDrawingClassifier模型来演示可更新模型的使用。

## 1 - 关于UpdatableDrawingClassifier模型

UpdatableDrawingClassifier是Apple官方提供的一个训练好的并且支持更新迭代的手绘识别模型。其尺寸大小约为394KB，从其大小也可以看出，其本身并没太多的识别能力，我们需要在iOS应用内对其进行更新，使其能够识别个性化的手绘事物。

此处可以下载到此模型：

[https://developer.apple.com/machine-learning/models/](https://developer.apple.com/machine-learning/models/)

首先，我们可以在Xcode中观察下模型的概要信息：

![](https://oscimg.oschina.net/oscnet/up-c723fea3f9ad3c9c35afe985e629a2d4379.png)

相比之前使用的MNIST模型，其多了Updates一栏。我们主要关注Predictions和Updates栏目，其中Predictions说明了使用模型预测时的参数和返回数据，Updates则说明了模型更新时的参数。如下图：

![](https://oscimg.oschina.net/oscnet/up-19881bd8e8997fecb79ce1312dfb28ceab0.png)

Predictions输入参数为图片（需要黑色背景，白色前景），输入有数据有两个，一个是字符串类型的最佳预测结果，一个是字典类型的可能的预测结果。

![](https://oscimg.oschina.net/oscnet/up-4d790fd9c0c8997230e7aac4a212ebe27a7.png)

Updates输入参数为图片（需要黑色背景，白色前景），以及此图片对应的预测文本。

## 2 - UpdatableDrawingClassifier使用示例

以前面文章的手写数字为例，之前使用此模型进行预测，会发现其并不能识别出数字。下面我们尝试让其对此图像进行Learning。

首先定义两个模型操作类实例，分别用来承载原模型与更新后的模型：

```swift
// 更新后的模型
var updatedDrawingClassifier: UpdatableDrawingClassifier?
// 原模型
var baseDrawingClassifier: UpdatableDrawingClassifier?
```

其中UpdatableDrawingClassifier类是Xcode自动生成的，此类文件前一篇文章有过详细的介绍，这里不再赘述。

我们首先使用原始的模型对图像进行预测，如下：

```swift
baseDrawingClassifier = try! UpdatableDrawingClassifier(configuration: MLModelConfiguration())

// 未更新前的模型进行测试
let output = try! baseDrawingClassifier.prediction(input: UpdatableDrawingClassifierInput(drawingWith: UIImage(named: "3")!.cgImage!))
print("未更新前的预测：'\(output.label)'")

label.text = label.text!.appending("模型未更新前的预测结果：\(output.label)\n")
```

不出意外，你将得到“unknow”的结果。

下面我们让模型以此图像作为输入进行Learning，先定义一些要使用到的路径URL：

```swift
// 原模型路径
let defaultModelURL = UpdatableDrawingClassifier.urlOfModelInThisBundle
// 用户目录
let appDirectory = FileManager.default.urls(for: .applicationSupportDirectory,
                                                           in: .userDomainMask).first!
// 临时模型文件路径
lazy var tempUpdatedModelURL = appDirectory.appendingPathComponent("personalized_tmp.mlmodelc")
// 更新后的模型文件路径
lazy var updatedModelURL = appDirectory.appendingPathComponent("personalized.mlmodelc")
```

如下代码演示了对模型进行更新的过程：

```swift
// 定义预期的预测结果
let outputValue = MLFeatureValue(string: "手写数字3")
// 更新的输入参数名
let inputName = "drawing"  // 图片参数
let labelName = "label"    // 预测结果参数

// 获取模型的描述
let description = baseDrawingClassifier.model.modelDescription
// 获取输入参数的描述
let imageInputDescription = description.inputDescriptionsByName[inputName]!
// 获取图片约束字段
let imageConstraint = imageInputDescription.imageConstraint!
// 将图片封装成特征对象
let imageFeatureValue = try! MLFeatureValue(cgImage: UIImage(named: "3")!.cgImage!,
                                            constraint: imageConstraint)
// 组合预期结果与对应的特征对象
let dataPointFeatures: [String: MLFeatureValue] = [inputName: imageFeatureValue,
                                                   labelName: outputValue]
// 定义特征Provider对象
let provider = try! MLDictionaryFeatureProvider(dictionary: dataPointFeatures)

// 使用一组provider创建更新任务
let updateTask = try? MLUpdateTask(forModelAt: defaultModelURL,
                                   trainingData: MLArrayBatchProvider(array: [provider]),
                                   configuration: nil,
                                   completionHandler: updateModelCompletionHandler)
// 执行更新任务
updateTask!.resume()
```

需要注意，通常为了增加模型的预测能力，我们不会仅仅使用一组输入来进行更新，以手写数字为例，我们可以让用户多写几次同样的数字，再进行更新。代码中的updateModelCompletionHandler是更新的回调函数，实现如下：

```swift
func updateModelCompletionHandler(updateContext: MLUpdateContext) {
    saveUpdatedModel(updateContext)
    print("更新完成")
    
    // 重新预测
    updatedDrawingClassifier = try! UpdatableDrawingClassifier(contentsOf: updatedModelURL)
    let output = try! updatedDrawingClassifier.prediction(input: UpdatableDrawingClassifierInput(drawingWith: UIImage(named: "3")!.cgImage!))
    print("更新后的预测：'\(output.label)'")
    DispatchQueue.main.async {
        self.label.text = self.label.text!.appending("新的预测结果：\(output.label)\n")
    }
}

// 存储更新后的模型
func saveUpdatedModel(_ updateContext: MLUpdateContext) {
    let updatedModel = updateContext.model
    let fileManager = FileManager.default
    do {
        // Create a directory for the updated model.
        try fileManager.createDirectory(at: tempUpdatedModelURL,
                                        withIntermediateDirectories: true,
                                        attributes: nil)
        
        try updatedModel.write(to: tempUpdatedModelURL)
        _ = try fileManager.replaceItemAt(updatedModelURL,
                                          withItemAt: tempUpdatedModelURL)
    } catch let error {
        return
    }
}
```

运行代码效果如下图所示，可以看到对同样的图片，新的模型已经可以正确识别了。

![](https://oscimg.oschina.net/oscnet/up-faa166462e380f8994645e1ade95a18b5aa.png)

最后，我们再来总结下可更新模型的使用流程：

1.  准备要更新的物料数据，包括模型的输入数据，以及预期的输出数据。（通常提供一组）
2.  将输入与预期结果对应起来创建MLDictionaryFeatureProvider对象。
3.  通过一组MLDictionaryFeatureProvider对象来创建MLUpdateTask更新任务对象。
4.  执行更新任务，并在回调中存储更新后的模型。
5.  使用新的模型进行预测。

完整的示例代码可以在如下地址找到：

[https://github.com/ZYHshao/MachineLearnDemo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FZYHshao%2FMachineLearnDemo)
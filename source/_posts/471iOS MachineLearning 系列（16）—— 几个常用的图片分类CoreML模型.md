---
title: iOS MachineLearning 系列（16）—— 几个常用的图片分类CoreML模型
date: 2023-05-13
categories: 从机器学习到AI
tags: []
---
# iOS MachineLearning 系列（16）—— 几个常用的图片分类CoreML模型

在本系列的前面文章中，有介绍使用Vision框架中的VNClassifyImageRequest进行图片物体识别。Apple也推荐了几个常用的图像分类模型可供开发者使用。可以在如下地址直接下载：

[https://developer.apple.com/machine-learning/models/](https://developer.apple.com/machine-learning/models/)

## 1 - 几个模型的简单介绍

对于图片识别分类的模型来说，其输入和输出都一样，输入都为图像参数，输入为两部分，一部分为最佳预测结果，一部分为可能得预测结果及其可信度。

### Resnet50

Resnet50是一种神经网络分类模型，其大小为102.6M左右，其可以从1000种类别中检测输入图像中的主要元素，包括树木，动物，食物，车辆和人等。其测量的误差率为7.8%。

### MobileNetV2

MobileNetV2是一种神经网络分类模型，其大小约为24.7M，其同样能从千余种类别中检测图像里的主要元素。其测量的准确率为74.7%。

### SqueezeNet

SqueezeNet也是一种神经网络分类模型，其最大的特点是非常小巧，参数可以减少50倍。我们直接使用的模型大小仅有5M左右。

整体来说，Resnet50体积最大，其预测的精准度也相比会更好，但是对于图片比较清晰，内容比较单一的图像，这些模型都可以满足我们的正常需求。

## 2 - 使用示例

关于CoreML模型的使用，之前的文章有做详细的介绍，这里并没有额外需要注意的地方。我们直接将模型下载下来引入到Xcode工程中，Xcode会自动帮我们生成模型的使用代码，分别初始化3个模型的操作对象如下：

```swift
// mobileNet2模型
let mobileNet2 = try! MobileNetV2(configuration: MLModelConfiguration())
// resent50模型
let resnet50 = try! Resnet50(configuration: MLModelConfiguration())
// squeezeNet 模型
let squeezeNet = try! SqueezeNet(configuration: MLModelConfiguration())
```

对应的，创建输入参数：

```swift
let mobileNet2Input = try! MobileNetV2Input(imageWith: image.cgImage!)
let resnet50Input = try! Resnet50Input(imageWith: image.cgImage!)
let squeezeNetInput = try! SqueezeNetInput(imageWith: image.cgImage!)
```

进行预测：

```swift
// 进行图像识别
let mobileOutput = try! mobileNet2.prediction(input: mobileNet2Input)
let resnetOutput = try! resnet50.prediction(input: resnet50Input)
let squeezeOutput = try! squeezeNet.prediction(input: squeezeNetInput)

label.text = label.text?.appending("mobileNet2模型预测结果：\n最可能：\(mobileOutput.classLabel)\n\n")
label.text = label.text?.appending("resnet50模型预测结果：\n最可能：\(resnetOutput.classLabel)\n\n")
label.text = label.text?.appending("squeeze模型预测结果：\n最可能：\(squeezeOutput.classLabel)\n\n")
```

结果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-95fb7ae46d019122d1232c16e1b9470d3ab.png)

可以看到，3种模型都正确的对图片进行了分类，正确的检测出了图片中的“猕猴”。

完整的示例代码可以在如下地址找到：

[https://github.com/ZYHshao/MachineLearnDemo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FZYHshao%2FMachineLearnDemo)
---
title: iOS MachineLearning 系列（8）—— 图片热区分析
date: 2023-04-29
categories: 从机器学习到AI
tags: []
---
# iOS MachineLearning 系列（8）—— 图片热区分析

对图片进行热区分析可以帮助我们图片中可能会受关注的区域，也可以获取到图片中需要被关注的事物的区域。

Vision框架中提供了两种热区分析的方式。

-   关注区分析
-   对象区分析

## 1 - 关注区分析

关注区分析请求使用VNGenerateAttentionBasedSaliencyImageRequest创建，此类没有特殊需要配置的，其继承自VNImageBasedRequest类，因此可以直接使用VNImageRequestHandler句柄来进行发起。返回的分析结果为VNSaliencyImageObservation对象，定义如下：

```swift
open class VNGenerateAttentionBasedSaliencyImageRequest : VNImageBasedRequest {
    // 关注区对象
    open var results: [VNSaliencyImageObservation]? { get }
}
```

VNSaliencyImageObservatio定义如下：

```swift
open class VNSaliencyImageObservation : VNPixelBufferObservation {
    open var salientObjects: [VNRectangleObservation]? { get }
}
```

下图展示了关注区分析的效果：

![](https://oscimg.oschina.net/oscnet/up-fdbafb9a3ec88f4c1f03f6c5a528554142c.jpg)  ![](https://oscimg.oschina.net/oscnet/up-b7118954ab1eba3bb34a72b69031b02152c.jpg)

可以看到，分析出的关注区基本是图片中内容最丰富的区域，也是我们在观察图片时最先关注到的。需要注意，分析出的关注区不一定会完整的包含图片中的物体。

## 2 - 物体区分析

物体区分析与关注区分析唯一的区别在于请求的创建，物体区分析使用VNGenerateObjectnessBasedSaliencyImageRequest类进行创建。下图演示了物体区分析的效果。

![](https://oscimg.oschina.net/oscnet/up-4516b26c7892f23840f84b74c7cde2e22d4.jpg)    ![](https://oscimg.oschina.net/oscnet/up-9ba8d27510f5ee0758bc0b6fbd4a929ee89.jpg)

可以看到，针对猫狗的那张图片，关注区分析得到的结果是相对居中的位置，物体区分析得到的结果会完整的包含猫狗的边界。

> 在实际应用中，我们可以通过这些分析能力来对图像的部分进行视觉增强。

完整的示例代码可以在如下地址找到：

[https://github.com/ZYHshao/MachineLearnDemo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FZYHshao%2FMachineLearnDemo)

**备注：Vision框架中的很多AI能力可能无法在模拟器上很好的进行。**
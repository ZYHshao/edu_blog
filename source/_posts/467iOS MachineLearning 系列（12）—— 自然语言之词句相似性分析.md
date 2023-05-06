---
title: iOS MachineLearning 系列（12）—— 自然语言之词句相似性分析
date: 2023-05-06
categories: 从机器学习到AI
tags: []
---
# iOS MachineLearning 系列（12）—— 自然语言之词句相似性分析

本篇文章将介绍如何使用NaturalLanguage框架来对词句的相似性进行分析。文本相似性的分析在实际开发中应用很多，比如我们可以通过查找与用户输入相似的词来进行内容推荐。

分析语句的相似性需要进行大量的训练，NaturalLanguage已经内置了许多语言的训练模型，使用非常方便。

## 1 - 文本相似性分析的示例

需要创建NLEmbedding实例来进行词句的相似性分析。例如：

```swift
let embedding = NLEmbedding.wordEmbedding(for: .english)!
let embedding2 = NLEmbedding.sentenceEmbedding(for: .english)!
```

其中，使用wordEmbedding创建出的实例用来进行单词分析，sentenceEmbedding创建出的实例用来进行句子分析。其参数设置所分析的语言，需要注意，并非所有语言都支持进行相似性分析。

下面示例代码演示了如何对单词，句子的相似性进行分析，并能够自动根据传入的单词来进行相近的词的推荐：

```swift
let label = UILabel(frame: CGRect(x: 0, y: 100, width: view.frame.width, height: 500))
label.numberOfLines = 0
label.text = ""
view.addSubview(label)
let word = "dog"
let word2 = "cat"
let word3 = "teacher"

// 计算单词间的矢量距离
let distance = embedding.distance(between: word, and: word2)
let distance2 = embedding.distance(between: word, and: word3)
label.text = label.text!.appending("单词1：\(word)\n单词2：\(word2)\n单词3：\(word3)")
label.text = label.text!.appending("\n\n单词1与单词2间的距离：\(distance)\n单词1与单词3间的距离：\(distance2)")

// 获取相似的词
embedding.enumerateNeighbors(for: word3, maximumCount: 5, using: { item, distance in
    label.text = label.text!.appending("\n与单词3相近的词：\(item) - \(distance)")
    return true
})

let sen = "Hello, Xiao."
let sen2 = "Hi, Xiao."
let sen3 = "My name is Xiao."
// 计算句子间的矢量距离
let distance3 =  embedding2.distance(between: sen, and: sen2)
let distance4 =  embedding2.distance(between: sen, and: sen3)
label.text = label.text!.appending("\n\n\n句子1：\(sen)\n句子2：\(sen2)\n句子3：\(sen3)")
label.text = label.text!.appending("\n\n句子1与句子2间的距离：\(distance3)\n句子1与句子3间的距离：\(distance4)")
```

计算出的矢量距离越大，标明词句间的差异越大，即相似性越差。效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-5238550d5872a91583f610906b388ba569c.png)

通常矢量距离小于1的词句相似性较高。取值范围为0-2之间。

## 2 - 关于NLEmbedding类

要进行词句的相似性分析，需要NLEmbedding类来完成：

```swift
open class NLEmbedding : NSObject {
    // 创建分析单词的实例
    open class func wordEmbedding(for language: NLLanguage) -> NLEmbedding?
    open class func wordEmbedding(for language: NLLanguage, revision: Int) -> NLEmbedding?

    // 创建分析句子的实例
    open class func sentenceEmbedding(for language: NLLanguage) -> NLEmbedding?
    open class func sentenceEmbedding(for language: NLLanguage, revision: Int) -> NLEmbedding?

    // 判断词汇表是否包含参数字符串
    open func contains(_ string: String) -> Bool
    // 向量空间维数
    open var dimension: Int { get }
    // 词汇表单词数量
    open var vocabularySize: Int { get }
    // 所使用的语言
    open var language: NLLanguage? { get }
    // 所使用的算法版本
    open var revision: Int { get }
    
    // 获取所支持的算法版本
    open class func supportedRevisions(for language: NLLanguage) -> IndexSet
    open class func supportedSentenceEmbeddingRevisions(for language: NLLanguage) -> IndexSet
    // 当前默认的算法版本
    open class func currentRevision(for language: NLLanguage) -> Int
    open class func currentSentenceEmbeddingRevision(for language: NLLanguage) -> Int
    
    // 计算两个字符串之间的向量距离
    public func distance(between firstString: String, and secondString: String, distanceType: NLDistanceType = .cosine) -> NLDistance
    // 计算参数字符串的空间向量数据
    public func vector(for string: String) -> [Double]?
    // 获取相似的词句，maxCount控制最大返回个数
    public func enumerateNeighbors(for string: String, maximumCount maxCount: Int, distanceType: NLDistanceType = .cosine, using block: (String, NLDistance) -> Bool)
    public func neighbors(for string: String, maximumCount maxCount: Int, distanceType: NLDistanceType = .cosine) -> [(String, NLDistance)]
    // 通过空间向量数据来获取相似的词句
    public func enumerateNeighbors(for vector: [Double], maximumCount maxCount: Int, distanceType: NLDistanceType = .cosine, using block: (String, NLDistance) -> Bool)
    public func neighbors(for vector: [Double], maximumCount maxCount: Int, distanceType: NLDistanceType = .cosine) -> [(String, NLDistance)]
}
```

其中NLDistance是Double类型的一个别名。直接使用这些API实际上并不能很好的支持中文，NaturalLanguage也支持我们使用自定义的模型，关于模型的使用和训练，我们后面文章会再介绍。

完整的示例代码可以在如下地址找到：

[https://github.com/ZYHshao/MachineLearnDemo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FZYHshao%2FMachineLearnDemo)
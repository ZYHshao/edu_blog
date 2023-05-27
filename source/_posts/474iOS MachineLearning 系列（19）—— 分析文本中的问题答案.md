---
title: iOS MachineLearning 系列（19）—— 分析文本中的问题答案
date: 2023-05-28
categories: 从机器学习到AI
tags: []
---
# iOS MachineLearning 系列（19）—— 分析文本中的问题答案

本篇文章将介绍Apple官方推荐的唯一的一个文本处理模型：BERT-SQuAD。此模型用来分析一段文本，并根据提供的问题在文本中寻找答案。需要注意，BERT模型不会生成新的句子，它会从提供的文本中找到最有可能的答案段落或句子。

BERT模型的使用比较复杂，大致可以分为如下几步：

1.  将词汇表导入。
2.  将问题和原文档分解为Token序列。
3.  使用词汇表将Token序列转换成id序列。
4.  将id序列转换成模型需要的多维数组进行输入。
5.  根据分析的结果解析答案序列。
6.  根据词汇表将答案序列转换为可读的字符串。

我们一步一步来进行介绍。

## 1 - 词汇表

词汇表没有过多需要讲的，其中定义了每个词汇对应的id，在本文末尾会有示例代码工程，工程中自带了需要使用的词汇表。此词汇表包含了3万余个词汇，每个词汇独占一行，其行号即表示当前词汇的id值。

加载词汇表的示例代码如下：

```swift
var tokensDic = Dictionary<Substring, Int>()
func readDictionary() {
    // 读取文件中的数据
    let fileName = "bert-base-uncased-vocab"
    guard let url = Bundle.main.url(forResource: fileName, withExtension: "txt") else {
        fatalError("Vocabulary file is missing")
    }
    guard let rawVocabulary = try? String(contentsOf: url) else {
        fatalError("Vocabulary file has no contents.")
    }
    // 按行进行分割
    let words = rawVocabulary.split(separator: "\n")
    let values = 0..<words.count
    // 加载到字典
    tokensDic = Dictionary(uniqueKeysWithValues: zip(words, values))
}
```

## 2 - 将问题和原文档分解为Token序列

在本系列前面的文章中，有介绍过NaturalLanguage这个框架，其实用来进行自然语言处理的，当然也包含文本的Token分解功能。示例代码如下：

```swift
var wordTokens = [Substring]()
let tagger = NLTagger(tagSchemes: [.tokenType])
// 全部转换成小写
tagger.string = content.lowercased()
tagger.enumerateTags(in: tagger.string!.startIndex ..< tagger.string!.endIndex,
                     unit: .word,
                     scheme: .tokenType,
                     options: [.omitWhitespace]) { (_, range) -> Bool in
    wordTokens.append(tagger.string![range])
    return true
}

var questionTokens = [Substring]()
tagger.string = question.lowercased()
tagger.enumerateTags(in: tagger.string!.startIndex ..< tagger.string!.endIndex,
                     unit: .word,
                     scheme: .tokenType,
                     options: [.omitWhitespace]) { (_, range) -> Bool in
    questionTokens.append(tagger.string![range])
    return true
}
```

## 3 - 将Token序列转换成ID序列

第三步，根据词汇表来将Token序列转换成ID序列，如下：

```swift
// 加载词汇表
readDictionary()
// 转换问题Token序列
let questionTokenIds = questionTokens.compactMap { token in
    tokensDic[token]
}
// 转换原文档Token序列
let contentTokenIds = wordTokens.compactMap { token in
    tokensDic[token]
}
```

## 4 - 将ID序列转换为模型的输入

这一步略微复杂，首先我们先看下BERT-SQuAD模式的输入输出：

![](https://oscimg.oschina.net/oscnet/up-a5c5baf6c41ac4c7efae466c04caacdd70b.png)

需要注意，其输入有两个，wordIDs是ID序列二维数组，其中包含问题，源文档，使用特殊的分隔符进行分割。wordTypes也是一个二维数组，对应的标记wordIDs数组中每个元素的意义。示例代码如下：

```swift
// 开始标记，使用特殊数值101
let startToken = 101
// 分隔符标记，使用特殊数值102
let separatorToken = 102
// 补位标记，使用特殊数值0
let padToken = 0
// 输入wordIDs
var inputTokens:[Int] = []
// 先拼入开始标记
inputTokens.append(startToken)
// 拼入问题ID序列
inputTokens.append(contentsOf: questionTokenIds)
// 拼入分隔符标记
inputTokens.append(separatorToken)
// 拼入源文档ID序列
inputTokens.append(contentsOf: contentTokenIds)
// 拼入分隔符标记
inputTokens.append(separatorToken)
// 不够384位，则用0补齐
while inputTokens.count < 384 {
    inputTokens.append(padToken)
}
// 输入wordTypes
var inputTokenTypes:[Int] = []
// 开始标记，分隔符，和补位标对应的数据位设0
inputTokenTypes.append(0)
// 问题内容位设1
inputTokenTypes.append(contentsOf: Array(repeating: 1, count: questionTokenIds.count))
inputTokenTypes.append(0)
// 源文档内容位设1
inputTokenTypes.append(contentsOf:  Array(repeating: 1, count: contentTokenIds.count))
inputTokenTypes.append(0)
while inputTokenTypes.count < 384 {
    inputTokenTypes.append(0)
}

// 构造MLMultiArray二维数组，其结构为1*384的二维结构
var tokenMultiArray = try! MLMultiArray(shape: [1, 384], dataType: .int32)
for (index, inputToken) in inputTokens.enumerated() {
    tokenMultiArray[[0, NSNumber(integerLiteral: index)]] = NSNumber(integerLiteral: inputToken)
}

var typesMultiArray = try! MLMultiArray(shape: [1, 384], dataType: .int32)
for (index, inputToken) in inputTokenTypes.enumerated() {
    typesMultiArray[[0, NSNumber(integerLiteral: index)]] = NSNumber(integerLiteral: inputToken)
}

```

## 5 - 使用模型进行预测

准备好了输入数据，这一步就非常简单，示例如下：

```swift
let model = try! BERTSQUADFP16(configuration: MLModelConfiguration())
let input = BERTSQUADFP16Input(wordIDs: tokenMultiArray, wordTypes: typesMultiArray)
let output = try! model.prediction(input: input)
handleOutput(output: output)
```

## 6 - 处理输出

BERT-SQuAD模型的输出为两个1\*1\*384*1的四位数组，指定了答案的起始位置与结束位置。虽然输出数据为4维的，但是其有3各维度都只有1个元素，因此我们可以将其提取为一维的，定义方法如下：

```swift
extension MLMultiArray {
    func doubleArray() -> [Double] {
        let unsafeMutablePointer = dataPointer.bindMemory(to: Double.self, capacity: count)
        let unsafeBufferPointer = UnsafeBufferPointer(start: unsafeMutablePointer, count: count)
        return [Double](unsafeBufferPointer)
    }
}

```

处理输出数据如下：

```swift
func handleOutput(output: BERTSQUADFP16Output) {
    // 值越大，表示当前索引为答案的开始位置的可能性越大，找到最可能的答案开始位置
    var startIndex = 0
    for p in startIndex ..< output.startLogits.doubleArray().count {
        if output.startLogits.doubleArray()[p] > output.startLogits.doubleArray()[startIndex] {
            startIndex = p
        }
    }
    // 同理，找到最可能得答案结束位置，这里我们设置答案长度不超过5个Token
    var endIndex = startIndex
    for p in endIndex ..< startIndex + 5 {
        if output.endLogits.doubleArray()[p] > output.endLogits.doubleArray()[startIndex] {
            endIndex = p
        }
    }
    // 获取答案ID序列
    let subs = inputTokens[startIndex ..< endIndex]
    // 将ID序列转回字符串
    for i in subs {
        for item in tokensDic where item.value == i {
            print(item.key)
        }
    }
}
```

代码运行效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-d89b66e2da6f803fee3744ab762f3aebf78.png)

本中所涉及到的代码，都可以在如下 Demo 中找到：

[https://github.com/ZYHshao/MachineLearnDemo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FZYHshao%2FMachineLearnDemo)

本系列文章到此已经将Apple官方所推荐的模型都做了介绍，当然这些模式的训练都是广泛的，不一定会适用于你的应用场景，CoreML框架也提供了更加强大的模型训练能力，我们可以根据自己的场景并提供有针对性的数据进行个性化的模型训练，在后续文章中会再详细讨论。
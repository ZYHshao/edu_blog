---
title: iOS MachineLearning 系列（11）—— 自然语言识别与文本分析
date: 2023-05-05
categories: 从机器学习到AI
tags: []
---
# iOS MachineLearning 系列（11）—— 自然语言识别与单词分析

在上一篇文章中，我们介绍了使用NaturalLanguage框架来进行自然语言的拆解，可以将一段文本按照单词，句子或段落的模式进行拆解。并且，在进行拆解时，其可以自动的识别所使用的语言。

其实，NaturalLanguage框架本身也提供了语言识别的能力，其可以分析一段文本所对应的语言，同样对于包含多种语言的文本，其可以分析出各种语言的占比。语言识别是其他高级自然语言处理任务的基础，本篇文章还将介绍NaturalLanguage关于文本分析的能力，其能够对文本中的人名，地名和组织名进行识别，也可以对词性进行分析，如动词，名词。甚至我们还可以分析文本的积极或消极程度来推测内容的取向，从而帮助开发者开发出更加智能的应用。

## 1 - 语言识别

NLLanguageRecognizer类用来进行语言识别，其可以对输入的文本所使用的语言进行推断，使用非常简单。

首先初始化一个NLLanguageRecognizer实例，如下：

```swift
let recognizer = NLLanguageRecognizer()

```

可以定义一些示例的字符串来测试识别能力，如：

```swift
let string1 = "世界，你好！"
let string2 = "Hello World!"
let string3 = "こんにちは中国"

```

调用NLLanguageRecognizer实例的processString方法即可对字符串进行解析，这个方法是同步的，解析完成后，通过dominantLanguage属性即可获取到这段文本所使用的最接近的语言，例如上面的示例字符串中，string1和string2是比较单纯的中文和英文，string3是日语，日语中很多字是和中文一样的，因此对其进行识别可能会出现误差，我们也可以使用languageHypotheses方法来获取可能识别出的语言，返回的结果中会对识别出的每种语言的可信度进行标记。上面的字符串识别效果如下：

![](https://oscimg.oschina.net/oscnet/up-cf5b18351ad888c1ef9dbde129df60a50b5.png)

其中，zh-Hant为汉语，en为英语，ja为日语。

NLLanguageRecognizer类的使用很简单，其中封装属性和方法列举如下：

```swift
open class NLLanguageRecognizer : NSObject {
    // 类方法，直接对字符串进行主要语言识别
    open class func dominantLanguage(for string: String) -> NLLanguage?
    // 对一个字符串进行识别任务
    open func processString(_ string: String)
    // 重置状态
    open func reset()
    // 最近一次识别任务的结果
    open var dominantLanguage: NLLanguage? { get }
    // 设置说支持的语言，可以设置只支持某些语言的识别
    open var languageConstraints: [NLLanguage]
    // 获取所有可能的语言，参数可以设置最多返回的结果个数，结果中value约接近1的语言可信度越高
    public func languageHypotheses(withMaximum maxHypotheses: Int) -> [NLLanguage : Double]
}

```

NLLanguag是描述语言的结构体，支持的语言列举如下：

```swift
extension NLLanguage {
    // 不确定的
    public static let undetermined: NLLanguage
    // 阿姆哈拉语
    public static let amharic: NLLanguage
    // 阿拉伯语
    public static let arabic: NLLanguage
    // 亚美尼亚
    public static let armenian: NLLanguage
    // 孟加拉语
    public static let bengali: NLLanguage
    // 保加利亚
    public static let bulgarian: NLLanguage
    // 缅甸语
    public static let burmese: NLLanguage
    // 加泰罗尼亚语
    public static let catalan: NLLanguage
    // 切罗基
    public static let cherokee: NLLanguage
    // 克罗地亚
    public static let croatian: NLLanguage
    // 捷克
    public static let czech: NLLanguage
    // 丹麦语
    public static let danish: NLLanguage
    // 荷兰
    public static let dutch: NLLanguage
    // 英语
    public static let english: NLLanguage
    // 芬兰语
    public static let finnish: NLLanguage
    // 法语
    public static let french: NLLanguage
    // 格鲁吉亚
    public static let georgian: NLLanguage
    // 德语
    public static let german: NLLanguage
    // 希腊语
    public static let greek: NLLanguage
    // 古吉拉特语
    public static let gujarati: NLLanguage
    // 希伯来语
    public static let hebrew: NLLanguage
    // 印地语
    public static let hindi: NLLanguage
    // 匈牙利
    public static let hungarian: NLLanguage
    // 冰岛语
    public static let icelandic: NLLanguage
    // 印度尼西亚语
    public static let indonesian: NLLanguage
    // 意大利语
    public static let italian: NLLanguage
    // 日语
    public static let japanese: NLLanguage
    // 埃纳德语
    public static let kannada: NLLanguage
    // 高棉语
    public static let khmer: NLLanguage
    // 韩国语
    public static let korean: NLLanguage
    // 老挝
    public static let lao: NLLanguage
    // 马来语
    public static let malay: NLLanguage
    // 马拉雅拉姆语
    public static let malayalam: NLLanguage
    // 马拉地语
    public static let marathi: NLLanguage
    // 蒙古语
    public static let mongolian: NLLanguage
    // 挪威语
    public static let norwegian: NLLanguage
    // 奥里亚语
    public static let oriya: NLLanguage
    // 波斯语
    public static let persian: NLLanguage
    // 波兰语
    public static let polish: NLLanguage
    // 葡萄牙语
    public static let portuguese: NLLanguage
    // 旁遮普语
    public static let punjabi: NLLanguage
    // 罗马尼亚语
    public static let romanian: NLLanguage
    // 俄语
    public static let russian: NLLanguage
    // 简体中文
    public static let simplifiedChinese: NLLanguage
    // 锡兰语
    public static let sinhalese: NLLanguage
    // 斯洛伐克语
    public static let slovak: NLLanguage
    // 西班牙语
    public static let spanish: NLLanguage
    // 瑞典语
    public static let swedish: NLLanguage
    // 泰米尔语
    public static let tamil: NLLanguage
    // 泰卢固语
    public static let telugu: NLLanguage
    // 泰语
    public static let thai: NLLanguage
    // 藏语
    public static let tibetan: NLLanguage
    // 繁体中文
    public static let traditionalChinese: NLLanguage
    // 土耳其语
    public static let turkish: NLLanguage
    // 乌克兰语
    public static let ukrainian: NLLanguage
    // 乌尔都语
    public static let urdu: NLLanguage
    // 越南语
    public static let vietnamese: NLLanguage
    // 哈萨克语
    public static let kazakh: NLLanguage
}


```

## 2 - 文本分析

文本分析支持对单词进行分析，也支持对句子和段落进行分析。针对不同的需求场景，可以使用不同的方案来分析。在NaturalLanguage框架中，使用NLTagScheme结构体来定义分析方案，支持的方案列举如下：

```swift
extension NLTagScheme {
    // 按元素类型进行标记 可以分析出单词，标点符号，空白符
    public static let tokenType: NLTagScheme
    // 比tokenType方案更进一步，还能分析出词性，如动词，名词等
    public static let lexicalClass: NLTagScheme
    // 名称分析方案，如分析出人名，地名，组织名
    public static let nameType: NLTagScheme
    // nameType和lexicalClass的聚合
    public static let nameTypeOrLexicalClass: NLTagScheme
    // 分析词干，如reading分析词干为read
    public static let lemma: NLTagScheme
    // 标记元素的语言
    public static let language: NLTagScheme 
    // 标记元素的ISO规范的脚本
    public static let script: NLTagScheme
    // 分析内容的消极/积极
    public static let sentimentScore: NLTagScheme
}

```

文本分析的结果会被封装为NLTag结构体，此结构体会包含一个字符串类型的原始值，对于lemma，language，script，sentimentScore分析方案，其结果会直接包装成字符串，其他的分析方案的结果则进行了定义，如下：

```swift
extension NLTag {
    // tokenType方案对应的结果
    public static let word: NLTag // 单词
    public static let punctuation: NLTag // 标点
    public static let whitespace: NLTag // 空白符
    public static let other: NLTag // 其他

    // lexicalClass方案对应的结果
    public static let noun: NLTag  // 名词
    public static let verb: NLTag // 动词
    public static let adjective: NLTag // 形容词
    public static let adverb: NLTag  // 副词
    public static let pronoun: NLTag // 代词
    public static let determiner: NLTag // 限定词
    public static let particle: NLTag // 小品词
    public static let preposition: NLTag // 介词
    public static let number: NLTag // 数词
    public static let conjunction: NLTag // 连词
    public static let interjection: NLTag // 感叹词
    public static let classifier: NLTag // 分类词
    public static let idiom: NLTag  // 惯用语
    public static let otherWord: NLTag // 其他单词
    public static let sentenceTerminator: NLTag // 语句终止符
    public static let openQuote: NLTag // 开引号
    public static let closeQuote: NLTag // 闭引号
    public static let openParenthesis: NLTag // 开括号
    public static let closeParenthesis: NLTag // 闭括号
    public static let wordJoiner: NLTag // 连字符
    public static let dash: NLTag // 破折号
    public static let otherPunctuation: NLTag // 其他标点
    public static let paragraphBreak: NLTag // 段落中断
    public static let otherWhitespace: NLTag // 其他空白符
} 

```

下面，我们来对每种分析方案进行介绍。

### tokenType

tokenType方法非常简单，直接对元素类型进行简单分类，效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-5681d72f4739b9c16c4df13353bc56639da.png)

### lexicalClass

lexicalClass方法相比tokenType更加高级，能够更加细致的单词进行分类，但是需要注意，lexicalClass方案只对英文支持较好。效果如下：

![](https://oscimg.oschina.net/oscnet/up-92df813a36f76c1a3e1069583eaaf56b3f9.png)

### nameType

此方案用来解析文本中的组织名，地名，人名。同样对英文支持较好，如下：

![](https://oscimg.oschina.net/oscnet/up-ac37887db47898c6b2d23b8295d5950a39b.png)

可以看到，其中国家的名字，人名和城市名都正确的解析了出来。

### nameTypeOrLexicalClass

此方案无需做过多的解释，只是两种方法的聚合。

### lemma

此方案用来分析词干，主要也是针对英文，效果如下：

![](https://oscimg.oschina.net/oscnet/up-33c3e49399594ec891709f1669b122d483d.png)

### language与script

这两个方案都是分析元素的语言相关。

### sentimentScore

此方案只能用来进行句子和段落的分析，可以推测出文案内容的积极程度，结果越接近1，标明内容的积极性越高，越接近-1表示越消极。例如：

![](https://oscimg.oschina.net/oscnet/up-9d9e3b488b0bc2fd0e800999caea7c665ba.png)

可以看到其对积极和消极的判定还是比较准确，通过测试，目前也只针对英文有效。

最后，我们再来介绍下用来触发文本分析的NLTagger类，在进行分析前，首先需要实例化此类：

```swift
let tagger = NLTagger(tagSchemes: [.lexicalClass, .tokenType, .lemma, .nameType, .script, .nameTypeOrLexicalClass, .sentimentScore, .language])

```

此实例化方法中传入的参数表示要支持的分析方案。使用如下代码来触发分析：

```swift
tagger.string = string
tagger.enumerateTags(in: string.startIndex ..< string.endIndex, unit: .paragraph, scheme: .sentimentScore) { tag, range in
    resultLabel.text = (resultLabel.text ?? "").appending("【[\(string[range])]-[\(tag?.rawValue ?? "")]】")
    return true
}

```

NLTagger类定义如下：

```swift
open class NLTagger : NSObject {
    // 初始化方法，设置支持的分析方案
    public init(tagSchemes: [NLTagScheme])
    open var tagSchemes: [NLTagScheme] { get }
    // 要进行分析的字符串
    open var string: String?
    // 获取支持的方案（对不同的拆解方式和语言，所能支持的方案不同）
    open class func availableTagSchemes(for unit: NLTokenUnit, language: NLLanguage) -> [NLTagScheme]
    // 输入文本的主语言
    open var dominantLanguage: NLLanguage? { get }
    // 使用自定义模型来定义方案
    open func setModels(_ models: [NLModel], forTagScheme tagScheme: NLTagScheme)
    open func models(forTagScheme tagScheme: NLTagScheme) -> [NLModel]
    open func setGazetteers(_ gazetteers: [NLGazetteer], for tagScheme: NLTagScheme)
    open func gazetteers(for tagScheme: NLTagScheme) -> [NLGazetteer]
    
    // 如果availableTagSchemes没有支持的方案，可能是有资源为加载到设备，使用此方法尝试请求资源
    open class func requestAssets(for language: NLLanguage, tagScheme: NLTagScheme, completionHandler: @escaping (NLTagger.AssetsResult, Error?) -> Void)
    open class func requestAssets(for language: NLLanguage, tagScheme: NLTagScheme) async throws -> NLTagger.AssetsResult
    
    // 获取元素所在字符串范围
    public func tokenRange(at index: String.Index, unit: NLTokenUnit) -> Range<String.Index>
    public func tokenRange(for range: Range<String.Index>, unit: NLTokenUnit) -> Range<String.Index>
    // 对某个位置的元素进行解析
    public func tag(at index: String.Index, unit: NLTokenUnit, scheme: NLTagScheme) -> (NLTag?, Range<String.Index>)
    // 对某个位置的元素进行解析，返回肯能的结果
    public func tagHypotheses(at index: String.Index, unit: NLTokenUnit, scheme: NLTagScheme, maximumCount: Int) -> ([String : Double], Range<String.Index>)
    // 进行完整解析
    public func enumerateTags(in range: Range<String.Index>, unit: NLTokenUnit, scheme: NLTagScheme, options: NLTagger.Options = [], using block: (NLTag?, Range<String.Index>) -> Bool)
    // 进行范围解析
    public func tags(in range: Range<String.Index>, unit: NLTokenUnit, scheme: NLTagScheme, options: NLTagger.Options = []) -> [(NLTag?, Range<String.Index>)]
    // 手动设置语言
    public func setLanguage(_ language: NLLanguage, range: Range<String.Index>)
    public func setOrthography(_ orthography: NSOrthography, range: Range<String.Index>)
}

```

其中availableTagSchemes获取到的可用方案不一定准确，有可能是资源未加载，使用requestAssets可以请求资源，如果最终不能支持，可以从其返回的结果判断：

```swift
public enum AssetsResult : Int, @unchecked Sendable {
    // 可用
    case available = 0
    // 不可用
    case notAvailable = 1
    // 异常
    case error = 2
}

```

enumerateTags方法中有一个options参数，此参数可以对分析的过程进行配置，支持的配置项如下：

```swift
public struct Options : OptionSet, @unchecked Sendable {
    // 忽略单词类型标记
    public static var omitWords: NLTagger.Options { get }
    // 忽略标点类型标记
    public static var omitPunctuation: NLTagger.Options { get }
    // 忽略空白符标记
    public static var omitWhitespace: NLTagger.Options { get }
    // 忽略其他类型元素标记
    public static var omitOther: NLTagger.Options { get }
    // 拼接多单词的名称
    public static var joinNames: NLTagger.Options { get }
    // 拼接缩进
    public static var joinContractions: NLTagger.Options { get }
}

```

完整的示例代码可以在如下地址找到：

[https://github.com/ZYHshao/MachineLearnDemo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FZYHshao%2FMachineLearnDemo)
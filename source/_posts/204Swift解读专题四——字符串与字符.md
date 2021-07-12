---
title: Swift解读专题四——字符串与字符
date: 2016-05-10
categories: Swift语法专题
tags: []
---
## Swift解读专题四——字符串与字符

### 一、引言

        Swift中提供了String类型与Characters类型来处理字符串和字符数据，Swift中的String类型除了提供了许多方便开发者使用的方法外，还可以与Foundation框架的NSString类进行转换，使用起来十分方便。

### 二、String基础

        在Swift中，使用双引号来定义字符串，开发者可以通过如下代码来创建一个字符串常量：

```
let str = "Hello, playground"
```

可以通过下面两种方式来创建空字符串：

```
let str1 = ""
let str2 = String()
```

调用isEmpty方法可以判断某个字符串是否为空字符串，这个方法将返回一个Bool值，可以直接用于if语句：

```
if str1.isEmpty {
    print("this String Object is Empty")
}
```

不像Objective-C有NSString与NSMutableString的区别，在Swift中，如果需要创建可变的字符串，只需用变量来接收：

```
var str3 = "Hello"
str3 += " "+"World"//str3 = Hello World
```

String也可以使用插值的方法来构造新的字符串，使用\\()的方式来将插值的表达式写在小括号内，示例如下：

```
let multiplier = 3
let message = "\(multiplier) times 2.5 is \(Double(multiplier) * 2.5)"//3 times 2.5 is 7.5
```

获取字符串的长度使用如下代码：

```
str3.characters.count
```

Swift中的String可以直接使用==运算符来进行比较，示例如下：

```
let comStr1 = "one two"
let comStr2 = "one two"
comStr1==comStr2//true
```

下面示例的代码，用来检验字符串是否包含前缀与后缀：

```
let tmp3 = "thank you"
tmp3.hasPrefix("thank")//true
tmp3.hasSuffix("you")//true
```

### 三、Character的使用

        Character为Swift中的字符类型，在for-in循环中，可以将字符串中所有的字符进行遍历：

```
for chara in str3.characters {
    print(chara)
}
```

也可以创建单独的字符类型量值，示例如下：

```
let char1 = "🐶"
var cgar2 = "HS"
```

事实上，Sting字符串也可以通过Character字符数组进行初始化：

```
let chars:[Character] = ["H","e","l","l","o"]
let str4 = String(chars)
```

向字符串中追加字符使用如下方法：

```
var str5 = ""
let ca:Character = "a"
str5.append(ca)
```

### 四、字符串中的特殊字符

        字符串中的特殊字符主要指转义字符，Swift中的转义字符列举如下：

```
"\0"//"" 空白符
"\\"//"\"反斜杠符号
"\t"//" "制表符
"\n"//换行符
"\r"//回车符
"\'"//"'"单引号
"\""//"""双引号
"\u{24}"//"$"unicode字符
```

### 五、关于字符串下标

        在Swift中，字符串也可以通过下标的方式来访问其中字符，并且提供了相关方法来方便的移动下标，示例代码如下：

```
let tmp = "Hello Swift"
//获取字符开始的下标值 0
let indexStart = tmp.startIndex
//获取某个下标后一个字符的下标 1
let next = indexStart.successor()
//获取最后一个字符的下标值 注意有\0的存在 
let indexEnd = tmp.endIndex
//获取某个下标前一个字符的下标
let pre = indexEnd.predecessor()
//通过下标获取字符串中的字符 t
var c = tmp[pre]
//进行下标移动 o
var c2 = tmp[indexStart.advancedBy(4)]
//通过遍历下标来遍历字符 H e l l o  S w i f t
for index in tmp.characters.indices {
     print("\(tmp[index]) ", terminator: "")
}

```

### 六、在字符串中插入和移除字符

        使用insert函数来向字符串中插入一个字符，示例如下：

```
var tmp2 = "Hello"
tmp2.insert("!", atIndex: tmp2.endIndex)
```

注意，上面示例代码中的insert函数只能用于插入一个字符，如果需要插入一组字符，需要使用如下方法：

```
tmp2.insertContentsOf(" Swift".characters, at: tmp2.endIndex)
```

使用removeAtIndex函数来移除字符串中的一个字符，示例如下：

```
tmp2.removeAtIndex(tmp2.endIndex.predecessor())
```

如果要移除一组字符，主要通过Range来实现，示例如下：

```
let range = tmp2.endIndex.advancedBy(-4)..<tmp2.endIndex
tmp2.removeRange(range)
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

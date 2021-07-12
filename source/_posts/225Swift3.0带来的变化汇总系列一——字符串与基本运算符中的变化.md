---
title: Swift3.0带来的变化汇总系列一——字符串与基本运算符中的变化
date: 2016-06-16
categories: Swift语法专题
tags: []
---
## Swift3.0带来的变化汇总系列一——字符串与基本运算符中的变化

### 一、引言

      Apple与今年6月13日正式发布了Swift3.0的第一个预览版本，并且相应推出了Xcode8的第一个bate版本。开发者已经可以在Xcode8bate版上来体验Swift3.0的新特性。首先，Swift3.0确实带来了很大改变，许多Swift中的结构体API都进行了更新，例如String，Array等，Swift3.0版本将许多类Objective-C风格的API都更换成了Swift风格的，其目的使开发者可以使用Swift更加惬意有趣的编程。本系列博客，是我观看WWDC视频中介绍的内容以及Swift3.0的开发者帮助文档整理总结而来，在期间，我也参考对比了Swift2.2中的实现方式，希望可以帮助需要的朋友尽快熟悉和上手Swift3.0。

### 二、String类中的API变化

      除了Swift版的Cocoa框架中的API有了大范围的修改外，Swift的一些核心库也有了很大的改动。

      Swift3.0中的字符串类型String在方法API上更加简洁，其中变动较大的是与下标相关的方法，列举如下：

```swift
var string = "Hello-Swift"
//获取某个下标后一个下标对应的字符 char="e"
//swift2.2
//var char = string[startIndex.successor()]
//swift3.0
var char = string[string.index(after: startIndex)]
//获取某个下标前一个下标对应的字符 char2 = "t"
//swift2.2
//var char2 = string[endIndex.predecessor()]
//swift3.0
var char2 = string[string.index(before: string.endIndex)]
//通过范围获取字符串中的一个子串 Hello
//swift2.2
//var subString = string[startIndex...startIndex.advancedBy(4)]
//swift3.0
var subString = string[startIndex...string.index(startIndex, offsetBy: 4)]
//swift2.2
//var subString2 = string[endIndex.advancedBy(-5)...endIndex.predecessor()]
//swift3.0
var subString2 = string[string.index(endIndex, offsetBy: -5)..<endIndex]
//获取某个子串在父串中的范围
//swift2.2
//var range =  string.rangeOfString("Hello")
//swift3.0
var range = string.range(of: "Hello")
//追加字符串操作 此时string = "Hello-Swift! Hello-World"
//swift2.2
//string.appendContentsOf(" Hello-World")
//swift3.0
string.append(" Hello-World")
//在指定位置插入一个字符 此时string = "Hello-Swift!~ Hello-World"
//swift2.2
//string.insert("~", atIndex: string.startIndex.advancedBy(12))
//swift3.0
string.insert("~", at: string.index(string.startIndex, offsetBy: 12))
//在指定位置插入一组字符 此时string = "Hello-Swift!~~~~ Hello-World"
//swift2.2
//string.insertContentsOf(["~","~","~"], at: string.startIndex.advancedBy(12))
//swift3.0
string.insert(contentsOf: ["~","~","~"], at: string.index(string.startIndex, offsetBy: 12))
//在指定范围替换一个字符串 此时string = "Hi-Swift!~~~~ Hello-World"
//swift2.2
//string.replaceRange(string.startIndex...string.startIndex.advancedBy(4), with: "Hi")
//swift3.0
string.replaceSubrange(string.startIndex...string.index(string.startIndex, offsetBy: 4), with: "Hi")
//在指定位置删除一个字符 此时string = "Hi-Swift!~~~~ Hello-Worl"
//swift2.2
//string.removeAtIndex(string.endIndex.predecessor())
//swift3.0
string.remove(at: string.index(before:string.endIndex))
//删除指定范围的字符 此时string = "Swift!~~~~ Hello-Worl"
//swift2.2
//string.removeRange(string.startIndex...string.startIndex.advancedBy(2))
//swift3.0
string.removeSubrange(string.startIndex...string.index(string.startIndex, offsetBy: 2))
var string2 = "My name is Jaki"
//全部转换为大写
//swift2.2
//string2 = string2.uppercaseString
//swift3.0
string2 = string2.uppercased()
//全部转换为小写
//swift2.2
//string2 = string2.lowercaseString
//swift3.0
string2 = string2.lowercased()
```

需要注意，在Swift3.0中Range结构体被划分成了两种类型，Range和ClosedRange，分别用来描述左闭右开区间和闭区间，对应到运算符为0..<10和0...10。

      从上面的示例代码中可以看出，String类型中的很多方法命名进行了Swift风格的简化，改动较大的一个点是关于下标index的改变，移除了两个Index下标移动的方法，使用String类型的index()方法来进行下标的移动操作，编程更加安全。

### 三.基础运算符中的改变

    Swift3.0中的基础运算符并无太大改动，只是移除了取余运算符的浮点数取余功能，取余运算符可以进行浮点运算本是Swift独有的一个特点，3.0版本的改变后，Swift中的"%"运算符功能将与Objective-C与C语言中的取余运算符保持一致。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

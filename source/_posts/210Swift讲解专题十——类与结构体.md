---
title: Swift讲解专题十——类与结构体
date: 2016-05-16
categories: Swift语法专题
tags: []
---
## Swift讲解专题十——类与结构体

### 一、引言

        Swift中的类与结构体十分相似，和Objective-C不同的是，Swift中的结构体不仅可以定义属性，也可以像类一样为其定义方法。

        Swift中的类与结构体有如下相似点：

1.定义属性来存储值。

2.定义函数来提供功能。

3.通过定义下标语法使用下标的方式取值。

4.定义构造方法来对其进行初始化。

5.通过扩展来在原始基础上添加功能。

6.通过协议来定义实现标准。

当然类和结构体也有许多不同点，下面这些功能是类独有的，结构体没有：

1.通过继承来创建类的子类。

2.在运行时允许对类的实例进行类型的检查和解释。

3.析构方法可以释放被类引用的资源。

4.通过引用计数允许一个类实例的多处引用。

        当开发者在代码中传递这些实例时，结构体总是被复制，而类则是被引用。这是结构体和类的最本质区别。

### 二、类与结构体的定义

        类与结构体在定义语法上相似，示例代码如下：

```
class MyClass {
    var name = "HS"
    var age = 25
}
struct MyStruct {
    var param1:Int
    var param2:String
}
//创建类的实例
var obj1 = MyClass()
//创建结构体的实例 所有结构体会默认生成一个逐个设置属性的构造方法 而类不会
var obj2 = MyStruct(param1: 1,param2: "1")
//可以通过点语法来获取类或者结构体中的属性值
print(obj1.age,obj2.param1)
```

通过实例间的传递，可以证明Swift中类被引用于结构体被复制这样的特点，示例如下：

```
//将类实例传递给另一个变量
var obj3 = obj1
//将结构体实例传递给另一个变量
var obj4 = obj2
//修改变量的值
obj3.name = "NewHS"
obj4.param1 = 2
//将 打印 NewHS 1 //说明类是被引用的 结构体则被赋值
print(obj1.name,obj2.param1)
```

注意：在实例传递时同样采用复制原理的还有枚举类型。

        由于类是通过引用来进行传递，Swift中还提供了一种运算符用来比较两个实例变量或常量是否指向同一个引用，示例如下：

```
if obj1===obj3{
    print("same refer")
}else if obj1 !== obj3 {
    print("not same refer")
}
```

实际上，===与!==运算符比较的是指针内容。

### 三、类和结构体的选择

        由于类和结构体有着不同的传递机制，因此其也适用于不同的开发任务，下面这些情况下，官方推荐开发者使用结构体来创建数据类型：

1.该数据类型封装少量的简单数据值。

2.该类型数据来传递时，应该被复制。

3.该类型中定义的数据类型在传递时也应该被赋值。

4.不需要通过继承另一个数据类型而来。

除了上面列举的一些情况，其它情况下，都推荐开发者使用类来描述数据，这也是开发中最后常用的手段。

扩展：在Swift中，Array，String，Dictionary这些类型都是采用的结构体的方式来实现，并不是采用引用的方式，NSString，NSArray，NSDictionary这些Objective-C的类是采用引用的方式实现的，因此在Swift中，String，Array，Dictionary在传递时总是被赋值。然而官方文档中还有一句话十分有意思：

> The description above refers to the “copying” of strings, arrays, and dictionaries. The behavior you see in your code will always be as if a copy took place. However, Swift only performs an _actual_ copy behind the scenes when it is absolutely necessary to do so. Swift manages all value copying to ensure optimal performance, and you should not avoid assignment to try to preempt this optimization.

大致意思是，在你的代码中，拷贝行为看起来似乎总会发生。然而，Swift 在幕后只在绝对必要时才执行实际的拷贝。Swift 管理所有的值拷贝以确保性能最优化，所以你没必要去回避赋值来保证性能最优化。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

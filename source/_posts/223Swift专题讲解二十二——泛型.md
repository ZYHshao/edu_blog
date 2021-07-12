---
title: Swift专题讲解二十二——泛型
date: 2016-05-30
categories: Swift语法专题
tags: []
---
## Swift专题讲解二十二——泛型

### 一、以泛型为参数的函数

        泛型是Swift语言强大的核心，泛型是对类型的抽象，使用泛型开发者可以更加灵活方便的表达代码意图。我们知道，有参函数的参数必须有一个明确的参数类型，有些时候开发者会遇到这样一种情况，编写一个函数用于交换两个变量的值，由于变量有类型之分，实现相同的功能，可能需要重载成多个函数来实现，这大大浪费了开发成本，使用泛型，可以完美的解决这个问题，示例代码如下：

```javascript
func exchange<T>(inout param1:T,inout param2:T){
    let tmp = param1
    param1 = param2
    param2 = tmp
}
var p1 = "15"
var p2 = "40"
exchange(&p1, param2: &p2)
```

上面的方法可以实现对任意相同类型变量的交换，函数参数中使用泛型，需要在函数名后的<>中定义参数占位符，如有多个参数占位符，用逗号隔开即可。

### 二、泛型在类型中的应用

        泛型除了可以作为函数的参数、返回值外，在定义类型时，灵活应用泛型也可以解决很多十分棘手的问题，例如实现一个栈结构的集合类型，示例代码如下：

```javascript
struct Stack<ItemType> {
    var items:[ItemType] = []
    mutating func push(param:ItemType)  {
        self.items.append(param)
    }
    mutating func pop()->ItemType{
        return self.items.removeLast()
    }
}
//整型栈
var obj1 = Stack<Int>()
obj1.push(1)
obj1.pop()
//字符串栈
var obj2 = Stack<String>()
obj2.push("HS")
obj2.pop()
```

        在对使用了泛型的类型进行扩展时，不需要在使用<>进行泛型的定义，直接使用原定义的泛型占位符即可，示例如下：

```javascript
extension Stack{
    func getArray() -> [ItemType] {
        return items
    }
}
```

        有时候，开发者需要对泛型进行一些约束，例如只允许此泛型是继承自某个类或者实现了某个协议，示例代码如下：

```javascript
class MyClass {
    
}
//只有MyClass的子类可以进行Stack栈的创建
struct Stack<ItemType:MyClass> {
    var items:[ItemType] = []
    mutating func push(param:ItemType)  {
        self.items.append(param)
    }
    mutating func pop()->ItemType{
        return self.items.removeLast()
    }
}
```

        在协议中，可以使用另一种方式来进行泛型编程，使用associatedtype关键字可以进行类型关联，示例如下：

```javascript
protocol MyProtocol {
    //实现协议时才指定类型
    associatedtype ItemType
    var param:ItemType {get set}
}
class MyClass:MyProtocol {
    //由于Swift可以自动识别类型 这是MyProtocol中的ItemType为Int
    var param: Int = 0
}
```

### 三、泛型与where子句的结合使用

        使用where子句可以对泛型进行更加严格约束，使其符合开发者需要的逻辑，示例如下：

```javascript
//T和C都要遵守整型协议
class MyClassTwo<T,C where T:IntegerType,C:IntegerType> {
    var param1:T
    var param2:C
    init(param1:T,param2:C){
        self.param1=param1
        self.param2=param2
        
    }
}

var obj3 = MyClassTwo(param1: 1, param2: 1)
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: Swift专题讲解二十——扩展
date: 2016-05-29
categories: Swift语法专题
tags: []
---
## Swift专题讲解二十——扩展

### 一、简介

        Swift中的扩展与Objective-C中的类别功能相似，扩展可以为一个已有的类、结构体、枚举或者协议添加新的属性或方法，与Objective-C的类别不同的是，Swift中的扩展没有名称。

        Swift中的扩展支持如下功能：

1.添加计算属性

2.定义实例方法和类型方法

3.定义新的构造方法

4.定义下标方法

5.定义嵌套类型

6.使一个已有的类遵守协议

7.对协议进行扩展添加新的方法

### 二、使用扩展添加计算属性

        使用extension来声明扩展，示例代码如下：

```javascript
//创建一个类 有两个属性
class MyClass {
    var name:String
    var age:Int
    init(){
        name = "HS"
        age = 24
    }
}
//为MyClass类扩展一个计算属性
extension MyClass {
    var nameAndAge:String{
        return "\(name)"+"\(age)"
    }
}
var obj = MyClass()
obj.nameAndAge
```

### 三、使用扩展添加构造方法

        需要注意的是，扩展不能为类添加指定构造方法，只可以为其添加便利构造方法，示例代码如下：

```javascript
//创建一个类 有两个属性
class MyClass {
    var name:String
    var age:Int
    init(){
        name = "HS"
        age = 24
    }
}
extension MyClass{
    convenience init(name:String,age:Int){
        self.init()
        self.name=name
        self.age=age
    }
}
var obj2 = MyClass(name: "ZYH", age: 24)
```

### 四、使用扩展添加实例方法与类型方法

        扩展可以为一个类型添加实例方法与类型方法，示例如下：

```javascript
//创建一个类 有两个属性
class MyClass {
    var name:String
    var age:Int
    init(){
        name = "HS"
        age = 24
    }
}

extension MyClass{
    func logName() -> String {
        print(name)
        return name
    }
    class func logClassName(){
        print("MyClass")
    }
}

var obj3 = MyClass()
obj3.logName()
MyClass.logClassName()
```

对于值类型的扩展，可以使用可变方法来修改实例本身，示例如下：

```javascript
extension Int{
    //修改本身需要使用nutating
    mutating func change() {
         self = self*self
    }
}
var count = 3
count.change()
//打印9
print(count)
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

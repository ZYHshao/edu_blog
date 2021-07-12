---
title: Swift专题讲解二十一——协议
date: 2016-05-29
categories: Swift语法专题
tags: []
---
## Swift专题讲解二十一——协议

### 一、引言

        协议约定了一些属性与方法，其作用类似Java中的抽象类，Swift中类型通过遵守协议来实现一些约定的属性和方法。Swift中的协议使用protocol关键字来声明。Swift中的协议还有一个十分有意思的特性，协议可以通过扩展来实现一些方法和附加功能。

### 二、在协议中定义属性和方法

        协议中定义的属性只约定名称和类型，在具体类型的实现中，其可以是存储属性也可以是计算属性，协议中还需要指定属性是可读的还是可读可写的。示例代码如下：

```javascript
protocol MyPortocol {
    //定义实例属性
    //可读的
    var name:String{get}
    //可读可写的
    var age:Int{set get}
    //可读的
    var nameAndAge:String{get}
    static var className:String{get}
}
class MyClass: MyPortocol {
    var name: String
    var age: Int
    var nameAndAge: String{
        get{
            return "\(name)"+"\(age)"
        }
    }
    static var className: String{
        get{
            return "MyClass"
        }
    }
    init(){
        name = "HS"
        age = 24
    }
}
```

有一点需要注意，协议中的可读并不是只读，协议中的属性约定成可读可写，则在实现时，这个属性必须是可读可写的，但是如果协议中约定成可读的，则此属性可以是只读的也可以是可读可写的，看具体的实现。

        协议中约定的方法可以是实例方法也可以是类型方法，示例如下：

```javascript
protocol MyPortocol {
    func logName()
    static func logClassName()
}
class MyClass: MyPortocol {
    var name: String
    var age: Int
    init(){
        name = "HS"
        age = 24
    }
    func logName() {
        print(name)
    }
    static func logClassName() {
        print(className)
    }
}
```

同样，协议中也可以对构造方法进行定义约定。

### 三、协议的特点

        协议中虽然没有任何属性和方法的实现，但是其仍然可以当做类型来使用，在函数参数、返回值中应用广泛，示例如下：

```javascript
protocol MyPortocol {
    //定义实例属性
    var name:String{get}
    var age:Int{set get}
    var nameAndAge:String{get}
    static var className:String{get}
    func logName()
    static func logClassName()
}
//将协议类型作为参数
func test(param:MyPortocol)  {
    param.logName()
}

```

协议作为类型这种用法另一个应用点是在集合类型中，协议可以作为所有遵守此协议的集合类型。

        协议可以像其他类型一样进行继承，子协议将自动拥有父协议约定的属性和方法。协议也可以通过class关键字来定义只有类可以进行遵守，示例如下：

```javascript
protocol MyPortocol {
    //定义实例属性
    var name:String{get}
    var age:Int{set get}
    var nameAndAge:String{get}
    static var className:String{get}
    func logName()
    static func logClassName()
}
//只有类可以继承此协议
protocol MySubPortocol:class,MyPortocol {
    
}
```

        协议既然可以像其他类型一样进行使用，当然它也可以使用is，as?，as!进行检查和转换，关于is，as的更多用法可以查看Swift关于类型转换的内容。

        协议也可定义其中的属性或方法为可选的，即遵守此协议的类可以实现也可以不实现可选的属性和方法，然而，声明为可选的需要此协议为@objc类型的，示例如下：

```javascript
@objc protocol MyPortocol {
    //定义实例属性
    var name:String{get}
    var age:Int{set get}
    var nameAndAge:String{get}
    static var className:String{get}
    func logName()
    //可选实现
    optional static func logClassName()
}
```

        Swift中的协议还有一个十分重要的特性，其可以通过扩展来进行属性、方法以及下标的实现。这对于一些通用类的方法十分方便，这相当于所有继承此协议的类都默认实现了这样的方法，示例如下：

```javascript
protocol MyPortocol {
    //定义实例属性
    var name:String{get}
    var age:Int{set get}
    var nameAndAge:String{get}
    static var className:String{get}
    func logName()
    static func logClassName()
}
extension MyPortocol{
    var name:String{
        return "HS"
    }
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: Swift讲解专题十一——属性
date: 2016-05-16
categories: Swift语法专题
tags: []
---
## Swift讲解专题十一——属性

### 一、引言

        属性将值与类，结构体，枚举进行关联。Swift中的属性分为存储属性和计算属性两种，存储属性用于存储一个值，其只能用于类与结构体，计算属性用于计算一个值，其可以用于类，结构体和枚举。

### 二、存储属性

        存储属性使用变量或者常量来存储一个值，在声明存储属性时，可以为其设置一个默认值，也可以在构造示例是进行值的设置，属性可以通过点语法来访问，结构体的存储属性示例代码如下：

```
struct MyStruct {
    var property1 = 1
    var property2:Int
}
var obj = MyStruct(property1: 1, property2: 2)
//通过点语法进行属性的访问
print(obj.property1,obj.property2)
```

如上结构体，如果有属性被声明成let常量，则此属性不能够被修改。还有一点需要注意，如果在创建结构体的实例时，使用的是let进行创建，则即便结构体中的属性是变量也不可进行修改。这和类有很大区别。

        还有一类存储属性叫做延时存储属性，可以设想一下这样的情形，类的某些属性可能并不是在每次类实例后都会用到，并且有些属性的构造可能会消耗大量的时间，这时一个比较聪明的设计便是在类进行实例化时，这类属性并不被构造，当次类的实例使用到这个属性时，这个属性才被构造出来，这样的属性被称为延时存储属性，使用lazy关键字来声明，示例如下：

```
//第一个类
class MyClass1 {
    init(){
        print("MyClass1类被构造")
    }
}
class MyClass2 {
    //声明为延时存储属性
    lazy var body = MyClass1()
}
//在构造MyClass2时 并不会进行body属性的构造 不会有打印信息
var obj2 = MyClass2()
//执行下面代码后 会有打印信息 使用body属性使得body被构造
obj2.body
```

注意，如果在多个线程中对延时构造属性进行使用，不能保证其只被构造一次。

### 三、计算属性

        简单的理解，计算属性并不是独立的用于存储值的属性，开发者甚至可以将其理解为一个计算方法，其主要用于通过计算来获取或者设置其他存储属性的值。示例如下：

```
struct Circle {
    //圆心
    var center:(Double,Double)
    //半径
    var r:Double
    //周长 将其作为计算属性
    var l:Double{
        get{
            //计算圆的周长
           return 2.0*r*M_PI
        }
        set{
           //通过周长重新计算半径 默认传入的参数名为newValue
            r = newValue/(M_PI*2)
        }
    }
}
var circle = Circle(center: (0,0), r: 2)
print(circle.l)
circle.l=24
print(circle.r)
```

通过上面的演示代码可以了解，l属性并非是一个新的属性，只是通过r属性来计算出l，或者通过l来反推出r，其中有一点需要注意，计算属性中可以创建两个代码块set和get，set代码块是可选的，其中会默认生成一个newValue参数来传递外界传进来的数据，get代码块是必须要实现的，当然也可以只实现get代码块，这时这个属性将是只读的计算属性，只可以获取，不能够设置。还有一点需要注意，开发者也可以在set代码块后面自定义一个参数名来接收外界传入的参数，示例如下：

```
struct Circle {
    //圆心
    var center:(Double,Double)
    //半径
    var r:Double
    //周长 将其作为计算属性
    var l:Double{
        get{
            //计算圆的周长
           return 2.0*r*M_PI
        }
        set(newL){
           //通过周长重新计算半径 默认传入的参数名为newValue
            r = newL/(M_PI*2)
        }
    }
}
```

只读的计算属性可以进行进一步的简写，因为没有了set代码块，所以关键字get和括号也可以给省略掉，不会产生歧义，示例如下：

```
struct Point {
    var x:Double
    var y:Double
    var center:(Double,Double){
        return (x/2,y/2)
    }
}
```

### 四、属性监听器

        Swift中的计算属性中的get和set方法和Objective-C中的get和set方法其实并非是一回事，Objective-C提供set和get方法可以让开发者在属性将要获取或者设置的时候来进行一些自定义的操作，这部分的开发需求在Swift中通过属性监听器来实现。

        属性监听器有willSet和didSet两种，willSet在属性值将要变化时执行，didSet在属性值已经变化时执行，并且其中会传入变化前后的值。示例如下：

```
struct Point {
    var x:Double
    var y:Double{
        willSet{
            print("将要进行值的更新设置,新的值是:",newValue)
        }
        didSet{
            print("已经进行值得更新设置,旧的值是:",oldValue)
        }
    }
    var center:(Double,Double){
        return (x/2,y/2)
    }
}
var point = Point(x: 3, y: 3)
//将打印
/*
 将要进行值的更新设置,新的值是: 4.0
 已经进行值得更新设置,旧的值是: 3.0
 */
point.y=4
```

willSet中默认会生成一个命名为newValue的参数，didSet中会默认生成一个命名为oldValue的参数，也可以自定义这些参数的命名，示例如下：

```
struct Point {
    var x:Double
    var y:Double{
        willSet(new){
            print("将要进行值的更新设置,新的值是:",new)
        }
        didSet(old){
            print("已经进行值得更新设置,旧的值是:",old)
        }
    }
    var center:(Double,Double){
        return (x/2,y/2)
    }
}
```

### 五、实例属性与类型属性

        实例属性是针对与一个类型的实例，类型属性则是直接针对与类型。  每对类型进行一次实例化，其实例都有一套独立的实例属性，而类型属性则是类的所有实例所共用的，在Objective-C中，通常使用全局的属性来实现这样的效果，在Swift中，使用static关键字来声明类型属性，示例如下：

```
struct Point {
    //类型存储属性
    static var name:String = "Point"
    //类型计算属性
    static var subName:String{
        return "sub"+name
    }
    var x:Double
    var y:Double{
        willSet(new){
            print("将要进行值的更新设置,新的值是:",new)
        }
        didSet(old){
            print("已经进行值得更新设置,旧的值是:",old)
        }
    }
    var center:(Double,Double){
        return (x/2,y/2)
    }
}
//类型属性 通过类型点语法来获取
print(Point.name,Point.subName)
```

注意，有一种特殊的情况是针对于类的类型计算属性，如果其需要子类进行继承重写，需要将static关键字，换成class关键字，示例如下：

```
class SomeClass {
    static var storedTypeProperty = "Some value."
    static var computedTypeProperty: Int {
        return 27
    }
    //支持子类进行重写的计算属性
    class var overrideableComputedTypeProperty: Int {
        return 107
    }
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

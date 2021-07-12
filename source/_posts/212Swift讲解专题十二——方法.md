---
title: Swift讲解专题十二——方法
date: 2016-05-17
categories: Swift语法专题
tags: []
---
## Swift讲解专题十二——方法

### 一、引言

        方法只是一个术语，其实就是将函数与特定的类型结合，类、结构体、枚举都可以定义方法，方法又分为实例方法和类型方法，类型方法类似于Objective-C中的类方法。Swift和Objective-C的一大不同是，Objective-C只有在类中可以定义方法。

### 二、实例方法基础

        实例方法的语法和函数完全一致，其和具体类型的实例所关联，实例方法在调用时由类型的实例点语法进行调用来完成一些功能模块。示例如下：

```
class Math {
    //完成加法功能的实例方法
    func add(param1:Double,param2:Double)->Double{
        return param1+param2
    }
}
//创建类型实例
var obj = Math()
//调用方法进行计算
obj.add(5, param2: 5)
```

与Objective-C类似，Swift中每一个类的实例中都隐藏含有一个self属性，self属性就是实例本身，开发者可以在实例方法中使用self来调用属性或者其他实例方法，示例如下：

```
class Math {
    //完成加法功能的实例方法
    func add(param1:Double,param2:Double)->Double{
        return param1+param2
    }
    func mul(param1:Double,param2:Double) -> Double {
        //使用self调用实例方法
        self.add(param1, param2: param2)
        return param1*param2
    }
}
```

然而，Swift并不要求开发者必须写self，默认情况下，开发者可以直接省略self来调用属性和方法：

```
class Math {
    //完成加法功能的实例方法
    func add(param1:Double,param2:Double)->Double{
        return param1+param2
    }
    func mul(param1:Double,param2:Double) -> Double {
        //使用self调用实例方法
        add(param1, param2: param2)
        return param1*param2
    }
}
```

有一种情况需要注意，对于属性的调用，如果方法中的参数名和类实例的属性名相同，则必须使用self来调用类的实例属性，防止歧义的产生：

```
class Math {
    var param1 = 10.0
    //完成加法功能的实例方法
    func add(param1:Double,param2:Double)->Double{
        //这里将使用param1=10，如果不加self 将使用参数中的param1
        return self.param1+param2
    }
    func mul(param1:Double,param2:Double) -> Double {
        //使用self调用实例方法
        add(param1, param2: param2)
        return param1*param2
    }
}
```

### 三、在实例方法中修改值类型的值

        首先需要清楚一个概念，Swift中有两种类型，值类型和引用类型，具体在类、结构体、枚举一节中有相关介绍，这里需要注意的是，对于值类型，即结构体和枚举，其并不能直接在实例方法中修改实例属性的值，Swift中提供了另一种方式，如果真有如此的需求，开发者可以使用mutating关键字将实例方法声明成可变的，实际上，如果在可变的实例方法中修改了值类型属性的值，是会创建一个新的实例来代替原来的实例的，示例如下：

```
struct Point {
    var x:Double
    var y:Double
    mutating func move(x:Double,y:Double) {
        self.x+=x
        self.y+=y
    }
}
var point = Point(x: 1, y: 1)
print(point)
point.move(3, y: 3)
print(point)
```

在值类型实例的可变方法中修改属性的值，实际上就是创建了一个新的实例，上面的写法和下面的写法原理是一样的：

```
struct Point {
    var x:Double
    var y:Double
    mutating func move(x:Double,y:Double) {
        self = Point(x: self.x+x,y: self.y+y)
    }
}
```

### 四、类型方法

        正如实例方法是通过类型的实例来进行调用的，类型方法是通过类型直接来调用的，相比于实例方法，类型方法中的self指当前类型，同样开发者可以使用self来区别类型属性和类型方法中的参数。使用Static关键字来进行类型方法的创建：

```
struct Point {
    var x:Double
    var y:Double
    mutating func move(x:Double,y:Double) {
        self = Point(x: self.x+x,y: self.y+y)
    }
    static func name(){
        print("Point")
    }
}
Point.name()
```

如果是在类中创建类型方法，若此方法可以被子类进行重写，则应该使用class关键字来创建，示例如下：

```
class Math {
    var param1 = 10.0
    //完成加法功能的实例方法
    func add(param1:Double,param2:Double)->Double{
        //这里将使用param1=10，如果不加self 将使用参数中的param1
        return self.param1+param2
    }
    func mul(param1:Double,param2:Double) -> Double {
        //使用self调用实例方法
        add(param1, param2: param2)
        return param1*param2
    }
    class func name(){
        print("Math")
    }
}
Math.name()
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

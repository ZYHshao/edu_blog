---
title: Swift专题讲解十四——继承
date: 2016-05-18
categories: Swift语法专题
tags: []
---
## Swift专题讲解十四——继承

### 一、引言

        Swift中，一个类可以从另一个类继承方法、属性、下标及其他特性。当一个类继承于另一个类时，这个类被称为子类，所继承的类被称为父类。在Swift中，继承是类区别于其他类型的主要特征。子类除了可以调用父类的属性，下标，方法外，其也可以对父类的属性，下标，方法进行覆写。

### 二、定义一个基类

        不继承于任何类的类被称为基类，示例如下：

```
class Shape {
    var center:(Double,Double)
    init(){
        center = (0,0)
    }
}
```

上面代码定义了一个图形类，其中定义了一个中心点，任何图形都会有中心点，所以把其作为基类属性。

### 三、定义一个子类

        图形基类可以派生出许多图形子类，例如矩形，圆形等，示例代码如下：

```
class Circle: Shape {
    var Radio:Double = 0
}
class Rect: Shape {
    var size:(Double,Double)=(0,0)
}
var circle = Circle()
circle.center = (1,1)
```

可以看到，Circle类从父类中继承到了center属性。默认子类也会继承父类的构造方法，如果子类需要实现自己的构造方法，可以对父类的方法进行覆写，使用override关键字：

```
class Rect: Shape {
    var size:(Double,Double)=(0,0)
    override init(){
        super.init()
        super.center = (1,1)
    }
}
```

通过super关键字可以调用父类的属性和方法，同样，也可以使用override关键字来对属性进行get和set的覆写。同样也可以重写属性的观察期willset和didset。

### 四、final关键字

        在开发中很多情况下为了安全考虑，有些方法和属性是不允许子类进行覆写的，使用final声明这些属性，方法或者下标可以起到这样的作用。示例如下：

```
class Shape {
    final var center:(Double,Double)
    init(){
        center = (0,0)
    }
}
```

如果想将某个类设置为不可继承的，可以将此类使用final关键字修饰，示例如下：

```
final class Shape {
    final var center:(Double,Double)
    init(){
        center = (0,0)
    }
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

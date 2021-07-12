---
title: Swift专题讲解十六——ARC在Swift中的应用
date: 2016-05-19
categories: Swift语法专题
tags: []
---
## Swift专题讲解十六——ARC在Swift中的应用

### 一、引言

        ARC（自动引用计数）是Objective-C和Swift中用于解决内存管理问题的方案。在学习Objective-C编程时经常会学习到一个关于ARC的例子：在一个公用的图书馆中，每次进入一人就将卡插入，走的时候将自己的卡拔出拿走。图书馆系统会判定只要有卡插入，就将图书馆的灯打开，当所有卡都被取走后，将图书馆的灯关掉。这个例子对应于Objective-C中的对象声明周期管理十分贴切。每当一个对象增加一个引用时，其引用计数会加1，当一个引用被取消时，对象的引用计数减1，当引用计数减为0时，说明此对象将不再有任何引用，对象会被释放掉，让出内存。Swift也采用同样的方式进行内存管理。

        注意：在Swift中只有引用类型有自动引用计数，结构体、枚举这类值类型是没有引用计数的。关于引用计数的示例代码如下：

```
class MyClass {
    deinit{
        print("MyClass deinit")
    }
}
var cls1:MyClass? = MyClass()
var cls2:MyClass? = cls1
var cls3:MyClass? = cls2
cls2 = nil
cls1 = nil
//执行下面代码后才会打印“MyClass deinit”
cls3 = nil
```

### 二、循环引用的处理方法

        在开发中，开发者一不小心就会写出产生循环引用的代码，在上面的示例中可以看出，除非实例的引用全部解除，否则实例将不会调用析构方法，内存不会被释放，如果在写代码时，A引用了B，同样B也引用了A，那么实际上现在A和B的引用计数都是2，将A和B都置为nil后，A和B实例依然保有1个引用计数，都不会被释放，实例如下：

```javascript
class MyClassOne {
    var cls:MyClassTwo?
    deinit{
        print("ClassOne deinit")
    }
}
class MyClassTwo {
    var cls:MyClassOne?
    deinit{
        print("ClassTwo deinit")
    }
}
var obj1:MyClassOne? = MyClassOne()
var obj2:MyClassTwo? = MyClassTwo()
obj1?.cls = obj2
obj2?.cls = obj1
obj1=nil
obj2=nil
//没有打印析构函数的调用信息
```

对于上面的情况，可以将属性声明称weak类型来防止这种循环引用，weak的作用在于只是弱引用实例，原实例的引用计数并不会加1，示例如下：

```javascript
//关于弱引用的演示
class MyClassThree{
    weak var cls:MyClassFour?
    deinit{
        print("ClassThree deinit")
    }
}
class MyClassFour {
    var cls:MyClassThree?
    deinit{
        print("ClassFour deinit")
    }
}
var obj3:MyClassThree? = MyClassThree()
var obj4:MyClassFour? = MyClassFour()
obj3?.cls = obj4
obj4?.cls = obj3
obj4=nil
//此时obj3中的cls也为nil
obj3?.cls

```

若引用的实例被释放后，其在另一个实例中的引用也将被置为nil，所以weak只能用于optional类型的属性，然而在开发中还有一种情况，某个类必须保有另一个类的示例，这个实例不能为nil，但是这个属性又不能影响其原始实例的释放，这种情况也会造成循环引用，示例如下：

```javascript
class MyClassFive{
    var cls:MyClassSix
    init(param:MyClassSix){
        cls = param
    }
    deinit{
        print("ClassFive deinit")
    }
}
class MyClassSix{
    var cls:MyClassFive?
    deinit{
        print("ClassSix deinit")
    }
}
var obj6:MyClassSix? = MyClassSix()
var obj5:MyClassFive? = MyClassFive(param: obj6!)
obj6?.cls = obj5
obj5=nil
obj6=nil
//没有打印任何信息
```

上面的示例也会造成循环引用，然而MyClassFive类中的cls属性为常量不可为nil，不可使用weak弱引用来做Swift中又提供了一个关键字unowned无主引用来处理这样的问题，示例如下：

```javascript
class MyClassFive{
    unowned var cls:MyClassSix
    init(param:MyClassSix){
        cls = param
    }
    deinit{
        print("ClassFive deinit")
    }
}
class MyClassSix{
    var cls:MyClassFive?
    deinit{
        print("ClassSix deinit")
    }
}
var obj6:MyClassSix? = MyClassSix()
var obj5:MyClassFive? = MyClassFive(param: obj6!)
obj6?.cls = obj5
obj5=nil
obj6=nil
```

关于弱引用和无主引用，其区别主要是在于：

1.弱引用用于解决Optional值的引起的循环引用。

2.无主引用用于解决非Optional值引起的循环引用。

3.个人以为，弱引用可用下图表示：

![](http://static.oschina.net/uploads/space/2016/0520/195243_WfvB_2340880.png)

4.无主引用可用如下图表示：

![](http://static.oschina.net/uploads/space/2016/0520/200314_99us_2340880.png)

若将上面的代码修改如下，程序会直接崩溃：

```javascript
class MyClassFive{
    unowned var cls:MyClassSix
    init(param:MyClassSix){
        cls = param
    }
    deinit{
        print("ClassFive deinit")
    }
}
class MyClassSix{
    var cls:MyClassFive?
    deinit{
        print("ClassSix deinit")
    }
}
var obj6:MyClassSix? = MyClassSix()
var obj5:MyClassFive? = MyClassFive(param: obj6!)
obj6?.cls = obj5
obj6=nil
obj5?.cls
```

上面所举的例子满足了两种情况，一种是两类实例引用的属性都是Optional值的时候使用weak来解决循环引用，一种是两类实例有一个为非Optional值的时候使用unowned来解决循环引用，然而还有第三种情况，两类实例引用的属性都为非Optional值的时候，可以使用无主引用与隐式拆包结合的方式来解决，这也是无主引用最大的应用之处，示例如下：

```javascript
class MyClassSeven{
    unowned var cls:MyClassEight
    init(param:MyClassEight){
        cls = param
    }
    deinit{
        print("ClassSeven deinit")
    }
}
class MyClassEight{
    var cls:MyClassSeven!
    init(){
        cls = MyClassSeven(param:self)
    }
    deinit{
        print("ClassEight deinit")
    }
}
var obj7:MyClassEight? = MyClassEight()
obj7=nil
```

除了在两个类实例间会产生循环引用，在闭包中，也可能出现循环引用，当某个类中包含一个闭包属性，同时这个闭包属性中又使用了类实例，则会产生循环引用，示例如下：

```objectivec
class MyClassNine {
    var name:String = "HS"
    lazy var closure:()->Void = {
        //闭包中使用引用值会使引用+1
        print(self.name)
    }
    deinit{
        print("ClassNine deinit")
    }
}
var obj9:MyClassNine? = MyClassNine()
obj9?.closure()
obj9=nil
//不会打印析构信息
```

Swift中提供了闭包的捕获列表来对引用类型进行弱引用或者无主引用的转换：

```javascript
class MyClassNine {
    var name:String = "HS"
    lazy var closure:()->Void = {
        [unowned self]()->Void in
        print(self.name)
    }
    deinit{
        print("ClassNine deinit")
    }
}
var obj9:MyClassNine? = MyClassNine()
obj9?.closure()
obj9=nil
```

捕获列表以中括号标识，多个捕获参数则使用逗号分隔。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

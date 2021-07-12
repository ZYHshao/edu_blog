---
title: Swift专题讲解十五——类型构造
date: 2016-05-19
categories: Swift语法专题
tags: []
---
## Swift专题讲解十五——类型构造

### 一、引言

        构造是类、结构体、枚举在实例化中必须执行的过程，在构造过程中，类、结构体必须完成其中存储属性的构造。Swift中的构造通过构造方法来完成，和Objective-C中的init初始化系列方法不同，Swift中的构造方法并不会也无需返回值，它的任务即是完成实例化过程。

### 二、属性的构造

        类和结构体的存储属性必须在实例化完成前被构造完成，因此，有两种方式来这么做：

1.类或者结构体中声明存储属性时直接为其设置默认值。

2.在类或者结构体的构造方法中对存储属性进行构造。

这里有一点需要注意：在存储属性设置默认值或者在构造方法中进行构造时，并不会触发属性监听器willSet、didSet方法。示例代码如下：

```
class MyClass {
    var count:Int=0{
        willSet{
            print("willset")
        }
    }
    var name:String
    init(){
        //必须进行构造或者设置值
        count=5
        count=6
        name = "HS"
    }
}
```

init()方法为不带参数的构造方法，所有构造方法都需要用init()来标识，开发者可以使用函数重载的方式来创建不同的构造方法。官方推荐，如果一个类的大多实例的某个存储属性都需要相同的值，强烈推荐开发者设置此存储属性的默认值，这样可以很好的应用Swift语言的类型推断功能并且可以使代码结构更加紧凑。

        如果一个属性在逻辑上是允许为nil的，则开发者可以将其声明称Optional值类型，在进行类的实例化时，Optional类型的属性如果没有赋值会被自动赋值为nil。

        注意，常量也需要在构造完成之前进行赋值，一旦赋值或构造完成，常量将不能被修改。

### 三、构造方法

        首先，如果类或者结构体中的所有存储属性都有默认值，那个如果开发者不提供构造方法，Swift也会自动生成一个默认构造方法，无参的init()，在进行类型的实例化时，将默认构造所有存储属性都是默认值的实例。示例如下：

```
class MyClassTwo {
    var count = 10
    var name = "HS"
}
//默认的init()构造方法
var obj = MyClassTwo()
print(obj.count,obj.name)
```

结构体会比较特殊，就算没有为其存储属性设置初值，它也会自动生成构造方法，这个构造方法中会自带所有没有赋默认值的属性名作为参数，示例如下：

```
struct Shape {
    var center:(Int,Int)
    var name:String
}
var shape = Shape(center: (1,1), name: "circle")
```

还有一点需要注意，对于值类型(结构体，枚举)，如果开发者自定义了一个构造方法，则默认的构造方法将会失效，这样设计是为了安全性考虑，防止误用到系统的默认构造方法。并且，对于值类型(结构体，枚举)的构造方法，是支持嵌套调用的，示例如下：

```
struct Shape {
    var center:(Int,Int)
    var name:String
    init() {
        center = (0,0)
        name = "HS"
    }
    init(param:String){
        self.init()
    }
}
var shape = Shape(param: "")
```

### 四、类的Designated构造方法与Convenience构造方法

        在前面的一篇博客中，我曾经专门讨论过Swift中的构造方法，博客地址如下，可供参考：

Swift中的构造方法解析：[http://my.oschina.net/u/2340880/blog/660134](http://my.oschina.net/u/2340880/blog/660134)。

        Designated构造方法也被称为指定构造方法，它是类的核心构造方法，指定构造方法将完成类中所有需要构造或赋值过程。如果一个类继承于另一个类而来，则指定构造方法需要调用父类的构造方法来完成父类中属性的初始。Convenience工作方法也被称为便利构造方法，其主要作为辅助的构造方法存在，便利构造方法需要调用类中的指定构造方法来完成构造，从这一点看，实际上类是通过便利构造方法来实现类似值类型的构造方法的嵌套使用。指定构造方法不需要多余关键字来修饰，其默认就是Designated类型的，便利构造方法需要使用convenience关键字类修饰，示例如下：

```javascript
class MyClassTwo {
    var count = 10
    var name = "HS"
    init(name:String){
        self.name = name
    }
    convenience init(count:Int){
        self.init(name:"HS")
        self.count = count
    }
}
var obj = MyClassTwo(count:5)
print(obj.count,obj.name)
```

类的构造方法需要遵守下面3条规则：

1.指定构造方法必须调用其父类的指定构造方法。

2.便利构造方法必须调用同类中的其他构造方法。

3.便利构造方法调用到最上层必须调用一个指定构造方法。

语言文档中提供如下示例图来结束指定构造方法和便利构造方法的关系：

![](http://static.oschina.net/uploads/space/2016/0519/105709_YBME_2340880.png)

### 五、构造方法的安全特性

        Swift是一种十分注重类型安全的语言，这种语言特性的优势在于类在实例化后，所有的属性都是开发者明确可控的。Swift的编译器在类的构造方法中会进行4中安全性检查：

检查1：指定构造器中必须完成所有存储属性的赋值后才能调用父类的指定构造方法，示例如下：

```javascript
class MyClassThree: MyClassTwo {
    var param:Int
    init(){
        param = 100
        super.init(name: "HS")
    }
}
```

检查2：子类如果要自定义父类中存储属性的值，必须要调用父类的构造方法之后设置，示例如下：

```javascript
class MyClassThree: MyClassTwo {
    var param:Int
    init(){
        param = 100
        super.init(name: "HS")
        //重设继承父类属性的name值
        self.name = "New"
    }
}
```

检查3：如果便利构造方法中需要重新设置某些属性的值，必须在调用指定构造方法之后设置，否则会被覆盖。

检查4：在完成父类构造方法之前，不能使用self来引用属性。

### 六、构造方法的继承

        Swift和Objective-C有很大不同，其构造方法不会被子类无条件的继承。Swift中类的构造方法的继承遵守下面两个原则：

1.如果子类没有定义任何的指定构造方法，则子类会默认继承父类所有的指定构造方法。

2.如果子类中提供了父类所有指定构造方法，无论是覆写的还是继承的，则子类会默认继承下来父类的便利构造方法。

上面两个原则可能有些难以理解，第1个原则实际上也说明子类如果定义了自己的指定构造方法，或者覆写了父类的某个指定构造方法，则子类不再继承父类所有的指定构造方法。第2个原则可以这样理解：因为所有便利构造方法最终都要调用到指定构造方法，所以只要子类中有提供这个便利构造方法需要调用的指定构造方法，这个便利构造方法就会被继承。

        重写父类的指定构造方法需要使用override关键字，但是，便利构造方法并不存在重写的概念，因为其必须调用本类的其他构造方法，因此无论子类中定义的便利构造方法与父类是否相同，都是子类独立的便利构造方法。

### 七、可失败构造方法

        在开发中还会遇到一种情况，某些构造方法需要传入一些参数，当参数不符合要求时，此构造过程可能会失败，这时，开发者可以使用可失败的构造方法来进行类型的构造，例如在类中创建可失败的构造方法示例示例如下：

```javascript
class MyClassThree: MyClassTwo {
    var param:Int
    init(){
        param = 100
        super.init(name: "HS")
        //重设继承父类属性的name值
        self.name = "New"
    }
    init?(suc:Bool){
        guard(suc)else{
            return nil
        }
        param=1
        super.init(name: "1")
    }
}
```

### 八、必要构造方法

        如果某些构造方法是类与其子类都必须实现的，则可以使用required关键字来将其修饰为必要的构造方法，子类必须继承或者覆写父类的必要构造方法，示例如下：

```javascript
class MyClassThree: MyClassTwo {
    var param:Int
    required init(){
        param = 100
        super.init(name: "HS")
        //重设继承父类属性的name值
        self.name = "New"
    }
    init?(suc:Bool){
        guard(suc)else{
            return nil
        }
        param=1
        super.init(name: "1")
    }
}
```

还有一点需要注意，如果某些属性的值设置十分复杂，开发者可以使用闭包的方式来为属性设置初始值，示例如下：

```javascript
class MyClassThree: MyClassTwo {
    //注意 闭包的后面必须加()，否则会将param当成一个闭包属性来处理
    var param:Int = {
        return 6*6+6
    }()
    required init(){
        param = 100
        super.init(name: "HS")
        //重设继承父类属性的name值
        self.name = "New"
    }
    init?(suc:Bool){
        guard(suc)else{
            return nil
        }
        param=1
        super.init(name: "1")
    }
}

```

### 九、析构方法

        当类实例将要被释放时，系统会自动调用类的析构方法，析构方法deinit()没有参数和返回值，并且只有类有析构方法，开发者可以在其中进行一些资源的释放操作，当var类型变量被赋值为nil时，实例会被释放。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: 【Swift专题】聊聊Swift中的属性
date: 2024-01-31
categories: Swift语法专题
tags: []
---
# 【Swift专题】聊聊Swift中的属性

## 引言

属性是面向对象语言中非常基础的语法特性，我们讲属性，实际上就是讲与类本身或类实例关联的数据。在面向对象的语言中，类作为重要的数据结构会封装数据与函数，类中的函数我们通常称其为方法，而数据则就是属性。

Swift语言是一门比较现代化的语言，并且直到今日，其还在不断进行语法特性与编程模式的更新。学习Swift语言不仅能够进行实用的编程，从其设计思想和许多语法定义细节上我们也可以受益匪浅。就好比读一本内容深厚的文学作品，它会启发你的思考，对编程的设计和应用有更深的理解。

本文将以”属性“为专题介绍Swift语言中相关功能的设计与应用。如果你正在寻找这部分的内容与知识，希望本文可以带给你帮助。

## 进入正题

和大多数编程语言一样，Swift语言中的属性也分为存储属性（stored）与计算属性（computed）。属性可以关联在类本身上，也可以关联在类的实例上，当然，这里说”类“并不准确，属性也适用于结构体和枚举。存储属性顾名思义会存储数据，通常大多数属性也都是以存储属性的方式定义。计算属性则更像是一个方法，其定义的是一个计算过程，计算属性本身并不存储任何数据，通常计算属性会用于二次处理其他存储属性的值。在Swift中，计算属性可以在_**类**_、_**结构体**_和_**枚举**_中定义，而存储属性只允许在**_类_**和_**结构体**_中定义。

### 存储属性

存储属性定义在类或结构体中，可以将存储属性定义为常量也可以定义为变量。在Swift语言中，类是引用类型和结构体是值类型，因此如果结构体实例被定义成了常量，则无论其中的存储属性是否是变量，都将不可修改，类则不同。

```swift
class ClassDemo {
    var value:Int = 0
}
struct StructDemo {
    var value:Int
}

let c = ClassDemo()
let s = StructDemo(value: 1)

c.value = 2
// 结构体常量不允许任何修改
// s.value = 3

```

上面代码中，虽然c类定义成了常量，但由于引用类型的性质，我们依然可以对其中定义为变量的存储属性进行修改。

#### 关于懒加载

在Objective-C语言中，如果我们想让某个属性在使用时再创建，可以手动为其实现Setter方法。Swift语言则方便很多，只需要使用Lazy关键字来修饰存储属性即可，懒加载是一种很实用的编程技巧，我们再设计某个类型时，如果其中某个属性并不是必须的，就可以将其设置为懒加载属性，这样只有当真正使用到此属性时，才会进行属性的创建，这会减轻实例初始化时的负担。

```swift
class ClassDemo {
    var value:Int = 0
    
    lazy var description: String = {
        print("懒加载属性创建")
        return "ClassDemo description"
    }()
}
```

上面的代码，只有当实例调用到description属性时，才会打印出创建信息。直观上看，懒加载属性的定义更像是定义了一个属性的值的构造方法，第一次用到时才会构造。上面的例子其实并不明显，如果我们某个属性的值是需要读文件来获取的，则使用懒加载可以大大提高实例创建的性能。

另外，Lazy只能修饰定义为变量的属性，不能修饰常量属性，这是因为懒加载的本身逻辑是与Swift常量属性的性质相悖的，Swift中的常量属性必须在实例构造好前完成初始化，而懒加载的属性是允许实例构造完成后属性并未初始化的。Lazy关键字虽然好用，但是其并不是线程安全的，如果在多个线程中访问懒加载属性，则其有可能会被初始化多次，造成难以预料的异常问题。

### 计算属性

与存储属性对应，计算属性并不真正的存储数据，而是提供一种计算算法，直接将计算出的结果作为计算属性的值。

```swift
struct StructDemo {
    var value:Int
    
    var exp: Int {
        set(newValue) {
            value = newValue / 2
        }
        get {
            value * 2
        }
    }
}
let s = StructDemo(value: 2)
print(s.exp) // 4
```

上面示例代码中，exp是一个计算属性，用来对value的值乘以2，从使用上看，计算属性和存储属性并没有太大差别，当对计算属性进行赋值时，会调用其中的set代码块，当读取计算属性的值时，会调用其中的get代码块。一个计算属性也可以只提供get而不提供set，这样的计算属性与只读存储属性类似，就只能读取不能赋值了。

#### 计算属性的简化写法

Swift语言的设计理念是极简的，简单层面的简化可以更聚焦逻辑，但同时也会带来一些弊端，极致的简化需要靠大量的语法静态约定来支持，这就需要开发者额外记忆一些约定，因此Swift为开发者提供了简写与非简写两种编码方式。我们可以根据自己的编程习惯以及业务的特性来选择。对于计算属性，set部分的形参是可以省略的，如果省略形参，则约定形参的名字就是newValue，例如如下的简写方式：

```swift
var exp: Int {
    set {
        value = newValue / 2
    }
    get {
        value * 2
    }
}
```

另外，如果只提供get块，则可以直接省略任何声明，直接做get计算即可：

```swift
var exp: Int {
    value * 2
}
```

需要注意，上面的示例是一种比较极端的情况，当只有一句计算语句时可以对return关键字进行省略。

### 类属性

前面提到的存储属性和计算属性都是实例属性，实例属性通过类实例进行访问，我们也可以直接将属性关联到类型上，这时定义的属性为类属性。类属性使用关键字static或class进行定义。static定义的类属性不能被子类覆写，如果需要定义子类和覆写的类计算属性，则需要使用class关键字。类属性直接使用类名来访问，其性质上和实例属性并没太大差别。

### 属性监听器

属性监听器提供了一种监听属性变化的方法，每当属性被赋值时，都会调用监听器回调，另外，如果赋值前后属性的值并没有变化，监听器依然是生效的，回调依然会正常执行。

```swift
class ClassDemo {
    var value:Int = 0
    
    lazy var description: String = {
        print("懒加载属性创建")
        return "ClassDemo description"
    }(){
        didSet {
            print("属性监听：didSet")
        }
        willSet {
            print("属性监听：willSet - \(newValue)")
        }
    }
    
    var exp: Int {
        get {value * 2}
        set {value = newValue / 2}
    }
}
```

其中，didSet会在属性赋值完成后回调，这是再取属性的值已经是赋值后的结果，willSet会在属性赋值前调用，willSet中也会自动传入一个newValue参数，它就是将要被赋值的数据值。

并非所有的场景都支持定义属性监听器，能够定义属性监听器的场景有：

1\. 类中定义的存储属性。

2\. 子类继承的存储属性。

3\. 子类继承的计算属性。

需要注意的是当前类中定义的计算属性并不能定义属性监听器，这很好理解，因为即使支持在这种场景定义属性监听器也没有任何意义，因为set块在调用时我们已经可以处理任何需要监听器处理的逻辑。上面的ClassDemo，子类继承示例如下：

```swift
class SubClassDemo: ClassDemo {
    override var value:Int {
        didSet {}
        willSet {}
    }
    
    override var exp: Int {
        didSet {}
        willSet{}
    }
}

```

### 属性包装器

属性包装器是Swift语言中有关属性部分非常强大的功能。我们知道，通过定义计算属性可以定义内部属性的存储方式，如果我们想让这一部分计算逻辑能够复用，例如前面示例代码中的对数据乘2的操作，使用属性包装器就非常方便。例如：

```swift
@propertyWrapper
struct MultipleTwo {
    private var number = 0
    
    var wrappedValue: Int {
        get { number * 2}
        set {number = newValue}
    }
}

```

使用**@propertyWrapper**可以将某个结构定义成属性包装器，属性包装器中通常会定义一个私有的存储属性存储本质的数据，wrappedValue计算属性用来提供外界访问的数据。在定义普通的存储属性时，可以使用包装器对其进行包装，其使用起来就会和包装器中wrappedValue逻辑一致，例如：

```swift
struct StructDemo {
    @MultipleTwo var exp: Int
}
var s = StructDemo()
// 赋值为2
s.exp = 2
// 实际上访问到了包装器的get，返回4
print(s.exp) // 4
```

属性包装器中存储的属性也支持通过初始化方法来设定初值，例如：

```swift
@propertyWrapper
struct MultipleTwo {
    private var number: Int
    
    var wrappedValue: Int {
        get { number * 2}
        set {number = newValue}
    }
    
    init(number: Int) {
        self.number = number
    }
}
struct StructDemo {
    @MultipleTwo(number: 2) var exp: Int
}
var s = StructDemo()
// 赋值为2
s.exp = 2
// 实际上访问到了包装器的get，返回4
print(s.exp) // 4
```

属性包装器在实际项目开发中是非常有用的，例如我们可以编写一个持久化存储的包装器，当属性被赋值时，自动的将数据同步到文件。

还有一点需要注意，一般情况下，我们无需访问属性包装器中真实存储数据的存储属性，但Swift语言也提供了一种方式来访问此属性的值，仍然是通过语法规范约定的方式，只需要将属性包装器中存储属性的属性名定义为projectedValue，之后即可使用属性名加$前缀的方式来访问，如下：

```swift
@propertyWrapper
struct MultipleTwo {
    private(set) var projectedValue: Int
    
    var wrappedValue: Int {
        get { projectedValue * 2}
        set {projectedValue = newValue}
    }
    
    init(number: Int) {
        self.projectedValue = number
    }
}
struct StructDemo {
    @MultipleTwo(number: 2) var exp: Int
}
var s = StructDemo()
// 赋值为2
s.exp = 2
// 实际上访问到了包装器的get，返回4
print(s.exp) // 4
// 访问到真实存储的数据，返回2
print(s.$exp) // 2
```

另外，上述的属性监听器和包装器其实也适用于变量中，本篇文章不再过多介绍。
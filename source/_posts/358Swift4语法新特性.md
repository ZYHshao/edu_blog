---
title: Swift4语法新特性
date: 2017-08-24
categories: Swift语法专题
tags: []
---
## Swift4语法新特性

      随着iPhone X的来到，iOS11的发布，Swift语言也更新到了第4个版本。在Swift4中，无论是代码风格还是编程理念都更进一步的融合了许多现代编程的思想。对于熟悉传统语言的开发者来说(尤其是Objective-C、Java和C++)，可能会感觉这些特性并没有多大的价值反而非常不习惯，但是我们依然可以茶余饭后(没事干的时候)，一窥Swift4语言的玩法，体验一下Swift语言的设计思想和编码风格。

### 一、独占内存访问权限

    独占访问权限是Swift4中引入的一大新特性。然而大部分人都将这一特性误解了，如果你在百度上搜索 swift4 exclusive access to memory相关关键字，大部分博客或总结都会说这是一种编译器的编译时特性，可以在例如数组越界时、对遍历中的数组进行删添元素时产生编译异常。其实并非如此，独占内存访问权限特性是一种编译时和运行时的安全特性，其和数组也没有任何关系，当两个变量访问同一块内存时，会产生独占内存访问限制。

    首先，在Swift中对内存的访问有读访问与写访问两种，例如：

```swift
//读访问
var name = "jaki"
//写访问
print(name)
```

在Swift4以前，程序对内存的读写访问并没有严格的控制，如果你在读内存时有写内存操作，或者写内存时有读操作并不会产生什么异常(当然，你自己要清楚读写后变量的值，以免产生逻辑歧义)。Swift4中则引入了独占内存访问权限的特性，如果复合如下3个条件，则程序会产生读写权限冲突：

1.至少有一个变量在使用写权限。

2.变量访问的是同一个内存地址。

3.持续时间有重叠。

    在开发中，可能会产生读写权限冲突的情况有3种：

#### 1.inout 参数读写权限冲突

####     一般情况下，值类型的传参总会产生复制操作。inout参数则使得函数内可以直接修改外部变量的值。inout参数是最容易产生读写冲突的场景，例如下面的代码：

```swift
var stepSize = 1

func increment(_ number: inout Int) {
    number += stepSize//crash
}

increment(&stepSize)
```

上面的代码在Swift3中没有任何问题，在Swift4环境中运行则会直接crash。在函数中，inout参数从声明开始到函数的结束，这个变量始终开启着写权限，对应上面代码，number参数开启这写权限，stepSize则进行了读访问，如此则满足上面的权限冲突规则，会产生读写冲突。同样，如果对两个inout参数访问同一个内存地址，也会产生读写权限冲突，例如：

```swift
var stepSize = 1

func increment(_ number: inout Int,_ number2: inout Int) {
   var a = number+number2
}

increment(&stepSize,&stepSize)
```

#### 2.结构体中自修改函数的读写冲突

    Swift语言中的结构体也是一种值类型，因此其也存在读写冲突的场景，例如如下代码：

```swift
struct Player {
    var name: String
    var health: Int
    var energy: Int
    
    let maxHealth = 10
    mutating func shareHealth(_ player:inout Player) {
        health = player.health
    }
}
var play = Player(name: "jaki", health: 10, energy: 10)
play.shareHealth(&play)//产生错误
```

上面shareHealth函数中使用到的health是对self自身的读访问，而inout参数是写访问，会产生读写权限冲突。

#### 3.值类型中属性的读写访问权限冲突

    在Siwft语言中，像结构体，枚举和元组中都有属性的概念。由于其都是值类型，在对不同的属性进行访问时也会产生冲突，例如：

```swift
class Demo {
    var playerInformation = (health: 10, energy: 20)
    func balance(_ p1 :inout Int,_ p2 :inout Int) {
        
    }
    func test()  {
        self.balance(&playerInformation.health, &playerInformation.energy)//crash
    }
}
let demo = Demo()
demo.test()
```

看到这里你一定觉得这太严格了，对不同属性的访问也会产生读写冲突。实际上，在开发中大部分的这种访问都会被认为是安全的，你需要满足下面3个条件：

1.你访问的是存储属性而不是计算属性。

2.你访问的是结构体局部变量(函数中的变量)而不是全局变量。

3.你的结构体不被闭包捕获，或者只是被非逃逸的闭包捕获。

将上面的playerInformation变量修改成局部的，程序就可以正常运行了：

```swift
class Demo {
   
    func balance(_ p1 :inout Int,_ p2 :inout Int) {
        
    }
    func test()  {
         var playerInformation = (health: 10, energy: 20)
        self.balance(&playerInformation.health, &playerInformation.energy)
    }
}
let demo = Demo()
demo.test()
```

其实，Swfit4中的独占内存访问权限特性一般情况下我们都不会使用到，但是了解一下还是很有必要，Swift是一种安全性极高的语言，也是其设计的核心思想与方向，例如类构造方法的安全性检查特性，变量类型的安全限制特性等等都是将开发者编写代码的安全交给语言特性来负责，而不是开发者的经验。这让初学者可以更少的出错，语言运行时的不可控因素更少。

### 二、关联类型可以添加where约束子句

    associatedtype是Swift协议中一个很有用的关键字，其也是Swift泛型编程思想的一种实现。在Swift3中，associatedtype从语法上是不能追加where子句的，Swift4增强了associatedtype的功能，其可以使用where子句进行更加精准的约束，看下面的代码：

```swift
//容器协议
protocol Container {
    //约束item 泛型为 Int类型
    associatedtype Item where Item == Int
    func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}

class MyIntArray: Container {
    //这个地方必须指定为Int否则会报错
    typealias Item = Int
    func append(_ item: Int) {
        self.innerArray.append(item)
    }
    
    var count: Int{
        get{
            return self.innerArray.count
        }
    }
    
    subscript(i: Int) -> Int {
        return self.innerArray[i]
    }
    
    var innerArray = [Int]()
    
}
```

### 三、可以创建多行字符串

    在Swift4以前，字符串只能创建单行的，Swift4中引入了字面量创建多行文本的语法，例如：

```swift
var multiLineString = """
abcd
jaki
24
"""
print(multiLineString)
```

这种方式可以大大减少在创建字符串时人为添加换行符。

    关于String操作的相关API，在Swift4中也有许多优化，例如字符串的下标操作与字符操作一直是Swift语言的硬伤，使用起来十分麻烦，在Swift4中都进行了优化。取字符串的子串的方式也更加规范。

### 四、增强区间运算符

    Swift语言中的区间运算符使用起来十分方便，例如在Swift3中，我们若要遍历数组的范围，可以使用如下的代码：

```swift
//Swift3代码
let array = ["1","2","3"]
for item in array[0..<array.count]{
    print(item)
}
```

Swift3中的...运算符只是作为闭区间运算符使用，在Swift4中，可以用它来取集合类型的边界，如字符串，数组等，看如下代码：

```swift
let array = ["1","2","3"]
for item in array[0...]{
    print(item)
}
```

### 五、下标方法支持泛型

    subscript方法可以为Swift中的类添加下标访问的支持，在Swift4中，subscript方法更加强大，其不只可以支持泛型，而且可以支持where子句进行协议中关联类型的约束，示例如下：

```swift
//下标协议
protocol Sub {
    associatedtype T
    func getIndex()->T
}
//实现下标协议的一种下标类
class Index:Sub {
    init(_ index:Int) {
        self.index = index
    }
    var index:Int
    func getIndex() -> Int {
        return self.index
    }
}

class MyArray {
    var array = Array<Int>()
    func push(item:Int)  {
        self.array.append(item)
    }
    //泛型 并进行约束
    subscript<T:Sub>(i:T)->Int where T.T == Int {
        return self.array[i.getIndex()]
    }
}

var a = MyArray()
a.push(item: 1)
print(a[Index(0)])
```

### 六、协议支持混合

    Swift在对变量类型进行界定时，是支持使用协议的，例如，在Swift3中，我们可以编写如下的代码：

```swift
//swift3
protocol People {
    var name:String{set get}
    var age:Int{set get}
}

protocol Teach {
    func teachSwift()
}

class Teacher: People,Teach {
    var name: String = "jaki"
    
    var age: Int = 25
    
    func teachSwift() {
        print("teaching...")
    }
}

func printTeacher(p:Teacher)  {
    print(p.name,p.age)
    p.teachSwift()
}
```

上面的代码中，printTeacher方法里使用Teacher类对参数进行的界定，实际上这种做法并不好，Teacher类知识Teach协议与People协议的一种混合实现，在定义方法参数时，应该使用协议来进行参数的界定，可是Teacher类同时实现了两个协议，这在Swift3版本中是无法解决的问题，在Swift4中你则可以这样写：

```swift
protocol People {
    var name:String{set get}
    var age:Int{set get}
}

protocol Teach {
    func teachSwift()
}

class Teacher: People,Teach {
    var name: String = "jaki"
    
    var age: Int = 25
    
    func teachSwift() {
        print("teaching...")
    }
}

func printTeacher(p:Teach&People)  {
    print(p.name,p.age)
    p.teachSwift()
}
```

&复合可以对协议进行混合，更加贴近面向协议的编程方式。

### 七、一点总结

    从Swift语言第1个版本发布到Swift3和Swift3.2进行了语言内容和风格的大改，Swift4中进行的改动实际并不大而且大多是你开发中可能并用不到的特性。随着Swift语言的成长，这种语言的设计风格是与其他传统语言有着本质的区别，我个人感悟，Swift语言如下的特点确实值得我们学习与思考：

#### 1.安全性极高

    所谓安全性，实际上就是语言是否容易出错，再通俗一些，即是一种编程语言是依赖其自身特性防止其出错还是依赖开发者经验防止其出错。我记得在初学JavaScript时感觉十分苦恼，因为JavaScript是变量弱类型的，并且其隐式转换十分危险(虽然代码编写起来畅快无比)。在Swift中，则基本不会出现类型不匹配，类型被隐式转换了等问题。当然，换句话说，这也使得编程者必须遵守更多的规则(或者说写更多的代码)，虽然各有利弊，但对初学者来说，Swift明显要友好很多。

    Swift语言安全性极高表现在如下几点：

1.用let和var来分别声明常量和变量，let声明的量值不可改，从逻辑上保证变量安全。

2.变量类型必须明确(很多时候你没指定是因为编译器的推断功能)，从类型上保证安全。

3.闭包分为逃逸和非逃逸，从逻辑上保证闭包使用的安全。

4.溢出运算符与算术运算符分开，从代码上保证安全。

5.类的初始化检查策略，从类的定义上保证安全。

6.删除++与--运算符，删除常规for循环，从习惯上保证安全。

#### 2.灵活性极高

    Swift语言的灵活性非常有现代编程语言的特点，有其是其对泛型的支持，是的面向协议的编程方式在Swift语言上可以畅行无阻。灵活性表现在如下几点：

1.强大的泛型编程方式，协议关联类型等。

2.where子句可以精准的进行泛型约束。

3.Optioal类型和可失败构造方法的支持。

4.Any与AntObject类型的支持。

5.强大的枚举和结构体。

6.递归枚举的支持。

7.支持重载与自定义运算符。

#### 3.编码体验极高

    编码体验这点并不完全依赖与Swift语法，也多有编译器的功劳。

1.支持字符串内嵌变量来构建字符串。

2.支持后置闭包的写法。

3.元组类型的支持。

4.支持默认隐式拆包类型。

5.支持区间运算符。

6.函数分内外两种参数名(外参数名可以省略)。

7.语法上支持便利构造方法。

8.语法层面支持的懒加载。

    上面只是列出了一些特性，Swift语言中有意思的地方多的举不胜举，如果你有意更深入的了解它，你可以搜索清华大学出版社的《Swift从入门到精通》一书，其中是我对Swift3进行的全面讲解，也包含iOS开发的部分知识和实战，你也可以直接通过QQ316045346联系我本人进行交流与互相学习。最后，一语以总结Swift语言：一门十分强大并且十分易入门的现代编程语言，只要你掌握了所有语法规则，想出错很难！

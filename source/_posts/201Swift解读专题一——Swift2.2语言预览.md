---
title: Swift解读专题一——Swift2.2语言预览
date: 2016-05-05
categories: Swift语法专题
tags: []
---
## 专题一——Swift2.2语言预览

### 一、引言

        本系列专题是我通过阅读Swift2.2语言开发文档，翻译总结加上自己的理解整理而成。其中大部分结构和内容都来自开发文档，有疏漏和错误之处，还望更多朋友指出，共同交流进步，我的QQ：316045346。

### 二、从HelloWorld开始

        在学习很多编程语言时，都是从HelloWorld入门，下面代码就是一个完整的HelloWorld程序：

```
print("Hello, World!")
```

分析上面代码，可以发现Swift语言的3个十分明显的特点：

1.开发者不需要引入输入输出相关的函数库。

2.在编写代码时，不需要在语句的结尾处添加分号。

3.全局的代码就是程序的入口，不需要类似C系语言的main()方法来作为程序入口。

### 三、常量与变量

        常量和变量是编程语言中最基础的两类数据类型，常量可以理解为为某个值起一个特定的名字，常量通常提供给开发者用于某些只赋值一次但却在程序中多处使用的量值。变量也可以进行多次修改。分别使用let和var创建常量和变量。例如：

```
let letValue = 4
var varValue = 8
varValue = 16
```

开发者在进行常量和变量的创建时，并不需要制定类型，编译器与根据第一次赋值的类型来推断出常量或者变量的类型，然而这并不是说Swift语言不严格要求变量或常量的类型，一旦编译器推断了值的类型，之后开发者若要修改变量，则必须严格遵守既定的变量类型，否则编译器会报错。

        如果开发者第一次对变量或常量进行的赋值不能够使编译器正确的推断出常量或变量的类型，开发者也可以通过冒号后跟类型的方式来强制定义变量或常量的类型，如下：

```
var varValue:Float = 8
varValue = 16.0
```

在Swift语言中，不存在隐式转换的概念，这也是Swift语言更加安全的特性之一，这样的设计可以保证变量在任何时候类型都被明确的指定。在进行类型转换时，可以通过类实例化的方式进行，示例如下：

```
//Float值转成Int
letValue+Int(varValue)
//Int转为Float
Float(letValue)+varValue
```

对于在字符串中使用其他类型的变量，Swift语言提供了一种更加便捷的写法，使用\\()的方式来转换，小括号内为变量的名称，例如：

```
var strValue = "Hello"
//Hello16.0
strValue+"\(varValue)"
```

### 四、数组与字典

        数组与字典是最常用的两种数据集合，在Swift语言中，使用\[\]来创建数组或字典，示例如下：

```
var array = [1,2,3]
var dic = [1:"one",2:"two",3:"three"]
```

同Int，Float类型的数据一样，数组和字典在第一次赋值时，也会根据赋值的类型来推断出变量类型，开发者同样也可以强制指定，如下：

```
var array:[Int] = [1,2,3]
var dic:[Int:String] = [1:"one",2:"two",3:"three"]
```

Swift允许创建或者重新赋值为空的数据或者字典，但是这有一个前提条件，被赋值为空的数据或字典必须是类型确定的，示例如下：

```
//这样写会报错
//var errorArray = []
//创建空的数据集合
//方式一
var array:[Int] = []
var dic:[Int:String] = [:]
//方式二
var array2 = [Int]()
var dic2 = [Int:String]()
//方式三
var array3 = [1]
var dic3 = [1:"1"]
array3 = []
dic3 = [:]
```

### 五、optional类型的值

        在理解optional类型的值之前，我们可以先来看一段C代码：

```
int a=1;
if(a){
    
}else{
    
}
```

上面这段代码对于C语言来说完全没有问题，当a为非0值时，就代表条件为真，在Swift语言中则不同，if选择语句中的条件必须为Bool类型的值，因此，对于某些可以为空的值，Swift中提供了optional类型，这种类型相当于对其他实际类型进行了包装，如果有值，则他拆包后为相应类型的值，如果没有值，则为空值nil。示例如下：

```
var optionalString: String? = "Hello"
if optionalString == nil {
    
}
```

在Swift中，当if与let共同使用时，将会构成一种更加奇特的语法方式，这种方式对于处理optional类型的值十分方便，示例如下：

```
/*
 if let 后面赋值为optional类型的值有这样的效果
 如果optional的值不为nil 则会走if条件为真的语句块并且将optional变量的值赋值给let常量 可以在if为真的语句块中使用
 如果optional的值为nil 则会走else语句块 并且name常量被释放 不能再else块中使用
*/
if let name=optionalName {
    greeting = "Hello, \(name)"
}else{
    print(greeting)
}
```

除了if let语法外，还有一种方式可以用来处理optional类型的值，示例如下：

```
var greeting = "Hello!"
greeting = "Hello" + (optionalString ?? "")
```

??运算符用来为optional类型的值设置一个默认值，如果optional值为nil，则会使用后面设置的默认值来代替。

        Swift语言的switch语句相比于C系的语言要强大的多，其不只可以用于判断整型，其可以处理任意类型的数据，同样，它也不只限于比较是否相等的运算，其可以支持各种负责运算，示例如下：

```
let vegetable = "red pepper"
switch vegetable {
case "celery":
    print("Add some raisins and make ants on a log.")
case "cucumber", "watercress":
    print("That would make a good tea sandwich.")
case let x where x.hasSuffix("pepper"):
    print("Is it a spicy \(x)?")
default:
    print("Everything tastes good in soup.")
}
```

如果匹配上了一个case，程序会结束switch选择，各个case之间是互斥的。

### 六、循环语句

        Swift2.2中，弃用了for i；param;param{}格式的循环语句，提供给开发者使用的循环语句主要有3种。

1.for in语句

for in语句多用于快速遍历字典，示例如下：

```
let interestingNumbers = [
    "Prime": [2, 3, 5, 7, 11, 13],
    "Fibonacci": [1, 1, 2, 3, 5, 8],
    "Square": [1, 4, 9, 16, 25],
]
var largest = 0
for (kind, numbers) in interestingNumbers {
    for number in numbers {
    //找出最大值
        if number > largest {
            largest = number
        }
    }
}
print(largest)
```

在for in循环中可以使用一个索引来指定循环次数，通过这种方式可以实现有序的遍历操作，示例如下：

```
for i in 0..<10 {
    print(i)
}
```

2.while语句

while语句用于条件循环，直到不再满足某个条件为止，示例如下：

```
var n = 2
while n < 100 {
    n = n * 2
}
print(n)
```

3.repeat {}while语句

repeat{}while语句与C语言中的do{}while作用相同，保证至少循环一次。示例如下：

```
var m = 2
repeat {
    m = m * 2
} while m < 100
print(m)
```

### 七、函数与闭包

        Swift中的函数使用关键字func来标识，格式如下：

func name(param1,param2...)->returnValue{}

示例代码如下：

```
func add(param1:Int,param2:Int) -> Int {
    return param1+param2
}
//下面表达式将返回8
add(5, param2: 3)
```

我比较了Swift语言与Objective-C、Java语言的函数特点：

        Objective-C实际上并没有函数重载的概念，不同参数的函数实际上拥有不同的函数名，Objective-C的风格将参数名嵌套进函数名中，这样有一个好处，开发者可以通过函数名明确的知道此函数的用途以及每个参数的意义，当然也有其局限性，Objective-C的函数大多十分冗长，不够简洁。

        Java不同参的函数采用重载的方式，这样的效果是，相同的函数名，参入不同的参数则会执行不同的操作，是不同的两个方法，这样的有点是使代码十分简洁，然而对开发者来说并不友好，开发者在开发时不能便捷的看出每个参数的意义和用法。

        个人见解，Swift对函数的设计综合了上面两种语言的有事，参数列表与函数名分离，简化了函数，同时，参数列表中保留了每个参数的名称，使开发者在调用函数时更加直观。

        在Objective-C中，如果需要某个函数返回一组值，开发者通常会需要使用字典或者数组，这样做有一个问题，在调用此函数时，返回值的意义十分模糊，开发者需要明确的知道其中数据的顺序与意义。Swift中可以采用返回元组的方式来处理一组返回值，示例如下：

```
//返回一组数据的函数
func calculateStatistics(scores: [Int]) -> (min: Int, max: Int, sum: Int) {
    var min = scores[0]
    var max = scores[0]
    var sum = 0
    
    for score in scores {
        if score > max {
            max = score
        } else if score < min {
            min = score
        }
        sum += score
    }
    
    return (min, max, sum)
}
//元组数据
let statistics = calculateStatistics([5, 3, 100, 3, 9])
//通过名称取元组中的最大值
print(statistics.max)
//通过角标取元组中的最小值
print(statistics.0)
```

        对于可变参数个数的函数，在Objective-C中，开发者大多会采用va_list指针的方式实现，示例如下：

```
-(void)myLog:(NSString *)str,...{//省略参数的写法
    va_list list;//创建一个列表指针对象
    va_start(list, str);//进行列表的初始化，str为省略前的第一个参数，及...之前的那个参数
    NSString * temStr = str;
    while (temStr!=nil) {//如果不是nil，则继续取值
         NSLog(@"%@",temStr);
         temStr = va_arg(list, NSString*);//返回取到的值，并且让指针指向下一个参数的地址
    }
    va_end(list);//关闭列表指针
}
```

在Swift语言中，实现这样的函数要简单的多，通过...来进行参数的省略，并且将这些省略的函数包装为数组传入函数内部，示例如下：

```
func sumOf(numbers: Int...) -> Int {
    var sum = 0
    //多参被包装为数组
    for number in numbers {
        sum += number
    }
    return sum
}
sumOf()
sumOf(42, 597, 12)
```

与Java类似，Swift中的函数也支持嵌套操作，嵌套内部的函数可以使用外部的变量，示例如下：

```
func returnFifteen() -> Int {
    var y = 10
    //嵌套函数
    func add() {
        y += 5
    }
    //调用
    add()
    return y
}
returnFifteen()
```

由于函数也是一种特殊的数据类型，函数也可以作为返回值，示例如下：

```
func makeIncrementer() -> ((Int) -> Int) {
    func addOne(number: Int) -> Int {
        return 1 + number
    }
    return addOne
}
var increment:((Int)->Int) = makeIncrementer()
increment(7)
```

一个函数也可以作为另一个函数的参数来使用，示例如下：

```
//参数中有函数
func func1(param1:Int,param2:(count:Int)->Void) {
    param2(count: param1+1)
}
func tmpFunc(count:Int) -> Void {
    print(count)
}
//将函数作为参数传入
func1(3, param2: tmpFunc)
```

与Objective-C中的block对应，Swift中有闭包的概念来创建一个代码块，可以理解为闭包为没有名字的函数，使用{()in }格式来创建闭包，示例代码如下：

```
var f:(count:Int)->Void = {(Count) in print(132) }
f(count:0)
```

通过这种写法，开发者在将函数作为参数传递时，无需再创建中间函数，示例如下：

```
//参数中有函数
func func1(param1:Int,param2:(count:Int)->Void,param3:(count:Int)->Void) {
    param2(count: param1+1)
}
func1(3, param2: { (count) in
        print(count)
    }, param3: { (count) in
        print(count)
})
```

还有一种更加简单的闭包书写方法，如果闭包类型是确定的，全完可以省略小括号中的参数名称与闭包格式in，使用角标来获取参数，示例如下：

```
//优化前
var f:(a:Int,b:Int)->Bool = {(a,b) in return a>b}
f(a: 3,b: 4)
//优化后
var f:(a:Int,b:Int)->Bool = {$0>$1}
f(a: 3,b: 4)
```

### 八、类与属性

        Swift中使用class关键字来定义类，类内部可以声明与定义一些属性与方法，类的实例对象可以通过点语法来调用类的属性和方法，示例如下：

```
class MyClass {
    var count = 100
    let name = "珲少"
    func run() {
        print("run 100 miter")
    }
}
var obj = MyClass()
let count = obj.count
let name = obj.name
obj.run()
```

类名加括号用于创建类的实例对象，可以通过重写init方法来重写类的默认构造方法，如果这个类有继承的父类，则需要遵守如下3条规则：

1.必须先将子类的属性初始化完成。

2.调用父类的构造方法。

3.修改父类需要修改的属性。

        在Swift中同样也有set和get方法，只是这里的set和get方法与Objective-C中的set和get方法有很大的不同，Objective-C中的get和set方法是截获了属性和存取过程，在其中加入额外的其他操作，Swift中的set和get方法原理上将属性的存取与其他逻辑操作进行了分离，抽象出了一种计算属性，示例如下：

```
class MyClass {
    var count:Int
    //实际上并不存在privateCount属性 通过pricatecount来操作count的值
    var privateCount:Int{
        get{
            return count;
        }
        set {
           count=newValue+100
        }
    }
    let name = "珲少"
    func run() {
        print("run 100 miter")
    }
    init(){
        count=200
    }
}
```

Swift采用这样的设计思路也有其一定的优化道理，我比较了一下，给大家举一个最简单的例子，在使用Objective-C进行iOS开发时，经常会遇到这样的情况，某个控件中有一个UILabel控件，开发者在不想将控件暴漏在.h文件中的情况下经常会声明一个NSString类型的变量，重写此变量的set方法来完成对UILabel控件的赋值，仔细想来，实际上声明的这个NSString变量完全是多余的，它只是为了用来做中间值得传递，Swift的set和get方法就在这里进行了优化。另外，在set方法中会自动生成一个命名为newValue的变量作为传递进来的值，开发者也可以自定义这个变量的名称，在set后加小括号即可，示例如下：

```
var privateCount:Int{
        get{
            return count;
        }
        set(myValue) {
           count=myValue+100
        }
    }
```

        Swift中也提供了监听属性赋值过程的方法，其使用的是willSet与didSet机制，示例如下：

```
class MyClass {
    var count:Int{
        //赋值前执行(除了第一次初始赋值) 将要赋值的值以newValue传入
        willSet{
            print("will set \(newValue)")
        }
        //赋值后执行(除了第一次初始赋值) 原来的值以oldValue传入
        didSet{
           print("did set \(oldValue)")
        }
    }
    let name = "珲少"
    func run() {
        print("run 100 miter")
    }
    init(){
        count=200
    }
}
```

### 九、枚举和结构体

        Swift中的枚举和C与Objective-C有很大的差别，在Swift中，枚举也被作为一种数据类型来处理，其中可以添加函数方法。最基本的枚举用法如下所示：

```
//枚举可以多个case并列 也可以写在一个case中以逗号分隔
enum MyEnum {
    case one
    case tew
    case three
    case Fir,Sec,Thr
}
var em = MyEnum.one
```

如果变量是类型确定的枚举，在赋值时可以省略枚举名，示例如下：

```
var em:MyEnum = .one
```

Swift中的枚举还有一个原始值的概念，要使用原始值，必须在创建枚举类型时设置原始值的类型，示例如下：

```
enum MyEnum:Int {
    case one=1
    case tew
    case three
    case Fir,Sec,Thr
}
var em = MyEnum.one.rawValue
```

如果原始值是Int类型，则默认从0开始依次递增，开发者也可以手动设置每个枚举值的原始值。同样，也支持使用原始值来创建枚举实例，如下：

```
var em = MyEnum(rawValue:1)
```

通过原始值实例的枚举对象实际上回返回一个optional类型的值，如果传入的原始值参数不能匹配到任何一个枚举case，则可以使用if let结构进行判断处理。

        在枚举中封装方法示例如下：

```
enum MyEnum:Int {
    case one
    case tew
    case three
    case Fir,Sec,Thr
    func des() {
        switch self {
        case .one:
            print("one")
        default:
            print("else")
        }
    }
}
var em = MyEnum(rawValue:1)
em?.des()
```

        Swift中的枚举也可以添加附加值，在switch语句中取到对应的枚举类型后，可以获取开发者设置的附加值进行逻辑处理，示例如下：

```
enum MyEnum {
//为这个类型天啊及一组附加值
    case one(String,Int)
    case tew
    case three
    case Fir,Sec,Thr
    func des() {
        switch self {
        case .one:
            print("one")
        default:
            print("else")
        }
    }
}
var em = MyEnum.one("第一个元素", 1)
switch em {
//前面的let指定附加值为常量 或者用var指定为变量，括号内为附加值参数名
case let .one(param1, param2):
    print("One param is  \(param1) and two param is  \(param2).")
default:
    print("else")
}
```

        Swift中使用struct关键字来进行结构体的创建，结构体的功能和类相似，支持属性与方法，但不同的是，结构体在传递时会被赋值，类的实例则会以引用的方式传递。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

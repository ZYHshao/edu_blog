---
title: Swift讲解专题九——枚举
date: 2016-05-15
categories: Swift语法专题
tags: []
---
## Swift讲解专题九——枚举

### 一、引言

        在Objective-C语言中，没有实际上是整型数据，Swift中的枚举则更加灵活，开发者可以不为其分配值类型把枚举作为独立的类型来使用，也可以为其分配值，可以是字符，字符串，整型或者浮点型数据。

### 二、枚举语法

        Swift中enum关键字来进行枚举的创建，使用case来创建每一个枚举值，示例如下：

```
//创建姓氏枚举,和Objective-C不同，Swift枚举不会默认分配值
enum Surname {
    case 张
    case 王
    case 李
    case 赵
}
//创建一个枚举类型的变量
var myName = Surname.张
//如果可以自动推断出类型 则枚举类型可以省略
myName = .李
var myName2:Surname = .王
```

同样可以将枚举值都写在同一个case中，使用逗号分隔：

```
enum Planet {
    case Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}
```

枚举经常会和Switch语句结合使用，示例如下：

```
switch myName {
case .张:
    print("姓氏张")
case .王:
    print("姓氏王")
case .李:
    print("姓氏李")
case .赵:
    print("姓氏赵")
}
```

### 三、枚举的相关值

        Swift中的枚举有一个很有意思的特点，其可以设置一些相关值，通过相关值，开发者可以从公用的枚举值中获取到传递的额外相关值，示例如下：

```
enum Number {
    case one(count:Int)
    case two(count:Int)
    case three(count:Int)
    case four(count:Int)
}
var num = Number.one(count: 5)
switch num {
    //获取num的相关值
case Number.one(let count):
    print(count)
default:
    print(num)
}
//如果一个枚举值所有的相关中都是常量，let关键字也可以提取到括号外面
switch num {
    //获取num的相关值
case let Number.one(count):
    print(count)
default:
    print(num)
}
```

有了相关值这样的句法，大大的增加了枚举的灵活性，例如一个形状枚举，可能的枚举值有矩形，圆形等，矩形的枚举值就可以提供宽高的相关值，圆形的枚举值就可以提供半径的相关值，是开发更加灵活。

### 四、枚举的原始值

        原始值也可以理解为为枚举设置一个具体类型，示例如下：

```
enum Char:String {
    case a = "A"
    case b = "B"
    case c = "C"
}
//”A“
var char = Char.a.rawValue
```

注意，如果枚举是Int类型的，则类似于Objective-C，枚举的原始值会从第一个开始之后依次递增：

```
enum Char:Int{
    case a = 0
    case b
    case c
}
//1
var char = Char.b.rawValue
```

同样可以通过原始值的方式来进行枚举对象的创建，示例如下：

```
enum Char:Int{
    case a = 0
    case b
    case c
}
//1
var char = Char.b.rawValue
//b
var char2 = Char(rawValue:1)
```

在通过原始值进行枚举对象创建的时候，有可能创建失败，例如传入的原始值并不存在，这时会返回Optional值nil。

### 四、递归枚举

        递归枚举是Swift枚举中一个难于理解的地方，实际上也并非十分难于理解，开发者只要明白枚举的实质，递归枚举就很好理解。首先，递归是一种算法，可以简单理解为自己调用自己，而枚举实际上并不是函数，它并不执行某项运算，它只是表达一个数据或者说他也可以表达一种表达式，示例如下：

```
enum Expression {
    //表示加
    case add
    //表示减
    case mul
}
```

前面有提到过相关值的概念，因此，对于上述例子，可以为add和mul枚举值添加两个相关值作为参数。

```
enum Expression {
    //表示加
    case add(Int,Int)
    //表示减
    case mul(Int,Int)
}
```

如此，如下的写法实际上就可以代表一个5+5的表达式：

```
var exp = Expression.add(5, 5)
```

还是需要强调一点，这个exp只是表达了5+5这样一个约定的表达式，它并没有真正进行5+5的运算。现在问题就来了，使用如上的枚举，怎样来表达类似(5+5)*5这样的复合表达式呢？可以使用递归枚举来实现，即将(5+5)作为枚举值得相关值再次创建枚举，改造如下：

```
enum Expression {
    //单值数据
    case num(Int)
    //表示加 indirect为递归枚举关键字
    indirect case add(Expression,Expression)
    //表示减
    indirect case mul(Expression,Expression)
}
var exp1 = Expression.num(5)
var exp2 = Expression.num(5)
var exp3 = Expression.add(exp1, exp2)
var exp4 = Expression.mul(exp1, exp3)
```

上面exp4实际上就表达了(5+5)*5这样一个过程，注意递归的枚举值必须加上indirect关键字来声明。处理递归枚举最好的方式是通过递归函数，示例如下：

```
func expFunc(param:Expression) -> Int {
    //进行枚举判断
    switch param {
        //如果是单独数字 直接返回
    case .num(let p):
        return p
        //如果是加法 则进行递归加
    case .add(let one, let two):
        return expFunc(one)+expFunc(two)
        //如果是乘法 则进行递归乘
    case .mul(let one, let two):
        return expFunc(one)*expFunc(two)
    }
}
//50
expFunc(exp4)
```

如果枚举中所有的case都是可递归的，可以将整个枚举声明为可递归的：

```
indirect enum Expression {
    //单值数据
    case num(Int)
    //表示加 indirect为递归枚举关键字
    case add(Expression,Expression)
    //表示减
    case mul(Expression,Expression)
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

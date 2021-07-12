---
title: Swift解读专题二——基本类型
date: 2016-05-08
categories: Swift语法专题
tags: []
---
## Swift解读专题二——基本类型

### 一、常量和变量

        Swift语言的常量和变量在使用之前，必须被定义。常量用于处理程序中只在初始化时设置的量值，之后不能进行赋值改变，变量用于处理程序中可以进行改变的量值。分别用let和var来声明常量和变量，示例如下：

```
var varValue = 1
let letValue = 10
```

Swift语法也支持在一行中声明多个量值，示例如下：

```
var a=1,b=2.9,c="string"
```

在声明量值时，编译器会根据第一次赋值的类型来推断出变量的类型，一旦量值的类型被推断，则不能够进行更改，开发者也可以手动注释量值的类型，示例如下：

```
var a:Int=1,b:Float=2.9,c:String="string"
```

在一行中声明多了变量并且没有提供初始值时，为最后一个变量注释的变量类型也会应用于本行中的所有变量，示例如下：

```
var one,two,three:Int
```

官方文档建议，在实际应用中，注释量值的类型是十分少用的，一般都会为其赋值初始值后让编译器自行推断。

        量值的命名可以包含Unicode字符和数字，需要注意，是不能以数字作为量值名称的开头的。空格，数学符号，制表符，箭头等符号也不可以使用。示例如下：

```
//中文符作变量名
var 珲少 = "me"
//表情符作为变量名
var 😄 = "开心"
//含有数字的变量名
var one2three = "123"
//含有下划线的变量名
var _d_s = "C++"
```

注意：如果使用Swift中的保留关键字作为量值的名，需要加上左右个加上`符号包围，除非特殊情况，否则不要使用这种方式命名量值，示例如下：

```
var `let` = "c"
```

使用print()方法可以进行量值的打印，在字符串中使用\\()格式可以插入变量，示例如下：

```
var _d_s = "C++"
print("123\(_d_s)")
```

### 二、关于注释与编写结构

        Swift语言可以使用//进行单行注释和/**/进行多行注释，除此之外，Swift语言还支持多行注释的嵌套，示例如下：

```
//我是单行注释
/*
 我是多行注释
 我是多行注释
 我是多行注释
 */
/*
 嵌套注释
    /*
    嵌套注释
    */
 */
```

使用Swift在编写代码时，以行为每句代码的分隔，当然，开发者也可以将多句代码写在一行中，但是需要以分号进行分隔。示例如下：

```
var tmp = 3;print(tmp)
```

### 三、整型与浮点型

        Swift中提供8位、16位、32位、64位类型的整型，整型数组不包含小数，包含负整数，0和正整数。在Swift语言中，整型是由结构体定义的，可以调用max和min方法获取对应位数的最大值和最小值，示例如下：

```
var maxInt8 = Int8.max     //127
var mimInt8 = Int8.min    //-128
var maxInt16 = Int16.max  //32767
var minInt16 = Int16.min  //-32768
var maxInt32 = Int32.max  //2147483647
var minInt32 = Int32.min  //-2147483648
var maxInt64 = Int64.max  //9223372036854775807
var minInt64 = Int64.min  //-9223372036854775808
```

Int类型的值在不同位数的系统会有不同的结果，在32位系统上，Int与Int32相同，在64位系统上，Int与Int64相同。

        Swift语言也提供了无符号整型，示例如下：

```
var maxUInt8 = UInt8.max       //255
var maxUInt16 = UInt16.max     //65535
var maxUInt32 = UInt32.max     //4294967295
var maxUInt64 = UInt64.max     //18446744073709551615
```

UInt类型在32位系统为UInt32，在64位系统为UInt64。

        浮点型用于创建小数，Swift提供了两种类型的浮点型，Float对应32位的浮点值，Double对应64位的浮点值。

        在定义整型或浮点型数据时，可以通过添加前缀的方式来指定其进制类型，示例如下：

```
var type_10 = 17;  //十进制的17
var type_2 = 0b10001 //二进制的17
var type_8 = 0o21  //八进制的17
var type_16 = 0x11 //16进制的17
```

对于科学计数法，在Swift中使用e和p来标识，在十进制中使用e代表10的n次方，在十六进制中，使用p代表2的n次方，示例如下：

```
var sum = 1.25e3 //1.25*10^3 = 1250
var sun2 = 0x1p3 //1*2^3 = 8
```

Swift中还有一个非常有意思的特性，无论是整型还是浮点型，都可以在数前使用0进行填充，并且可以使用下划线进行可读性分隔，是代码看起来更加清晰，这些都不会改变原数据值，示例如下：

```
var num1 = 001.23
var num2 = 1_000
var num3 = 1_000.1_001
```

开发者也可以使用typealias关键字来为某个数据类型添加一个别名，示例如下：

```
typealias MyType = Int
```

### 四、布尔类型

        在Objective-C中，BOOL值实际上是无符号的整型数据，其约定0为NO，非0都为YES。在Swift中，Bool被作为一种独立的数据类型，提供true和false两种值。示例如下：

```
var boolVale:Bool = true
```

### 五、元组

        元组是Swift语言十分重要的一个特点，它允许开发者将任意个不同类型的数据组合成一个数据类型，这也是Swift语言的一个强大之处。例如如下示例代码可以创建一个元组：

```
var tuples:(param1:Int,param2:Float,param3:String,param4:Bool) = (3,3.14,"圆周率",true)
```

tuples就是一个类型为(param1:Int,parame:Float,param3:String,param4:Bool)类型的元组。取元组数据的对应值有两种方式，一种是使用数据参数名称，一种是直接使用数据的角标，示例如下：

```
//通过参数名取元组中的数据
var tuplesInt = tuples.param1;
var tuplesFloat = tuples.param2;
var tuplesString = tuples.param3;
var tuplesBool = tuples.param4;
//通过角标取元组中的数据
var tuplesInt2 = tuples.0;
var tuplesFloat2 = tuples.1;
var tuplesString2 = tuples.2;
var tuplesBool2 = tuples.3;
```

开发者也可以将元组分解成单独的常量进行访问，示例如下：

```
let (fir,sec,thr,four) = tuples
print(fir,sec,thr,four)
```

有时候，某个元组中的所有数据开发者并不一定都需要使用，开发者可以选择只提取元组中所需要的值，示例如下：

```
let (fir,_,thr,_) = tuples
print(fir,thr)
```

开发文档提示，元组只适合临时的简单组合数据，并不适合处理复杂的数据逻辑，对复杂数据逻辑的处理更提倡使用类。

### 六、Optionals值

        Optional也是一种具体的数据类型，其寄附与其他数据类型上，其只有两个值：

1.如果有值，则它为具体的值。

2.如果没有值，则它为nil。

对于习惯了Objective-C语言设计风格的开发者来说，Optional的概念可能有些难于理解，通过一个例子就很好理解，示例代码如下：

```
let tmp = 123
let tmp2 = Int("123")
```

上面创建的两个常量tmp和tmp2虽然值都是123，然而其并不是相同的类型，tmp是严格的Int类型值，tmp2是基于Int类型的Optional值，他们在使用时，Optional值需要使用!进行拆包操作，示例如下：

```
let tmp = 123
let tmp2 = Int("123")
let tmp3 = tmp + tmp2!
```

有时候，Int()构造方法并不一定能构造成功，这时tmp2是会为nil值的，示例如下：

```
let tmp2 = Int("a")
```

将普通类型声明为Optional类型，只需在类型名后添加?符号即可，示例如下：

```
let optionalValue:Int? = 1
```

Swift中的nil与Objective-C中的nil意义并不相同，在Objective-C中，nil代表指针指向一个不存在的对象，Swift中的nil并不是指针，它是一种抽象类型的值，在Swift不只对象的Optional类型可以设置为nil，任何数据类型的Optional类型都可以设置为nil。

        Optional值经常会和if条件语句一起使用，用来判断某个值是否被初始化了，示例如下：

```
if optionalValue != nil {
    print(optionalValue)
}
```

Swift还提供了if let语法进行Optional值得绑定，示例如下：

```
//如果optionalValue值不为nil，则会将拆包后的值赋值给tip
if let tip=optionalValue {
    print(tip)
}
```

开发者还可以在一个绑定语句后进行多个Optional值的绑定，并使用where进行条件判断，示例如下：

```
let optionalValue:Int? = 1
let optionalValue2:Int? = 2
if let tip=optionalValue,tip2=optionalValue2 where tip<tip2{
    print(tip,"<",tip2)
}
```

### 七、异常处理

        Swift中也有一套十分强大的异常处理系统。在编写函数时，如果这个函数可能抛出异常，则需要加上throw关键字，并且在函数中也是使用throw关键字来进行异常的抛出。示例如下：

```
//异常的捕获 自定义的异常必须继承与ErrorType类
enum MyErrorType:ErrorType {
    case CanNotZero
    case Other
}
//可能抛出异常的函数
func ErrorTest() throws {
    let a=0;
    if a==0 {
        throw MyErrorType.CanNotZero
    }
}
//进行异常捕获
do {
    //使用try进行可能抛异常函数的执行
    try ErrorTest()
    //没有错误执行的代码块
    //catch加错误类型 为捕获相应的异常
}catch MyErrorType.Other {
    //抛异常后执行的代码块
    print("MyErrorType.Other")
}catch MyErrorType.CanNotZero {
    print("MyErrorType.CanNotZero")
}catch{
    //如果不写捕获的异常类型 则会捕获所有异常 并且传入一个error异常参数
    print(error)
}
```

### 八、断言

        在Objective-C中，使用Assert相关的宏来进行断言处理，在Swift中也同样有断言的相关操作，断言可以帮助开发者为某种情况添加一个异常中断，为开发者提供调试信息。断言会要求提供一个条件进行判断，当条件为真时，程序继续运行，如果条件为假，则程序会断开，示例如下：

```
let age = -3
assert(age>0, "age must be bigger than zero")
```

官方文档为开发者提供了几种断言使用的场景，参考如下：

1.对于索引过小或过大的检查。

2.当无效的参数传递进函数时。

3.对于一个可能为nil的值，当为nil时后续代码无法工作时。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

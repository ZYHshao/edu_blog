---
title: Swift专题讲解二十三——高级运算符
date: 2016-05-31
categories: Swift语法专题
tags: []
---
## Swift专题讲解二十三——高级运算符

### 一、引言

        除了前边博客中介绍的基本运算符外，Swift中还支持更多高级运算符，也支持开发者进行运算符的自定义。Swift中的算符运算符有一个特点，其不会产生溢出，如果有操作产生溢出，程序会直接抛出异常。如果开发者在开发中需要有溢出操作，需要使用溢出操作符来实现。

### 二、位运算符

        Swift支持C语言中的全部位运算符，示例如下：

```javascript
//二进制数据8 实际上a = 00001000 8位
var a:UInt8 = 0b1000
//使用~ 进行按位取反运算 a = 0b11110111  247
a = ~a
//使用& 进行按位与运算 a = 0b11110000 240
a = 0b11110000&a
//使用|进行按位或运算 a=0b11111111 255
a = 0b11111111|a
//使用^进行按位异或运算 a = 0b00001111 15
a = 0b11110000^a
//使用<<进行按位左移运算 a = 0b00011110 30
a = a<<1
//使用>>进行按位右移运算 a = 0b00001111
a = a>>1
```

Swift中还提供了一种检查机制，当存在溢出操作时，程序会抛出异常，这样可以是开发者编写的代码更加安全，如果开发者真的需要使用溢出操作，Swift中还额外提供了支持溢出操作的运算符：

```javascript
//a = 255 + 1 这样的运算会报错 &+ 为溢出加运算符 计算后a=0
a = 255 &+ 1
//&- 为溢出减运算符 计算后 a = 255
a = a &- 1
//&* 为溢出乘运算符
a = a &* 2
```

### 三、重载运算符

        运算符的重载是为原有的运算符增加新的功能，开发者可以自定义一些运算符函数来实现对具体类和结构体运算的功能，示例如下：

```javascript
class Circle {
    //圆心
    var point:(p1:Float,p2:Float)
    //半径
    var r:Float
    init(point:(Float,Float),r:Float){
        self.point = point
        self.r = r
    }
}
func + (c1:Circle,c2:Circle) -> Circle {
    return Circle(point: c1.point, r: c1.r+c2.r)
}
```

上面代码演示的例子中重载了中缀运算符，即运算符是出现在两个操作数和中间的，还可以进行前缀运算符与后缀运算符的重载，使用prefix与postfix即可。示例如下：

```javascript
prefix func + (c:Circle) -> Circle {
    return Circle(point: c.point, r: c.r*2)
}
```

复合运算符也可以支持重载，需要注意的是，复合运算符的参数必须是inout修饰的，因为复合运算符会直接操作参数值：

```javascript
func += (inout c1:Circle,c2:Circle) {
    c1 = Circle(point: c1.point, r: c1.r+c2.r)
}
```

等价运算符也可以用来重载，通常用来进行比较操作，示例如下：

```javascript
func == (c1:Circle,c2:Circle) -> Bool {
    return (c1.point==c2.point && c1.r==c2.r)
}
func != (c1:Circle,c2:Circle) -> Bool {
    return ((c1.point != c2.point) || (c1.r != c2.r))
}
```

### 四、自定义运算符

        Swift中除了可以对一些已经存在的运算符进行重载操作外，开发者还可以自定义一些运算符，在自定义运算符时，必须指定运算符是前缀、中缀或是后缀，示例如下：

```
//定义一个中缀运算符+!+ operator关键字用于定义运算符
infix operator +!+{}
//进行运算符的实现
func +!+ (param:Int,param2:Int)->Int{
    return (param+param2)*param2
}
var b = 5 +!+ 5
```

还有一点需要注意，在进行自定义运算符时，开发者也可以为其设置结合性与优先级，结合性由associativity关键字定义，可选left，right，none，优先级的默认值为100，由precedence关键字指定，示例如下：

```javascript
//定义一个中缀运算符+!+
infix operator +!+{associativity left precedence 140}
//进行运算符的实现
func +!+ (param:Int,param2:Int)->Int{
    return (param+param2)*param2
}
var b = 5 +!+ 5
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

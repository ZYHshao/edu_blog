---
title: Swift讲解专题七——函数
date: 2016-05-13
categories: Swift语法专题
tags: []
---
## Swift讲解专题七——函数

### 一、引言

        函数是有特定功能的代码段，函数会有一个特定的名称调用时来使用。Swift提供了十分灵活的方式来创建与调用函数。事实上在Swift，每个函数都是一种类型，这种类型由参数和返回值来决定。Swift和Objective-C的一大区别就在于Swift中的函数可以进行嵌套。

### 二、函数的创建与调用

        函数通过函数名，参数和返回值来定义，参数和返回值决定一个函数的类型，在调用函数时，使用函数名来进行调用，示例如下：

```
//传入一个名字 打印并将其返回
func printName(name:String) -> String {
    print(name)
    return name
}
//进行函数的调用
printName("HS")
```

也可以创建没有参数的函数：

```
func onePuseTwo()->Int {
    return 1+2
}
onePuseTwo()
```

同样也可以创建没有返回值的函数：

```
func sayHello(){
    print("Hello")
}
sayHello()
```

上面介绍的函数类型都比较常见，对于多返回值的函数，在Objective-C中十分难处理，开发者通常会采用字典、数组等集合方式或者干脆使用block回调，在Swift中，可以使用元组作为函数的返回值，示例如下：

```
func tuples()->(Int,String){
    return (1,"1")
}
tuples()
```

也可以是函数返回一个Optional类型的值，支持返回nil，示例如下：

```
func func1(param:Int)->Int? {
    guard(param>0)else{
        return nil
    }
    return param
}
func1(0)
func1(1)
```

在函数的参数名前，开发者还可以再为其添加一个参数名称作为外部参数名，示例如下：

```
func func1(count param:Int ,count2 param2:Int)->Int? {
    //内部依然使用param
    guard(param>0)else{
        return nil
    }
    return param
}
//外部调用使用count
func1(count: 0,count2: 0)
func1(count: 1,count2: 1)
```

其实Swift函数中的参数列表有这样一个特点，除了第一个参数外，之后的参数都默认添加一个一个和内部名称相同的外部名称，如果开发者不想使用这个外部名称，使用_符号设置，示例如下：

```
func func2(param:Int,param2:Int,param3:Int) {
    
}
//有外部名称
func2(0, param2: 0, param3: 0)
func func3(param:Int,_ param2:Int,_ param3:Int) {
    
}
//没有外部名称
func3(0, 0, 0)
```

Swift也支持开发者为函数的参数创建一个默认值，如果函数的某个参数有设置默认值，则开发者在调用时可以省略此参数，示例如下：

```
func func4(param:Int=1,param2:Int=2,param3:Int) {
    print(param,param2,param3)
}
func4(3,param3:3)
```

还有一种情形在Objective-C中也很处理，对于参数数量不定的函数，在前面章节介绍过，Objective-C一般会使用list指针来完成，在Swift中编写这样的函数十分简单，示例如下：

```
func func5(param:Int...)  {
    for index in param {
        print(index)
    }
}
func5(1,2,3,4)
```

Swift中参数默认是常量，在函数中是不能修改外部传入参数的值得，如果有需求，需要将参数声明成inout类型，示例如下：

```
func func6(inout param:Int)  {
    param = 10
}
var count = 1
//实际上传入的是参数地址
func6(&count)
print(count)
```

### 三、函数类型

        函数是一种特殊的数据类型，每一个函数属于一种数据类型，示例如下：

```
func func7(a:Int,_ b:Int)->Int{
    return a+b
}
var addFunc:(Int,Int)->Int = func7
addFunc(1,2)
```

函数也可以作为参数传入另一个函数，这十分类似于Objective-C中的block语法，示例如下：

```
func func7(a:Int,_ b:Int)->Int{
    return a+b
}
var addFunc:(Int,Int)->Int = func7
addFunc(1,2)
func func8(param:Int,param2:Int,param3:(Int,Int)->Int) -> Int {
   return param3(param,param2)
}
//传入函数
func8(1, param2: 2, param3: addFunc)
//闭包的方式
func8(2, param2: 2, param3:{ (a:Int,b:Int) -> Int in
    return a*b
    })
```

一个人函数也可以作为另一个函数的返回值，示例如下：

```
func func9()->(Int)->Int{
    //Swift支持嵌套函数
    func tmp(a:Int)->Int{
       return a*a
    }
    return tmp
}
var myFunc = func9()
myFunc(3)
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

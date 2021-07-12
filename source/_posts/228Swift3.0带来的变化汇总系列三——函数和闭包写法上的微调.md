---
title: Swift3.0带来的变化汇总系列三——函数和闭包写法上的微调
date: 2016-06-26
categories: Swift语法专题
tags: []
---
## Swift3.0带来的变化汇总系列三——函数写法上的微调

### 一、函数方面

    Swift3.0相比Swift2.2的版本在API上做了大量的修改，代码风格也更加统一。在函数方面，Swift3.0中做的最大修改是修改了内部名称与外部名称的默认规则。

    在Swift2.2中，函数参数列表的第一个参数如果开发者不手动设置外部名称，默认是匿名的，除第一个参数以外的其他参数，开发者如果不设置外部名称，默认外部名称是和内部名称相同的，因此在调用函数时，代码常常是这样的：

```objectivec
//多参数函数Swift2.2中 第一个参数默认匿名，其他参数默认内部命名与外部命名相同
func myFunc5(param1: Int,param2: Int,param3: Int) {
    //这里使用的param1，param2，param3是参数的内部命名
    param1+param2+param3
}
//调用函数的参数列表中使用的param2和param3为外部命名
myFunc5(1, param2: 2, param3: 3)
```

Swift3.0中将这一规则修改为：如果开发者不设置函数中参数的外部名称，则全部参数都默认外部名称和内部名称相同，上面相同的代码，在Swift3.0的环境下是下面这样的：

```objectivec
//多参数函数 默认内部命名与外部命名相同
func myFunc5(param1: Int,param2: Int,param3: Int) {
    //这里使用的param1，param2，param3是参数的内部命名
    param1+param2+param3
}
//调用函数的参数列表中使用的param1、param2和param3为外部命名
//swift3.0
myFunc5(param1: 1, param2: 2, param3: 3)
```

Swift3.0在函数参数名方面的微调使得函数的参数名规则更加统一也更加符合Swift语言的风格。

        在函数方面，Swift3.0中做的另一项更改是关于inout参数的声明方式，修改了inout关键字的声明位置，Swift2.2与Swift3.0版本比如如下：

```objectivec
//在函数内部修改参数变量的值
//swift2.2
func myFunc12(inout param:Int){
    param+=1
}
//swift3.0
func myFunc12( param:inout Int){
    param+=1
}

```

有关Swift中函数的更多内容，可以在如下博客连接中找到：

[http://my.oschina.net/u/2340880/blog/674616](http://my.oschina.net/u/2340880/blog/674616)

### 二、闭包方面

        在闭包方面，Swift3.0版本中只对某些修饰符的位置做了修改。示例如下：

```objectivec
//逃逸闭包
//swift2.2
//func myFunc(@noescape closure:(Int,Int)->Bool){
//    
//}
//swift3.0
func myFunc( closure:@noescape(Int,Int)->Bool){

}
//自动闭包
//swift2.2
//func myFunc2(@autoclosure(escaping) closure:()->Bool)  {
//    
//}
func myFunc2( closure:@autoclosure(escaping)()->Bool)  {
    
}
```

关于Swift中闭包的更多内容，可以在如下博客链接中找到：

[http://my.oschina.net/u/2340880/blog/675233](http://my.oschina.net/u/2340880/blog/675233)。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

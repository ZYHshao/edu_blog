---
title: Swift讲解专题十三——下标访问
date: 2016-05-17
categories: Swift语法专题
tags: []
---
## Swift讲解专题十三——下标访问

### 一、引言

        在以前的博客中，讨论过在Objective-C中，通过下标的方式访问自定义数据模型的方法。Objective-C中主要是通过实现一系列方法来使自定义的数据类型支持下标的访问方式，博客地址如下：

在Objective-C中使用下标访问自定义数据模型：[http://my.oschina.net/u/2340880/blog/632294](http://my.oschina.net/u/2340880/blog/632294)。

        Swift中的Array，Dictionary类型可以通过下标或者键值的方式来进行数据的访问，实际上在Swift的语法中，下标可以定义在类、结构体、枚举中。开发者可以通过下标的方式来对属性进行访问而不用使用专门的存取方法。并且定义的下标不限于一维，开发者可以定义多维的下标来满足需求。

### 二、下标的语法结构

        下标使用subscript来定义，其有些类似于方法，参数和返回值分别作为下标入参和通过下标所取的值。但是在subscript实现部分，又十分类似于计算属性，其需要实现一个get块和可选实现一个set块，get块用于使用下标取值，set块用于使用下标设置值，因此，subscript结构更像是计算属性和方法的混合体，示例如下：

```
class MyClass {
    var array=[1,1,1,1,1]
    subscript(param1:Int)->Int{
        set{
            array[param1] = newValue
        }
        get{
            return array[param1]
        }
    }
}
var obj = MyClass()
obj[0] = 3
```

开发者可以只编写get块来实现只读的下标访问。对于多维下标的访问方式，只需修改subscript中的参数个数即可，示例如下：

```
class MyClass {
    var array=[1,1,1,1,1]
    subscript(param1:Int,param2:Int)->Int{
        set{
            array[param1] = newValue
        }
        get{
            return array[param1]
        }
    }
}
var obj = MyClass()
obj[0,1] = 3
```

### 三、下标的特性

        Swift中的下标可以自定参数个数和参数类型，返回数据的类型开发者也可以进行自定义。但是有一点需要注意，下标的参数不能设置默认值，也不能设置为in-out类型。多维下标常用语行列数据的访问，示例如下：

```
class SectionAndRow {
    var array:Array<Array<Int>> = [  [1,2]
                                    ,[3,4]
                                    ,[5,6]
                                    ,[7,8]
                                  ]
    subscript(section:Int,row:Int)->Int{
        get{
            let temp = array[section]
            return temp[row]
        }
    }
    
}
var data = SectionAndRow()
//通过二维下标取值
data[1,1]
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

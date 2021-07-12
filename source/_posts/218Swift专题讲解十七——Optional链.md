---
title: Swift专题讲解十七——Optional链
date: 2016-05-24
categories: Swift语法专题
tags: []
---
## Swift专题讲解十七——Optional链

        Swift中的Optional值有这样的特性，当对其进行可选拆包时，即使用?进行Optional类型值的取值时，如果Optional值不为nil，则会返回原始类型的数据值，如果为nil，则会返回nil。因此，当使用?对Optional拆包后进行方法、属性或者下标的调用时，如果有值，则会成功相应调用，如果没有值，则会调用失败，返回nil。

        注意：使用!则会进行强制拆包，这时如果Optional值为nil，则会出现运行时错误，因此开发者在使用!进行强制拆包时，必须确认Optional类型值不为nil。

        当对可选值进行可选拆包并调用其属性或方法后，无论原属性或者方法返回值是什么类型的，都会被包装成Optional值类型。当使用?对一个Optional值进行拆包并调用其方法时，方法的返回值一会被包装为Optional类型，示例如下：

```javascript
class Myclass {
    var cls:MyClassTwo?
    
}
class MyClassTwo {
    func run() -> String {
        return "run"
    }
}

let obj:Myclass = Myclass()
//将返回nil
obj.cls?.run()
```

        在进行Optional链调用的时候，会遵守如下一些特性：

1.如果进行?拆包Optional值的属性或者方法返回值原来为非Optional值，则会包装成Optional值。

2.如果进行?拆包Optional值的属性或者方法返回值原来为Optional值，则依然会返回Optional值，并且并不会进行Optional值类型的嵌套。

3.由于使用Optional值?可选拆包时会将其属性和方法的返回值都包装成Optional类型的，因此使用?可以进行Optional链式调用，这其间，有一个环节调用失败，整个链都会返回nil。示例如下：

```javascript
let obj:Myclass = Myclass()
//将返回nil
(obj.cls?.run())?.startIndex
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

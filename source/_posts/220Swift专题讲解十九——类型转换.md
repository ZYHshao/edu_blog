---
title: Swift专题讲解十九——类型转换
date: 2016-05-27
categories: Swift语法专题
tags: []
---
## Swift专题讲解十九——类型转换

### 一、类型检查与转换

        在Objective-C和Java中，任何类型实例都可以通过强转使编译器认为它是另一种类型的实例，这么做其实是将所有的安全检查工作都交给了开发者自己来做。先比之下，Swift中的Optional类型转换就会比较安全与可靠。

        Swift中使用is关键字来进行类型的检查，其会返回一个布尔值true或者false来表明检查是否成立，示例如下：

```javascript
var str = "HS"
if str is String {
    print(str)
}

```

        Swift中有向上兼容与向下转换的特性，就是说，一个父类类型的集合可以接收子类的实例，同样，在使用这些实例变量时可以将其向下转换为子类类型，示例如下：

```javascript
//自定义一个类及其子类
class MyClass {
    var name:String?
}

class MySubClassOne: MyClass {
    var count:Int?
}
class MySubClassTwo: MyClass {
    var isBiger:Bool?
}
//创建3个实例
var obj1 = MyClass()
obj1.name = "HS"
var obj2 = MySubClassOne()
obj2.count = 100
var obj3 = MySubClassTwo()
obj3.isBiger=true
//将实例存放在其公共父类类型的数组集合中
var array:[MyClass] = [obj1,obj2,obj3]
//进行遍历
for var i in 0..<array.count {
    var obj = array[i]
    if obj is MySubClassOne {
        print((obj as! MySubClassOne).count!)
        continue
    }
    if obj is MySubClassTwo {
        print((obj as! MySubClassTwo).isBiger!)
        continue
    }
    if obj is MyClass {
        print(obj.name!)
    }
}
```

有一点需要注意，在进行类型转换时，可以使用as!或者as?来进行，as!是一种强制转换方法，它在开发者确定类型无误是使用，如果用as!转换的类型有误，则会出现运行时错误。as?是Optional类型转换，如果转换失败，则会返回nil。

### 二、Any和AnyObject类型

        在Objective-C中，常常使用id来表示引用类型的泛型，Swift中的AnyObject与之类似。示例如下：

```javascript
//进行遍历
for var i in 0..<array.count {
    var obj = array[i]
    if obj is MySubClassOne {
        print((obj as! MySubClassOne).count!)
        continue
    }
    if obj is MySubClassTwo {
        print((obj as! MySubClassTwo).isBiger!)
        continue
    }
    if obj is MyClass {
        print((obj as! MyClass).name!)
    }
}
```

Any类型则比AnyOject类型更加强大，其可以混合值类型和引用类型一起工作，示例如下：

```javascript
var anyArray:[Any] = [100,"HS",obj1,obj2,false,(1.1),obj3,{()->() in print("Closures")}]
```

上面示例的数组中包含了整型，字符串类型，引用类型，布尔类型和闭包。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

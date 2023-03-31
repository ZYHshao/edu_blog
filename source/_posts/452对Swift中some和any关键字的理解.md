---
title: 对Swift中some和any关键字的理解
date: 2022-10-31
categories: Swift
tags: []
---
# 对Swift中some和any关键字的理解

在最新Swift版本中（Xcode14,Swift5.7），如果协议中有使用泛型，则如果要将此协议作为参数类型，必须使用any关键字进行修饰。其实在Swift5.1中也引入过一个some关键字，any和some都适用于协议，这两个关键字从语义上和写法上对泛型的使用进行了优化。

## 1. any

我们知道，协议中会规定一些属性和方法，用来约束其他结构的实现。举个简单的例子，我们可以使用协议定义了一个可飞行的实例需要实现的方法和属性，如下：

```swift
protocol Fly {
    var name:String {get set}
    func fly()
}

class Bird:Fly {
    var name = "Bird"
    func fly() {
        print(name + "Fly")
    }
}

func test(f: Fly) {
    f.fly()
}

test(f: Bird())

```

这个程序当前运行的很好，语义也很明确，即test的函数的参数需要是实现了Fly协议的任意类型，其实在此中情况下，虽然在调用是我们传入的是Bird实例，但是由于协议类型的约束较弱，在函数执行时编译器会将其解释成了Fly类型，实际上产生了类型丢失。尤其是当协议中有使用泛型时，此时上面的写法在最新的Xcode版本中会提示错误，需要我们添加any关键字。any关键字的意义其实就是实现上述的语义，将参数类型定义为遵守某个协议的任意类型，如下：

```swift
import Foundation

protocol Fly {
    associatedtype T
    var name:T {get set}
    func fly()
}


class Bird:Fly {
    typealias T = String
    var name = "Bird"
    func fly() {
        print(name + "Fly")
    }
}

func test(f: any Fly) {
    f.fly()
}

let f = test(f: Bird())

```

## 2.some

针对于上面代码的应用场景，我们只需要约束参数的类型是遵守Fly协议的即可，但是有时候这并不够，有时协议中的函数会需要多个参数，我们需要使用泛型约束其参数的类型一致，例如：

```swift
import Foundation

protocol Fly {
    associatedtype T
    var name:T {get set}
    func fly()
    func add(a:T, b:T)
}


class Bird:Fly {
    typealias T = String
    var name = "Bird"
    func fly() {
        print(name + "Fly")
    }
    func add(a: String, b: String) {
        
    }
}

func test(f: any Fly) {
    f.fly()
    // 这里会报错 因为any Fly类型在运行时无法确定成某个具体的类型
    f.add(a: f.name, b: f.name)
}

test(f: Bird())


```

可以看到，上面的代码中，test函数会报错，核心的原因在于any Fly类型的语音是任意实现了Fly协议的类型，无论是编译时还是运行时，编译器都无法推导出此f参数的类型。要解决上面的问题，可以采用泛型的方式来改写，如下：

```swift
func test<T:Fly>(f: T) {
    f.fly()
    f.add(a: f.name, b: f.name)
}

```

此时代码则没有任何问题了，some关键字其实也是用于这一种场景，其表示的是一种透明类型，在运行时编译器知道其具体的类型是什么，只是对调用方来说是抽象的。下面的写法与上面使用泛型的写法作用完全一致：

```swift
func test(f: some Fly) {
    f.fly()
    f.add(a: f.name, b: f.name)
}

```

整体看来，相对与泛型那种写法，使用some的写法语义更加清晰，风格上也与any刚好一致。

最后，我们再来总结下，整体看来，any和some都是用来描述语义的关键字，any和协议一起使用，表示的是语义比较传统，及遵守了某个协议的类型，具体什么类型编译器也不知道。而some和协议一起使用表示的是具象的一个类型，此类型编译时不知道，调用时也开发者来说也是透明的，但是编译器自己是知道的，它就是具体的一个类型。

> 专注技术，懂的热爱，愿意分享，做个朋友
> 
> QQ：316045346
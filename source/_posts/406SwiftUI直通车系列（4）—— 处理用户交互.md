---
title: SwiftUI直通车系列（4）—— 处理用户交互
date: 2020-09-29
categories: SwiftUI
tags: []
---
# SwiftUI直通车系列（4）—— 处理用户交互

前情回顾：

[SwiftUI直通车系列（1）—— 视图的布局与组织](https://my.oschina.net/u/2340880/blog/4529951)

[SwiftUI直通车系列（2）—— 列表视图](https://my.oschina.net/u/2340880/blog/4534222)

[SwiftUI直通车系列（3）—— 使用导航](https://my.oschina.net/u/2340880/blog/4652002)

前面，我们介绍了在SwiftUI中布局与页面跳转的相关逻辑，其中页面跳转可以算是处理用户交互，SwiftUI虽然从名字上听，是一个UI框架，但是其依然有很强大的交互处理能力。本篇博客，我们就其用户交互能力进行介绍和学习。

## 一、SwiftUI中的按钮

    按钮是处理用户交互的最基础的组件，在SwiftUI中，使用Button可以十分方便的定义按钮组件，如下：

```objectivec
var count = 0
struct SimpleView:View {
    
    var body: some View {
        VStack(alignment: .center, spacing: 20) {
            Button(action:{
                count += 1
            }){
                Text("按钮一")
            }
            Text("\(count)")
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
       SimpleView()
    }
}
```

在预览界面，可以看到一个按钮和计数器已经布局在了页面上，我们尝试点击按钮，会发现变量count虽然已经被改变了，但是页面并没有更新。能够接收到用户交互只是第一步，在SwiftUI中通过用户交互而进行页面的刷新才是核心我们需要关注的。效果如下：

![](https://oscimg.oschina.net/oscnet/up-a83971b647e670d58586138c47ffcde6615.png)

## 二、状态

    SwiftUI采用的是描述性的结构化布局策略，即通过定义结构体这种结构化的数据来描述页面的布局。这种布局思路开发直观快速，并且非常易于入门上手。其与流行的前端框架Flutter，React等布局思路已经非常相似。因此，对于页面的刷新操作，我们也不能采用原始的iOS开发页面刷新的方式(对对象进行操作)，而是要使用状态来对组件进行管理，改变组件的状态来实现组件刷新操作。

    修改上面代码如下：

```swift
struct SimpleView:View {
    @State var count = 0
    var body: some View {
        VStack(alignment: /*@START_MENU_TOKEN@*/.center/*@END_MENU_TOKEN@*/, spacing: 20) {
            Button(action:{
                self.count += 1
            }){
                Text("按钮一")
            }
            Text("\(self.count)")
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
       SimpleView()
    }
}
```

重新恢复预览，可以看到计数器的功能已经正常了。@State是SwiftUI中提供的一个属性装饰器，其可以将某个属性指定为有状态的，当有状态的属性发生变化时，与其绑定的组件会自动刷新。

    如果有状态属性在父组件中，而我们需要在子组件中处理用户交互逻辑进行页面刷新，这时，如果不做特殊的处理，子组件会因为结构体不可修改的原则而导致无法对传递进来的状态属性进行更改。对于这种场景，我们可以使用SwiftUI中提供的@Binding装饰器来修改子组件中的属性，例如：

```swift
struct SimpleSubText:View {
    @Binding var count:Int
    var body: some View {
        Button(action:{
            self.count += 1
        }){
            Text("按钮一")
        }
        Text("\(self.count)")
    }
}


struct SimpleView:View {
    @State var count = 0
    var body: some View {
        VStack(alignment: /*@START_MENU_TOKEN@*/.center/*@END_MENU_TOKEN@*/, spacing: 20) {
            SimpleSubText(count: $count)
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
       SimpleView()
    }
}
```

## 三、使用环境对象

    使用状态来管理组件刷新并不复杂，但是如果组件的嵌套层数较多，有状态数据的传递可能需要层层下传，这会使数据的流转变得相对复杂。在SwiftUI中，还有另外一种方式，可以定义环境对象，环境对象在SwiftUI组件的任意位置都可以自由访问和修改，与环境对象有绑定关系的组件都会接收到变动的通知并刷新。示例代码如下：

```swift
// 作为环境数据的类，必须是可绑定监听的，需要继承ObservableObject
class EnviData: ObservableObject {
    @Published var count = 0
}

struct SimpleSubText:View {
    // 获取环境数据进行使用
    @EnvironmentObject var data:EnviData
    var body: some View {
        Button(action:{
            self.data.count += 1
        }){
            Text("按钮一")
        }
        Text("\(self.data.count)")
    }
}


struct SimpleView:View {
    var body: some View {
        VStack(alignment: .center, spacing: 20) {
            SimpleSubText()
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        // 传递环境数据
        SimpleView().environmentObject(EnviData())
    }
}
```

需要注意，并不是环境对象中所有的属性都会触发绑定和监听，只有使用@published的属性才会触发，因此环境对象也是支持静态的内部属性。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：805263726

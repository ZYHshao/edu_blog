---
title: SwiftUI直通车系列（1）—— 视图的布局与组织
date: 2020-08-26
categories: SwiftUI
tags: []
---
# SwiftUI直通车系列（1）—— 视图的布局与组织

## 一、引言

       SwiftUI提供了一种更快、更高效也更简单的页面开发方式。我们知道相对于Objective-C，Swift语言本身就更加高效简洁，SwiftUI采用了结构化的布局方式，使得应用的界面开发更加直观快速。本系列博客，基于Apple官方的SwiftUI Tutorials为参考，配合代码示例介绍了SwiftUI的基础应用和特性，帮助大家快速入门SwiftUI开发，为工作中的开发效率赋能。

## 二、SwiftUI初体验

     做为Apple相关产品的开发者，影响界面开发效率的一大问题是每次代码的修改要查看效果都需要重新编译运行。往往，调试验证的时间要远远大于编码时间，严重影响了开发效率。当然，这也是大多数编译型语言所共有的痛点。使用SwiftUI配合Xcode的预览功能，可以做到代码的实时修改实时预览效果，界面的开发效率非常高。

      首先，我们可以创建一个模板工程体验下SwiftUI的基础功能。使用Xcode新建一个App项目，在选择语言时，选择Swift，并使用SwiftUI创建界面入口，如下图所示。

![](https://oscimg.oschina.net/oscnet/up-6106e0a07c098ea3227c19b048fd9742e40.png)

创建出的工程中包含3个swift文件，其中AppDelegate.swift文件是应用的入口文件，其在任何模板项目中几乎都是一样的，没有什么特别支持。SceneDelegate.swift文件是iOS 13后新引入的，用来进行多场景的管理，我们也无需关注。ContentView.swift文件是最终的界面定义文件，其中使用SwiftUI定义了界面的显示内容，ContentView.swift文件中的代码如下：

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Text("Hello, SwiftUI!")
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

如上代码所示，ContentView是定义的一个视图结构体，其描述的就是界面的渲染信息，其中Text会在页面上渲染出一个文本标签，显示“Hello Swift!”，ContentView_Previews定义了当前界面的预览视图，你可以尝试对Text中的文本进行修改，修改后预览页面就会实时的展示出当前的界面模样，如下图所示。

![](https://oscimg.oschina.net/oscnet/up-72a90544f07953fb51f23b54e5f196b2557.png)

可以通过为Text组件设置属性来实时的改变界面，例如文本的颜色，字体，是否加下划线等等，如下：

```swift
struct ContentView: View {
    var body: some View {
        Text("Hello, SwiftUI!")
            .foregroundColor(Color.red)
            .underline()
            .font(Font.system(size: 25))
    }
}
```

此时页面效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-1fa7e50699956661d08e8b89000a6051158.png)

通过代码可以修改UI元素的属性，除此之外，我们还可以设置容器的对齐方式和元素间距等，如下：

```swift
struct ContentView: View {
    var body: some View {
        VStack (alignment: .leading, spacing: 10) {
            Text("Hello, SwiftUI!啊啊啊")
                .foregroundColor(Color.red)
                .underline()
                .font(Font.system(size: 25))
            Text("Hello, SwiftUI!")
            .foregroundColor(Color.red)
            .underline()
            .font(Font.system(size: 25))
        }
    }
}
```

其中alignment设置内部组件的对其方式，spacing设置组件间距离，对于水平栈，spacing设置的是列间距，对于垂直栈，spacing设置的是行间距。默认，组件会自动计算其做占据的位置，SwiftUI中也提供了一种特殊的占位元素，其可以将剩余的空间充满，例如在两个垂直布局的元素中间加入一个占位元素，则其会进行两端布局，如下：

```swift
struct ContentView: View {
    var body: some View {
        VStack (alignment: .leading, spacing: 10) {
            Text("Hello, SwiftUI!啊啊啊")
                .foregroundColor(Color.red)
                .underline()
                .font(Font.system(size: 25))
            Spacer()
            Text("Hello, SwiftUI!")
            .foregroundColor(Color.red)
            .underline()
            .font(Font.system(size: 25))
        }
    }
}
```

效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-529ff8c2e8f243a5d1edd778412f2734955.png)

可以发现，SwiftUI在布局上，很多思路都和CSS布局很像，对于组件元素，我们也可以为其追加内间距Padding，例如：

```swift
struct ContentView: View {
    var body: some View {
        VStack (alignment: .leading, spacing: 10) {
            Text("Hello, SwiftUI!啊啊啊")
                .foregroundColor(Color.red)
                .underline()
                .font(Font.system(size: 25))
            Spacer()
            Text("Hello, SwiftUI!")
            .foregroundColor(Color.red)
            .underline()
            .font(Font.system(size: 25))
        }
        .padding(EdgeInsets(top: 30, leading: 0, bottom: 30, trailing: 0))
    }
}
```

## 三、使用图片组件

      在SwiftUI中，可以方便的布局图片元素，并设置图片的圆角，阴影，边框等。使用如下代码即可在界面中间布局出一个图片元素：

```swift
struct ContentImage:View {
    var body: some View {
        Image("demo")
    }
}
```

其中“demo”为工程中一个资源图片的名字。预览效果如下：

![](https://oscimg.oschina.net/oscnet/up-7bab0b7e7c0655ac8b0fbad18c067b9e12b.png)

图片元素也有很多属性可以设置，如clipShape设置元素的形状，shadow设置元素的阴影，示例代码如下：

```swift
struct ContentImage:View {
    var body: some View {
        Image("demo")
        .clipShape(Circle())
        .shadow(radius: 30)
    }
}
```

效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-1683dcdf11dac7dd20f6480d2877c25c5b2.png)

## 四、在SwiftUI中使用UIKit组件

      上面我们使用的文本与图片元素，都是SwiftUI框架中定义的基础组件。在实际开发中的更多时候，你需要结合UIKit来自定义一个SwiftUI组件，这本身也非常方便。例如我们要使用UIKit中的UILabel组件，示例如下：

```swift
struct Label:UIViewRepresentable {
    func makeUIView(context: Context) -> UILabel {
        UILabel(frame: .zero)
     }
    
    func updateUIView(_ uiView: UILabel, context: Context) {
        uiView.text = "Hello"
    }
}
```

UIViewRepresentable协议用来将UIKit中的组件定义为一个SwiftUI元素，其中有两个方法是必须要实现的，makeUIView用来返回一个指定的UIKit组件，updateUIView当组件更新时会被调用。

> 有了这些基础知识，我们已经可以使用SwiftUI来实现简单的页面构建了，你可以尝试用其做些简单的页面组合，只需要把握住，在SwiftUI中，无论简单还是复杂的界面都是通过水平和和垂直栈的组合，加上组件的位置偏移所布局出来的。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：805263726

---
title: SwiftUI直通车系列（3）—— 使用导航
date: 2020-09-27
categories: SwiftUI
tags: []
---
# SwiftUI直通车系列三（3）—— 使用导航

    关于SwiftUI，我们前两篇博客介绍了独立组件的布局与属性设置相关内容，并且介绍了开发中最常用的列表视图的使用。但是一个完整的应用程序不可能是单界面的，如何使用SwiftUI进行界面间的导航跳转，是我们本博客讨论的重点。前两篇博客地址如下：

[SwiftUI直通车系列（1）—— 视图的布局与组织](https://my.oschina.net/u/2340880/blog/4529951)

[SwiftUI直通车系列（2）—— 列表视图](https://my.oschina.net/u/2340880/blog/4534222)

    在本系列的第二篇博客中，我们能够使用SwiftUI来构建列表视图，通常，列表视图用来展示目录页面，当用户点击列表中的某一项时，需要跳转到详情页。在iOS应用中，页面的跳转常用导航控制器进行管理。在SwiftUI中，创建一个导航也是非常简单的，只需要在组件最外层嵌套NavigationView即可。以我们之前的列表视图Demo为例，改写代码如下：

```swift
import SwiftUI
import UIKit

struct ContactModel {
    var name:String
    var phone:String
}

let modelData = [
    ContactModel(name:"王小丫", phone:"15137348888"),
    ContactModel(name:"李小二", phone:"15137348989")
]

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

struct Label:UIViewRepresentable {
    func makeUIView(context: Context) -> UILabel {
        UILabel(frame: .zero)
     }
    
    func updateUIView(_ uiView: UILabel, context: Context) {
        uiView.text = "Hello"
    }
}


struct ContentImage:View {
    var body: some View {
        Image("demo")
        .clipShape(Circle())
        .shadow(radius: 30)
    }
}

struct RowContent:View {
    
    var contactModel:ContactModel
    
    var body: some View {
        HStack(alignment:.top) {
            Image("demo").resizable().frame(width: 70, height: 70)
            VStack(alignment:.leading, spacing: 10) {
                Text(self.contactModel.name).bold().font(Font.system(size: 25))
                Text(self.contactModel.phone).font(Font.system(size: 20))
            }
            Spacer()
            }.padding(EdgeInsets(top: 10, leading: 20, bottom: 10, trailing: 20))
    }
}

struct ListContent:View {
    var body: some View {
        NavigationView {
            List(modelData, id: \.name) { model in
                RowContent(contactModel: model)
            }
            .navigationBarTitle("通讯录")
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ListContent()
    }
}
```

你只需要关注ListContent结构体即可，其中为列表视图设置了导航，并且定义了导航栏上的标题，效果如下图：

![](https://oscimg.oschina.net/oscnet/up-7ae66a20edea05be5ac94708d99761c004e.png)

要实现页面的跳转，需要为要触发跳转的组件添加NavigationLink包裹，NavigationLink可以指向要跳转的详情页面，如下：

```swift
struct ListContent:View {
    var body: some View {
        NavigationView {
            List(modelData, id: \.name) { model in
                NavigationLink(destination: ContentImage()) {
                    RowContent(contactModel: model)
                }
            }
            .navigationBarTitle("通讯录")
        }
    }
}
```

如果详情页面是动态的，在跳转时，我们也可以将数据传递过去，如下：

```swift
struct ContentImage:View {
    var name:String
    var phone:String
    var body: some View {
        VStack() {
            Image("demo")
                .clipShape(Circle())
                .shadow(radius: 30)
                .offset(x: 0, y: -70)
            Text(name).offset(x: 0, y: -50)
            Text(phone).offset(x: 0, y: -30)
        }
    }
}

struct ListContent:View {
    var body: some View {
        NavigationView {
            List(modelData, id: \.name) { model in
                NavigationLink(destination: ContentImage(name: model.name, phone: model.phone)) {
                    RowContent(contactModel: model)
                }
            }
            .navigationBarTitle("通讯录")
        }
    }
}
```

效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-5f3c221291140f8332a10a7729da2ed4521.png)

默认导航的标题是大文字风格的，若要使用常规风格的，需要配置其displayMode属性，如下：

```swift
struct ListContent:View {
    var body: some View {
        NavigationView {
            List(modelData, id: \.name) { model in
                NavigationLink(destination: ContentImage(name: model.name, phone: model.phone)) {
                    RowContent(contactModel: model)
                }
            }
            .navigationBarTitle("通讯录", displayMode: .inline)
        }
    }
}
```

效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-3566cc83b2270ba0869397723bb5d97d66b.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：805263726

---
title: SwiftUI直通车系列（2）—— 列表视图
date: 2020-08-28
categories: SwiftUI
tags: []
---
# SwiftUI直通车系列（2）—— 列表视图

    列表视图的开发中非常常用的页面元素。SwiftUI中也有专门用来渲染列表的元素提供。

## 一、编写行视图

      列表实际上是一组行视图的组合，在布局列表视图之前，你首先需要定义好行视图的布局。例如，我们使用一个Image元素和两个Text元素来布局一个简单的联系人行视图。

```swift
struct RowContent:View {
    var body: some View {
        HStack(alignment:.top) {
            Image("demo").resizable().frame(width: 70, height: 70)
            VStack(alignment:.leading, spacing: 10) {
                Text("王小丫").bold().font(Font.system(size: 25))
                Text("15137344444").font(Font.system(size: 20))
            }
            Spacer()
            }.padding(EdgeInsets(top: 10, leading: 20, bottom: 10, trailing: 20))
    }
}
```

在预览界面上与布局情况进行预览，如下图：

![](https://oscimg.oschina.net/oscnet/up-93dd88b3e90c1cdec6f4cb8df1b354b16a9.png)      

## 二、关联数据

     列表中展示的数据往往是一组相似类型的数据。以上联系人行视图为例，我们可以定义一组联系人数据来填充到列表的行视图中。首先定义一个结构体用来描述联系人信息，如下：

```swift
struct ContactModel {
    var name:String
    var phone:String
}

let modelData = [
    ContactModel(name:"王小丫", phone:"15137348888"),
    ContactModel(name:"李小二", phone:"15137348989")
]
```

如上代码所示，其中ContactModel定义了联系人的基本信息，modelData是一组联系人模型，实际应用中，modelData的数据来源可能是网络，也可能是本地文件。修改RowContent代码如下：

```swift
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
```

SwiftUI的实时预览功能也支持对一组组件进行预览，示例如下：

```swift
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            RowContent(contactModel: modelData[0])
            RowContent(contactModel: modelData[1])
        }.previewLayout(.fixed(width: 400, height: 80))
    }
}
```

效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-630df8c7f8a427714e24d9e193e7d5effc3.png)

## 三、使用列表组件

     SwiftUI中使用List组件来构建列表，将布局好的列表行视图嵌入其中即可展示出列表界面，如下：

```swift
struct ListContent:View {
    var body: some View {
        List {
           RowContent(contactModel: modelData[0])
           RowContent(contactModel: modelData[1])
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ListContent()
    }
}
```

此时，预览效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-6df25d463d533b28527a6016f8fc8ccd362.png)

在实际开发中，一般我会采用动态的方式来构建列表，通过对数据源的便利，可以循环生成列表行，示例如下：

```objectivec
struct ListContent:View {
    var body: some View {
        List(modelData, id: \.name) { model in
           RowContent(contactModel: model)
        }
    }
}
```

其中id是一个行标识字段，使用数据源中能够保证唯一的字段来设置即可。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：805263726

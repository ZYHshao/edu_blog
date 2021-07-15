---
title: SwiftUI直通车系列（6）—— 使用动画
date: 2020-11-30
categories: SwiftUI
tags: []
---
# SwiftUI直通车系列（6）—— 使用动画

     本系列博客是针对SwiftUI开发框架的快速入门介绍，之前系列博客地址：

[SwiftUI直通车系列（1）—— 视图的布局与组织](https://my.oschina.net/u/2340880/blog/4529951)

[SwiftUI直通车系列（2）—— 列表视图](https://my.oschina.net/u/2340880/blog/4534222)

[SwiftUI直通车系列三（3）—— 使用导航](http://xn--swiftui%283%29%20-5n6ga9439p5gfhqnmm1cjh7gx4h5r8ao6y1m4etxd/)

[SwiftUI直通车系列（4）—— 处理用户交互](https://my.oschina.net/u/2340880/blog/4654523)

[SwiftUI直通车系列（5）—— 自定义绘制](https://my.oschina.net/u/2340880/blog/4698216)

前面的博客整体对使用SwiftUI进行页面布局与简单的用户交互做了介绍。我们知道，作为一个优秀的UI框架，其很重要的一点是对动画的支持，动画是提升用户体验的重要方式，在SwiftUI中，实现动画效果将变得非常容易。

## 一、使用属性动画

     属性动画是指当组件的属性发生变化时所展示动画效果，例如组件的颜色、位置、尺寸、旋转角度等。在SwiftUI中，我们知道要动态的对组件样式进行更改，需要使用状态来控制。我们也可以使用状态来控制动画动作。

      例如，我们定义一个按钮组件，当点击时，让其旋转180度，并略微放大，代码如下：

```swift
struct DrawContent:View {
    @State private var begin = false
    var body: some View {
        VStack(alignment: .center, spacing: 20) {
            Button(action: {
                self.begin.toggle()
            }) {
                Text("开始")
                    .font(Font.system(size: 30))
                    .rotationEffect(.degrees(self.begin ? 180 : 0))
                    .scaleEffect(self.begin ? 2 : 1)
                    .animation(.easeInOut(duration: 3))
                                        
            }
        }
    }
}
```

其中，begin状态属性用来记录动画的状态，当一次点击此按钮时，会将当前按钮放大一倍，并旋转180度，再次点击按钮会动画还原。

      rotationEffect用来定义组件的旋转角度，scaleEffect用来定义组件缩放的比例，与之类似，大部分用来渲染视图样式的属性都支持进行动画。animation用来设置动画的执行方式，如上代码所示，设置的为easeInOut表示动画执行的过程采用渐入渐出的方式，且动画执行的时长为3秒。

## 二、使用转场动画

      属性动画用在组件的某些属性发生变化的场景，转场动画会影响组件的切换动作。当对组件进行展示或隐藏时，通常可以采用转场动画来处理。我们以上一篇博客自定义的图形组件为例，我们可以为其出现时加一个滑动动画，示例代码如下：

```swift
extension Animation {
    static func custom() -> Animation {
        Animation.spring(response: 1.5, dampingFraction: 0.5, blendDuration: 0.5)
            .speed(2)
            .delay(0.03)
    }
}

struct DrawContent:View {
    @State private var begin = false
    var body: some View {
        VStack(alignment: /*@START_MENU_TOKEN@*/.center/*@END_MENU_TOKEN@*/, spacing: 20) {
            Button(action: {
                withAnimation {
                    self.begin.toggle()
                }
            }) {
                Text("开始")
                    .font(Font.system(size: 30))
                    .rotationEffect(.degrees(self.begin ? 180 : 0))
                    .scaleEffect(self.begin ? 2 : 1)
                    .animation(.easeInOut(duration: 3))
                                        
            }
            if self.begin {
                ZStack{
                    ForEach(0 ..< 10) { i in
                        DrawTestView(angle: Angle(degrees:Double(36*i))).opacity( 0.3)
                    }
                }.animation(.custom())
                .transition(.slide)
            }

        }

    }
}
```

如上代码所示，其中custom是我们自定义的一种动画模式，上面采用了阻尼动画，并自定义了动画的阻尼参数，速度，延时等属性。

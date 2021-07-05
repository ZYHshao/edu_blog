---
title: Swift学习第一练——用Swift实现的FlappyBird小游戏
date: 2015-06-11
categories: COME ON SWIFT
tags: []
---
## 用Swift实现的FlappyBird小游戏

      伴随着apple公司对swift的推广态度深入，swift火的很快，并且swift精简便捷的语法和强大的功能，对于使用Object—C开发iOS的开发者来说，也有必要了解学习一下swift。这篇博客跳过swift干涩的语法，直接从一个小游戏项目开始使用swift，将其中收获总结如下：

    FlappyBird是前段时间很火的一款小游戏，通过手指点击屏幕平衡小鸟通过障碍。我是将以前OC版的项目拿来改成了swift，所以整体的思路还是OC的开发思路。

    首先，我需要定义两个宏，一个用来模拟重力加速度G，一个用来便捷获取设备屏幕尺寸。因为这个游戏非常简单，开发起来也只需要几个小时，所以我们只需要在一个文件中写代码：viewController.swift。

    swift中没有一般语言中的宏定义，但是可以通过定义常量的形式实现宏的效果：

```
//用常量的形式代理OC中的宏定义
let G:Float=9.8
let SCREEN_SIZE = UIScreen.mainScreen().bounds
```

    我们需要定义一些成员变量，如下：

```
class ViewController: UIViewController {
    var timer:NSTimer?//背景移动的定时器
    var i:Int=0//背景移动的速度
    var timer2:NSTimer?//柱子和地面移动的定时器
    var timer3:NSTimer?//小鸟移动的定时器
    var bird:UIImageView?
    var t:Float=0.0//小鸟下落的速度
    var isDowm:Bool=false//标记小鸟是否在下落
    var isGameOver:Bool=false//标记是否游戏结束
}
```

    对于？和！号的理解，网上概念很多，简单理解声明变量时如果不初始化系统是不会给变量赋nil的，会报错，？的作用就是告诉系统这里如果没有初始化就是nil。同理，在用这类变量的时候，也需要加上？解包，如果加！就是强制解包，可以理解为让系统认为这个变量一定不是nil。

    对于UI的创建等部分函数和OC一样，只是调用的方式略有不同，后面会附上源码。

    在控制小鸟下落的部分代码如下，其中有一点需要注意，在swift中没有隐式转换这个概念，比如你要使用int a + float b 你必须手动将int转为float:(Float)(a)+b

```
func birdMove(){
        if !isDowm{
            if bird?.frame.origin.y < SCREEN_SIZE.height-100{
                var rant:CGRect=bird!.frame
                rant.origin.y += (CGFloat)(G*(t*t/2))
                bird?.frame=rant
                t+=0.025
            }
        }else{
            if t<0.24{
                var rant = bird?.frame
                rant?.origin.y -= 4.9-(CGFloat)(G*t*t/2)
                bird?.frame=rant!
                t+=0.025
            }else{
                isDowm=false
            }
        }
    }
```

  游戏效果图如下：

![](http://static.oschina.net/uploads/space/2015/0611/165325_TOIO_2340880.png)

![](http://static.oschina.net/uploads/space/2015/0611/165343_JoFU_2340880.png)

我相信，实践是学习的必经途径，希望与志同道合的朋友，一起进步。

项目github地址：[https://github.com/ZYHshao/swiftFlappyBird](https://github.com/ZYHshao/swiftFlappyBird)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

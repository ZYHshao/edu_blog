---
title: AppleWatch开发入门九——Watch帧动画的实现
date: 2015-10-19
categories: Apple Watch开发手记
tags: []
---
## AppleWatch开发入门九——Watch帧动画的实现

        动画一直是iOS系统的一大亮点，CoreAnimation和粒子效果的支持，开发者可以很容易的做出效果炫酷的动画特效。在watchOS中，由于性能和屏幕尺寸的限制，对于动画，并没有强大的框架支持，但是这并不是说开发者就没办法在watch上添加动画的特效了。在watchOS中唯一可以让开发者用于动画操作的就是帧动画。

        和iOS类似，watchOS中的真动画也是通过UIImage对象的合集来展示的。只是设置和用法略有不同。

        首先，watchOS中帧动画的操作被单独封装成了一个协议，当然，WKInterfaceImage类是遵守了这个协议的：

```
public protocol WKImageAnimatable : NSObjectProtocol {
    //从默认帧开始播放动画
    public func startAnimating()
    //播放一个指定范围的帧动画 NSRange是帧的范围，durtion是播放一遍的时间，repeatCount是重复播放次数，0为无限循环
    public func startAnimatingWithImagesInRange(imageRange: NSRange, duration: NSTimeInterval, repeatCount: Int)
    //停止播放动画
    public func stopAnimating()
}
```

创建帧动画的步骤与一些注意：

1、关联一个视图中的WKInterfaceImage对象

2、所有帧动画的图片帧必须有统一的格式：比如image1.png，image2.png等等

3、给WKInterfaceImage对象设置帧前缀：

```
imageInterface.setImageNamed("image")
```

注意：这里使用的方法和设置图片的方法一样，但是参数有别，图片的设置需要完整的图片名，动画帧前缀的设置只要设置帧图片的前缀。

4、开始动画：

```
 imageInterface.startAnimatingWithImagesInRange(NSRange(location: 1, length: 3), duration: 3, repeatCount: 0)
```

注意：素材帧必须放入watchKit App这个Target中，才可以使用。 

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

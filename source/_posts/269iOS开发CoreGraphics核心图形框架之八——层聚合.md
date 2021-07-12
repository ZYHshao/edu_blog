---
title: iOS开发CoreGraphics核心图形框架之八——层聚合
date: 2016-12-04
categories: iOS逻辑初窥
tags: []
---
## iOS开发CoreGraphics核心图形框架之八——层聚合

    正常情况下，在使用CoreGraphics框架中的方法进行图形绘制时，每一闭合的图形都是一个独立的层，如果在绘制时添加了阴影效果，则通过阴影可以很明显的看到图形的分层情况，后绘制的图形在上层，先绘制的图形在下层，示例代码如下：

```objectivec
-(void)drawRect:(CGRect)rect{
    float width = rect.size.width/2;
    CGPoint center = CGPointMake(rect.size.width/2, rect.size.height/2);
    CGSize          myShadowOffset = CGSizeMake (10, -20);
    CGContextRef myContext = UIGraphicsGetCurrentContext();
    //设置阴影
    CGContextSetShadow (myContext, myShadowOffset, 10);
    //绘制三个圆形
    CGContextSetRGBFillColor (myContext, 0, 1, 0, 1);
    CGContextFillEllipseInRect(myContext, CGRectMake(center.x-width/2, center.y-width/4*3, width, width));
    CGContextSetRGBFillColor (myContext, 0, 0, 1, 1);
    CGContextFillEllipseInRect(myContext, CGRectMake(center.x-width/4, center.y-width/4, width, width));
    CGContextSetRGBFillColor (myContext, 1, 0, 0, 1);
    CGContextFillEllipseInRect(myContext, CGRectMake(center.x-width/4*3, center.y-width/4, width, width));
}
```

运行效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1204/095100_Oh7l_2340880.png)

从图中可以发现，所绘制的3个圆形并非是在同一层级上，有时开发者可能需要绘制边界复杂的图形，还以上面的例子来说，如果开发者需要绘制某个图形的边界是有3个圆形拼接而成，出现这样的层级效果是不合理的。CoreGraphics框架中也提供了进行图形聚合绘制的方法，示例如下：

```objectivec
-(void)drawRect:(CGRect)rect{
    float width = rect.size.width/2;
    CGPoint center = CGPointMake(rect.size.width/2, rect.size.height/2);
    CGSize          myShadowOffset = CGSizeMake (10, -20);
    CGContextRef myContext = UIGraphicsGetCurrentContext();
    CGContextSetShadow (myContext, myShadowOffset, 10);
    CGContextBeginTransparencyLayer (myContext, NULL);
    //开启图形聚合绘制
    //之后的绘制代码都将绘制到统一层上
    CGContextSetRGBFillColor (myContext, 0, 1, 0, 1);
    CGContextFillEllipseInRect(myContext, CGRectMake(center.x-width/2, center.y-width/4*3, width, width));
    CGContextSetRGBFillColor (myContext, 0, 0, 1, 1);
    CGContextFillEllipseInRect(myContext, CGRectMake(center.x-width/4, center.y-width/4, width, width));
    CGContextSetRGBFillColor (myContext, 1, 0, 0, 1);
    CGContextFillEllipseInRect(myContext, CGRectMake(center.x-width/4*3, center.y-width/4, width, width));
    //结束聚合绘制
    CGContextEndTransparencyLayer (myContext);
}
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1204/095446_NRk0_2340880.png)

有了聚合绘制这样的方法，进行复杂图形的绘制将更加灵活！

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

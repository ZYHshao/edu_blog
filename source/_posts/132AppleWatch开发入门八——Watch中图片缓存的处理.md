---
title: AppleWatch开发入门八——Watch中图片缓存的处理
date: 2015-10-19
categories: Apple Watch开发手记
tags: []
---
## AppleWatch开发入门八——Watch中图片缓存的处理

        由于iWatch在存储和性能上都和iPhone有着很大的差距，这就要求开发者对程序有更高的性能优化，下载与传输图像，在Watch操作中是一个非时的过程，因此，watchOS中为我们提供了一个缓存图片的框架，并且接口和使用都非常简单。

        WatchOS中缓存图片的方法封装在WKInterfaceDevice这个类中，其中添加图片进入缓存的方法如下：

```
//添加一个UIImage对象进入缓存目录，设置name，当我们设置图片时，可以直接通过name进行设置
public func addCachedImage(image: UIImage, name: String) -> Bool
//添加一个Data图片进入缓存目录，设置name，当我们设置图片时，可以直接通过name进行设置
public func addCachedImageWithData(imageData: NSData, name: String) -> Bool
//上面两个方法的返回值用于判断缓存是否成功，因为watch缓存目录的大小有限，可能会失败
```

同样，我们也可以将已经缓存的图片数据删除掉：

```
 //根据name删除一个图片数据
 public func removeCachedImageWithName(name: String)
 //删除缓存目录中所有的图片数据
 public func removeAllCachedImages()
```

我们也可以通过下面的方法获取所有缓存图片的name值：

```
//下面这个函数返回一个字典，string为缓存图片的name值，NSNumber为相应的图片大小，单位为b
public var cachedImages: [String : NSNumber] { get }
```

注意：系统缓存目录的大小为20M，如果缓存失败，可以尝试删掉旧的缓存。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: watchOS中进行异步图片加载和缓存的策略
date: 2015-10-24
categories: Apple Watch开发手记
tags: []
---
## watchOS中进行异步图片加载和缓存的策略

### 一、引言

        iWatch是智能手表的一次革命。iWatch的应用也将会越来越多，基于watch的一些特点，watchOS的开发者需要更加精益的把握watch的UI和性能。运用watchOS自带的缓存体系进行数据的缓存，是增强用户体验度的一种方式，这篇博客，介绍在watchOS中进行异步加载图片和缓存的方法，愿与志同道合的朋友，一起交流。

关于watchOS中的缓存框架，在这里：[http://my.oschina.net/u/2340880/blog/519023   ](http://my.oschina.net/u/2340880/blog/519023)。

### 二、存储的命名规则

        在进行设计之前，我们应该先了解，watchOS的缓存容量为最大20M，因为有限，我们更应该认真的利用每一份空间，因此，缓存我们不仅可以存，在即将装满的时候，我们还要有办法从缓存中删去一些东西，让出空间，那么应该删除哪些东西了，我们应该都可以想到，当然是旧的了，把最早的缓存删掉，所以，在存的时候，我们要设计一种规则，可以保存存入的时间，并且不影响我寻找这个缓存文件。我的方法是通过格式化的命名：

```
//这是一个规范缓存命名的方法
func checkString(str:NSString)->NSString{
    let result:NSMutableString=NSMutableString()
    //先将所有的非字母和数字剔除掉
    for var i=0 ; i<str.length ; i++ {
        if (str.characterAtIndex(i)>=48&&str.characterAtIndex(i)<=57)||(str.characterAtIndex(i)>=65&&str.characterAtIndex(i)<=90)||(str.characterAtIndex(i)>=97&&str.characterAtIndex(i)<=122){
            result.appendFormat("%c",str.characterAtIndex(i))
        }
    }
    //拼接上当前时间戳
    let date:Double = NSDate().timeIntervalSince1970
    result.appendFormat("?%.0f",date)
    return result
}
```

通过？符号将名称和时间戳进行了拼接。

### 二、进行异步加载图片和缓存

        这一步是如下的设计思路：通过图片url从缓存的路径中进行寻找，如果有，直接取出图片，如果没有，开启一个线程进行异步加载，完成后刷新主线程UI并将图片文件规范命名后进行缓存：

```
//进行存取缓存的操作
//取出watchOS的缓存目录
let imagedic:NSDictionary = WKInterfaceDevice().cachedImages as NSDictionary
    //取图片存储的名称
    let imageUrl:NSMutableString=NSMutableString()
        //这里的url是外界传进来的图片地址url，进行去掉特殊字符
        for var i=0 ; i<url?.length ; i++ {
            if (url?.characterAtIndex(i)>=48&&url?.characterAtIndex(i)<=57)||(url?.characterAtIndex(i)>=65&&url?.characterAtIndex(i)<=90)||(url?.characterAtIndex(i)>=97&&url?.characterAtIndex(i)<=122){
                imageUrl.appendFormat("%c",(url?.characterAtIndex(i))!)
             }
        }
        //查找缓存中是否有图片
        //遍历watchOS的缓存目录
        for var i=0 ; i<imagedic.allKeys.count ; i++ {
           //通过规定好的？进行分割
           let str:NSArray =  imagedic.allKeys[i].componentsSeparatedByString("?")
                if str[0].isEqualToString(imageUrl as String) {
                    //找到图片 view是要设置的interfaceImage
                    view.setImageNamed(imagedic.allKeys[i] as? String)
                    return;
                }
            }
            //设置缺省图片 这里是外界传进来的缺省图片，如果需要下载，先设置缺省图片
            if defaultImage != nil {
                view.setImageNamed(defaultImage as? String)
            }
            
            //进行下载和存储
            let dispath = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0)
            //在新的线程中下载
            dispatch_async(dispath, { () -> Void in
                let imgURL:NSURL = NSURL(string: url as! String)!
                let imageData:NSData? = NSData(contentsOfURL: imgURL)
                if imageData != nil {
                    //主线程中刷新
                    dispatch_async(dispatch_get_main_queue(), { () -> Void in
                        view.setImageData(imageData!)
                    })
                    //写缓存  如果缓存满了 就删掉时间戳最早的一张缓存
                    //这个方法会返回bool值，判断是否存入成功
                    while !WKInterfaceDevice().addCachedImageWithData(imageData!, name: checkString(url!) as String) {
                        //如果存入失败，删去时间戳最早的缓存
                        var temp:NSString?
                        //保存最早的缓存名称
                        var result:NSString?
                        for var i=0 ; i<imagedic.allKeys.count ; i++ {
                            let str:NSArray =  imagedic.allKeys[i].componentsSeparatedByString("?")
                            if temp == nil {
                                temp = str[1] as? NSString
                                result = imagedic.allKeys[i] as! String
                                break
                            }
                            if str[1].doubleValue < temp?.doubleValue {
                                //找到更早的缓存
                                temp = str[1] as? NSString
                                result = imagedic.allKeys[i] as! String
                            }
                        }
                        //删掉缓存
                        WKInterfaceDevice().removeCachedImageWithName(result as! String)
                    }
                }
            })
```

上面的代码和注释，已经介绍了所有的思路，有错误之处或者更好的方式，还望多多指点。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

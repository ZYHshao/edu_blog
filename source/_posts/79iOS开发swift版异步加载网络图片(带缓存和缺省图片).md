---
title: iOS开发swift版异步加载网络图片(带缓存和缺省图片)
date: 2015-06-25
categories: COME ON SWIFT
tags: []
---
## iOS开发之swift版异步加载网络图片

    与SDWebImage异步加载网络图片的功能相似，只是代码比较简单，功能没有SD的完善与强大，支持缺省添加图片，支持本地缓存。

     异步加载图片的核心代码如下：

```
 func setZYHWebImage(url:NSString?, defaultImage:NSString?, isCache:Bool){
        var ZYHImage:UIImage?
        if url == nil {
            return
        }
        //设置默认图片
        if defaultImage != nil {
            self.image=UIImage(named: defaultImage!)
        }
        //是否进行缓存处理
        if isCache {
        //缓存管理类
            var data:NSData?=ZYHWebImageChcheCenter.readCacheFromUrl(url!)
            if data != nil {
                ZYHImage=UIImage(data: data!)
                self.image=ZYHImage
            }else{
            //获取异步线程
               var dispath=dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0)
                dispatch_async(dispath, { () -> Void in
                    var URL:NSURL = NSURL(string: url!)!
                    var data:NSData?=NSData(contentsOfURL: URL)
                    if data != nil {
                        ZYHImage=UIImage(data: data!)
                        //写缓存
                        ZYHWebImageChcheCenter.writeCacheToUrl(url!, data: data!)
                        //主线程中刷新UI
                        dispatch_async(dispatch_get_main_queue(), { () -> Void in
                            //刷新主UI
                            self.image=ZYHImage
                        })
                    }
                    
                })
            }
        }else{
            var dispath=dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0)
            dispatch_async(dispath, { () -> Void in
                var URL:NSURL = NSURL(string: url!)!
                var data:NSData?=NSData(contentsOfURL: URL)
                if data != nil {
                    ZYHImage=UIImage(data: data!)
                    //写缓存
                    dispatch_async(dispatch_get_main_queue(), { () -> Void in
                        //刷新主UI
                        self.image=ZYHImage
                    })
                }
                
            })
        }
    }
    
}
```

缓存的处理这里采用的是写文件的方式，通过文件名来对缓存进行管理，这个框架还不完善，后面会加入缓存清除等功能。缓存的核心代码如下：

```
class func readCacheFromUrl(url:NSString)->NSData?{
        var data:NSData?
        var path:NSString=ZYHWebImageChcheCenter.getFullCachePathFromUrl(url)
        if NSFileManager.defaultManager().fileExistsAtPath(path) {
            data=NSData.dataWithContentsOfMappedFile(path) as? NSData
        }
        return data
    }
    
    class func writeCacheToUrl(url:NSString, data:NSData){
        var path:NSString=ZYHWebImageChcheCenter.getFullCachePathFromUrl(url)
       println(data.writeToFile(path, atomically: true))
    }
    //设置缓存路径
    class func getFullCachePathFromUrl(url:NSString)->NSString{
        var chchePath=NSHomeDirectory().stringByAppendingString("/Library/Caches/MyCache")
        var fileManager:NSFileManager=NSFileManager.defaultManager()
        fileManager.fileExistsAtPath(chchePath)
        if !(fileManager.fileExistsAtPath(chchePath)) {
            fileManager.createDirectoryAtPath(chchePath, withIntermediateDirectories: true, attributes: nil, error: nil)
        }
        //进行字符串处理
        var newURL:NSString
        newURL=ZYHWebImageChcheCenter.stringToZYHString(url)
        chchePath=chchePath.stringByAppendingFormat("/%@", newURL)
        return chchePath
    }
    
    class func stringToZYHString(str:NSString)->NSString{
        var newStr:NSMutableString=NSMutableString()
        for var i:NSInteger=0; i < str.length; i++ {
            var c:unichar=str.characterAtIndex(i)
            if (c>=48&&c<=57)||(c>=65&&c<=90)||(c>=97&&c<=122){
                newStr.appendFormat("%c", c)
            }
        }
        return newStr.copy() as NSString
        
    }
```

框架的github地址，欢迎指正与扩展：[https://github.com/ZYHshao/swift-ZYHWebImage](https://github.com/ZYHshao/swift-ZYHWebImage)

因xcode的版本不同，swift语言语法随环境时常会变化，此版本在6.1中可用，更高版本中需要修改少部分即可。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

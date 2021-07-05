---
title: Swift学习第二练——Swift项目时光电影
date: 2015-06-28
categories: COME ON SWIFT
tags: []
---
## Swift学习第二练——Swift项目时光电影

    很早以前的一个OC的练习项目，用swift重新写了一遍，因为xcode版本的更新对swift的兼容度也在不断改变，此版本适用于xcode6.1。

    这个项目中，用swift将iOS官方SDK中的HTTP进行了封装，使用了swift编写的异步加载网络图片的方法。练习了用swift操作界面布局，跳转界面等的方法。

    下面是封装的下载类的核心代码：

```
private var httpConnection:NSURLConnection?
class ZYHHttpRequset: NSObject,NSURLConnectionDataDelegate{
    var requestUrl:String?
    var downloadData:NSMutableData=NSMutableData()
    var isDownloadSuccess:Bool?
    var delegate:ZYHHttpRequestDelegate?
    class func requestFormUrl(url:NSString)->ZYHHttpRequset{
        var oldRequest:ZYHHttpRequset?=ZYHHttpRequestManager.sharedHttpRequestManager().requestForKey(url)
        if (oldRequest != nil){
            println("该任务存在")
            return oldRequest!
        }
        //新建下载任务
        var request:ZYHHttpRequset=ZYHHttpRequset()
        request.requestUrl=url
        request.startRequestUrl(url)
        ZYHHttpRequestManager.sharedHttpRequestManager().addTask(request, key: url)
        return request
    }
    
    func stop(){
        if httpConnection != nil {
            httpConnection?.cancel()
            httpConnection = nil
        }
    }
    
    //开始下载请求
    private func startRequestUrl(url:NSString){
        if httpConnection != nil {
            httpConnection!.cancel()
            httpConnection==nil
        }
        //创建连接对象
        var request=NSURLRequest(URL: NSURL(string: url)!)
        httpConnection=NSURLConnection(request: request, delegate: self)
        
    }
    //重写协议中的方法
    func connection(connection: NSURLConnection, didReceiveResponse response: NSURLResponse) {
        downloadData.length=0
    }
    func connection(connection: NSURLConnection, didReceiveData data: NSData) {
        downloadData.appendData(data)
    }
    func connectionDidFinishLoading(connection: NSURLConnection) {
        isDownloadSuccess = true
        delegate!.ZYHHttpRequestSuccsee(self)
        ZYHHttpRequestManager.sharedHttpRequestManager().removeTaskFromUrl(self.requestUrl!)
    }
    func connection(connection: NSURLConnection, didFailWithError error: NSError) {
        println("加载失败")
        println(error)
        self.isDownloadSuccess=false
        ZYHHttpRequestManager.sharedHttpRequestManager().removeTaskFromUrl(self.requestUrl!)
    }
    
    
    
    
}
protocol ZYHHttpRequestDelegate{
    func ZYHHttpRequestSuccsee(request:ZYHHttpRequset)
}
```

项目部分截图：

![](http://static.oschina.net/uploads/space/2015/0628/162314_ZUcS_2340880.png)

![](http://static.oschina.net/uploads/space/2015/0628/162328_uUb2_2340880.png)

![](http://static.oschina.net/uploads/space/2015/0628/162344_jqVp_2340880.png)

![](http://static.oschina.net/uploads/space/2015/0628/162400_Jxdu_2340880.png)

github源码地址：[https://github.com/ZYHshao/SwiftMovie](https://github.com/ZYHshao/SwiftMovie)

其中错误之处，欢迎指教，希望在交流中，不断进步！

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

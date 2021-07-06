---
title: iOS网络编程之二——NSURLSession的简单使用
date: 2016-02-22
categories: iOS逻辑初窥
tags: []
---
## iOS网络编程之二——NSURLSession的简单使用

### 一、NSURLSession简介

    在iOS7之后，NSURLSession作为系统推荐使用的HTTP请求框架，在进行前台请求的情况下，NSURLSession与NSURLConnection并无太大差异，对于后台的请求，NSURLSession更加灵活的优势就将展现无遗。

####         1.NSURLSession集合的类型

        NSURLSession类提供3中Session类型：

        Default类型：提供前台请求相关方法，支持配置缓存，身份凭证等。

        Ephemeral类型：即时的请求类型，不使用缓存，身份凭证等。

        Background：后台类型，支持在后台完成请求任务。

####         2.NSURLSession任务的类型

        在NSURLSession中添加的请求任务支持3中类型：

        数据任务：使用NSData对象进行数据的发送和获取，一般用于短数据的任务。

        下载任务：从文件下载数据，支持后台下载。

        上传任务：以文件的形式上传数据，支持后台上传。

### 二、创建并配置NSURLSession

        通过NSURLSessionConfiguration类对象对NSURLSession进行配置与创建，创建和配NSURLSession的示例代码如下：

```
    //默认类型的
    NSURLSessionConfiguration * defaultConfiguration = [NSURLSessionConfiguration defaultSessionConfiguration];
    //即时类型的
    NSURLSessionConfiguration * ephemeralConfiguration = [NSURLSessionConfiguration ephemeralSessionConfiguration];
    //后台类型的
    NSURLSessionConfiguration * backgroundConfiguration = [NSURLSessionConfiguration backgroundSessionConfigurationWithIdentifier:@"SessionId"];
    
    //创建并设置session
    NSURLSession * defaultSession = [NSURLSession sessionWithConfiguration:defaultConfiguration];
    NSURLSession * ephemeralSession = [NSURLSession sessionWithConfiguration:ephemeralConfiguration];
    NSURLSession * backgroundSession = [NSURLSession sessionWithConfiguration:backgroundConfiguration];
```

NSURLSessionConfiguration还可以配置如缓存，网络模式等参数

### 三、使用NSURLSession进行网络请求的两种方式

        NSURLSession有两种方式进行网络数据的请求，一种是通过block的方式获取网络数据，一种是通过代理回调的方式获取网络数据。通过block的方式进行请求代码如下：

```
    //创建session配置对象
    NSURLSessionConfiguration * defaultConfiguration = [NSURLSessionConfiguration defaultSessionConfiguration];
    //创建请求对象
    NSURLRequest * request = [NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.baidu.com"]];
    //创建session对象
    NSURLSession * defaultSession = [NSURLSession sessionWithConfiguration:defaultConfiguration];
    //添加任务
    NSURLSessionTask * task= [defaultSession dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        NSLog(@"%@",data);
    }];
    //开始任务
    [task resume];
```

使用代理回调的方式进行请求需要遵守如下协议：

```
@interface ViewController ()<NSURLSessionDataDelegate>
@end
```

将请求代码修改如下：

```
    NSURLSessionConfiguration * defaultConfiguration = [NSURLSessionConfiguration defaultSessionConfiguration];
    NSURLRequest * request = [NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.baidu.com"]];
    NSURLSession * defaultSession = [NSURLSession sessionWithConfiguration:defaultConfiguration delegate:self delegateQueue:[NSOperationQueue mainQueue]];

    NSURLSessionTask * task= [defaultSession dataTaskWithRequest:request];
    [task resume];
```

实现代理方法如下：

```
//开始接受数据
-(void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data{
    NSLog(@"=======%@",data);
}
//接受数据结束
-(void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error{
    NSLog(@"完成：error%@",error);
}
```

### 四、进行后台下载任务

        NSURLSession最大的优势在于其后台下载的灵活性，使用如下的代码进行后台数据下载：

```
 NSURLSessionConfiguration * backgroundConfiguration = [NSURLSessionConfiguration backgroundSessionConfigurationWithIdentifier:@"com.zyprosoft.backgroundsession"];
    NSURLRequest * request = [NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.baidu.com"]];
    NSURLSession *  backgroundSession   = [NSURLSession sessionWithConfiguration:backgroundConfiguration delegate:self delegateQueue:nil];
    [[backgroundSession downloadTaskWithRequest:request]resume];
```

在下面的回调方法中可以进行下载进度的监听：

```
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didWriteData:(int64_t)bytesWritten totalBytesWritten:(int64_t)totalBytesWritten totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite
{
    NSLog(@"######");
}
```

如果在下载过程中点击Home键使应用程序进入后台，NSURLSession的相关代理方法将不再被回调，但是下载任务依然在进行，当后台下载完成后会与AppDelegate进行交互，会调用AppDelegate中的如下方法：

```
-(void)application:(UIApplication *)application handleEventsForBackgroundURLSession:(NSString *)identifier completionHandler:(void (^)())completionHandler{
    NSLog(@"1111");
}
```

之后应用程序在后台会调用NSURLSesstion代理的如下方法来通知下载结果：

```
//此方法无论成功失败都会调用
-(void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error{
    NSLog(@"完成：error%@",error);
}
//此方法只有下载成功才会调用 文件放在location位置
-(void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didFinishDownloadingToURL:(NSURL *)location{
    
}
```

最后将调用NSURLSesstion的如下方法：

```
-(void)URLSessionDidFinishEventsForBackgroundURLSession:(NSURLSession *)session
{
    
    NSLog(@"All tasks are finished");
    
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

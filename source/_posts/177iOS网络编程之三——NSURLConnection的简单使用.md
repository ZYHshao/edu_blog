---
title: iOS网络编程之三——NSURLConnection的简单使用
date: 2016-02-22
categories: iOS逻辑初窥
tags: []
---
## iOS网络编程之三——NSURLConnection的简单使用

### 一、引言

    在iOS7后，NSURLSession基本代替了NSURLConnection进行网络开发，在iOS9后，NSURLConnection相关方法被完全的弃用，iOS系统有向下兼容的特性，尽管NSURLConnection已经被弃用，但在开发中，其方法依然可以被使用，并且如果需要兼容到很低版本的iOS系统，有时就必须使用NSURLConnection类了。

### 二、使用NSURLConnection进行同步请求

    对于网络请求分为同步和异步两种，同步是指在请求结果返回之前，程序代码会卡在请求处，之后的代码不会被执行，异步是指在发送请求之后，一边在子线程中接收返回数据，一边执行之后的代码，当返回数据接收完毕后，采用回调的方式通知主线程做处理。

    使用如下方法进行NSURLConnection的同步请求：

```
    NSURL * url = [NSURL URLWithString:@"http://www.baidu.com"];
    NSURLRequest * request = [NSURLRequest requestWithURL:url];
    NSData * data = [NSURLConnection sendSynchronousRequest:request returningResponse:nil error:nil];
    NSLog(@"%@",data);
    NSLog(@"继续执行");
```

打印信息如下图所示，从中可以看出，当数据返回结束时才执行后面的代码：

![](http://static.oschina.net/uploads/space/2016/0222/173912_xvgl_2340880.png)

### 三、使用NSURLConnection进行异步请求

        使用同步的方式进行请求有一个很大的弊端，在进行网络请求时，数据的返回往往需要一定时间，不可能瞬间完成，使用同步的方式将导致界面卡死，没有提示也不能交互任何用户操作，这样的话，很有可能会给用户程序卡死的假象。

        NSURLConnection类提供两种方式进行异步请求操作。

####         1.使用block的方式进行异步请求

        使用如下代码进行block方式的异步请求，在block中会传入请求到的返回数据和数据信息等参数：

```
    NSURL * url = [NSURL URLWithString:@"http://www.baidu.com"];
    NSURLRequest * request = [NSURLRequest requestWithURL:url];
    //其中的queue参数决定block中的代码在哪个队列中执行
    [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse * _Nullable response, NSData * _Nullable data, NSError * _Nullable connectionError) {
        NSLog(@"%@",data);
    }];
    NSLog(@"继续执行");
```

####         2.使用代理回调的异步请求方式

        首先遵守协议与生命一个可变的NSData用于接收数据：

```
@interface ViewController ()<NSURLConnectionDataDelegate>
{
    NSMutableData * _data;
}
@end
```

使用如下的代码进行请求：

```
    _data = [[NSMutableData alloc]init];
    NSURL * url = [NSURL URLWithString:@"http://www.baidu.com"];
    NSURLRequest * request = [NSURLRequest requestWithURL:url];
    [NSURLConnection connectionWithRequest:request delegate:self];
```

请求发出后，会一次调用如下代理方法进行请求过程的监听和数据的获取：

```
-(void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response{
    //开始接收数据
    [_data setLength:0];
}
-(void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data{
    //正在接收数据
    [_data appendData:data];
}
-(void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error{
    //接收数据失败
    NSLog(@"%@",error);
}
-(void)connectionDidFinishLoading:(NSURLConnection *)connection{
    //接收数据完成
    NSLog(@"%@",_data);
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

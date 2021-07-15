---
title: iOS开发之CFNetwork框架使用
date: 2018-10-24
categories: iOS逻辑初窥
tags: []
---
## iOS开发之CFNetwork框架使用

### 一、引言

    在iOS应用开发中，CFNetwork框架其实并不是非常常用的，相对NSURLSession框架而言，这是一个相对底层的网络工作框架。官方文档中的下图描述了CFNetwork在整个网络体系中的位置：

![](https://oscimg.oschina.net/oscnet/3d1f42aabe7e50ae4311ccfb378624023e5.jpg)

CFNetwork与CoreFoundation关系密切，其实基于CoreFoundation框架的，结构如下图所示：

![](https://oscimg.oschina.net/oscnet/2e9543e4dc18f178e22b55322933b44a737.jpg)

本篇博客中不会过多的设计CoreFoundation框架中的内容，主要总结和介绍CFNetwork的相关内容与简单应用。

### 二、使用CFNetwork进行简单的网络请求

      CFNetwork是使用C语言实现的一套网络访问框架，进行一个简单的网络请求示例代码如下：

```objectivec
//创建请求方法字符串    
CFStringRef method = CFSTR("GET");
//创建请求URL字符串
CFStringRef urlStr = CFSTR("http://www.baidu.com");
//创建请求URL对象
CFURLRef url = CFURLCreateWithString(kCFAllocatorDefault, urlStr, NULL);
//创建HTTP消息对象
CFHTTPMessageRef request = CFHTTPMessageCreateRequest(kCFAllocatorDefault, method, url, kCFHTTPVersion1_1);
//进行请求头的设置
CFHTTPMessageSetHeaderFieldValue(request, CFSTR("key"), CFSTR("Value"));
//创建读取流对象
CFReadStreamRef readStream = CFReadStreamCreateForHTTPRequest(kCFAllocatorDefault, request);
//定义读取流上下文
CFStreamClientContext ctxt = {0, (__bridge void *)(self), NULL, NULL, NULL};
//设置读取的客服端 即回调相关
CFReadStreamSetClient(readStream, kCFStreamEventHasBytesAvailable|kCFStreamEventEndEncountered|kCFStreamEventOpenCompleted|kCFStreamEventCanAcceptBytes|kCFStreamEventErrorOccurred, myCallBack, &ctxt);
//将读取流加入runloop中
CFReadStreamScheduleWithRunLoop(readStream, CFRunLoopGetCurrent(), kCFRunLoopCommonModes);
//开启流
printf("%d",CFReadStreamOpen(readStream));
```

实现myCallBack回调函数如下：

```objectivec
void myCallBack (CFReadStreamRef stream,CFStreamEventType type,void *clientCallBackInfo){
    //流中有可读数据的回调  
    if (type == kCFStreamEventHasBytesAvailable) {      
        //将流中的数据存入到数组中
        UInt8 buff [1024];
        CFReadStreamRead(stream, buff, 1024);
        printf("%s",buff);
    //流打开完成的回调
    }else if(type==kCFStreamEventOpenCompleted){
        NSLog(@"open");
    //流异常的回调
    }else if (type==kCFStreamEventErrorOccurred){
        NSLog(@"error:%@",CFErrorCopyDescription( CFReadStreamCopyError(stream)));
    //可以接收写数据时调用
    }else if (type==kCFStreamEventCanAcceptBytes){
        NSLog(@"kCFStreamEventCanAcceptBytes");
    //读取结束回调
    }else if(type==kCFStreamEventEndEncountered){
        NSLog(@"end");
        //关闭流
        CFReadStreamClose(stream);
        //将流从runloop中移除
        CFReadStreamUnscheduleFromRunLoop(stream, CFRunLoopGetCurrent(), kCFRunLoopCommonModes);
    }
}
```

上面演示了简单的GET请求，如果使用的请求方法为POST，则可以进行请求体的设置，上面示例代码中，CFStringRef、CFURLRef、CFReadStreamRef等相关的类为CoreFoundation框架中的，这里暂不深究，简单使用即可。后面我们将详细的探讨CFNetwork中相关类的使用。

### 三、CFHTTPMessageRef详解

    在基于C的框架中，类对象都是使用结构体指针描述的，CFHTTPMessageRef是HTTP消息的封装，其可以是一个HTTP请求，也可以是一个HTTP回执。与其相关的方法解析如下：

```objectivec
//返回CGHTTPMessageRef的类型ID
CFTypeID CFHTTPMessageGetTypeID(void);
//创建一个HTTP请求消息
/*
alloc为内存管理器 一般使用默认的kCFAllocatorDefault
requestMethod为请求方法
url为请求的路径
httpVersion为请求的HTTP版本
HTTP版本定义如下：
kCFHTTPVersion1_0
kCFHTTPVersion1_1
kCFHTTPVersion2_0
*/
CFHTTPMessageRef CFHTTPMessageCreateRequest(CFAllocatorRef __nullable alloc, CFStringRef requestMethod, CFURLRef url, CFStringRef httpVersion);
//创建一个HTTP回执消息
/*
alloc内存管理器
statusCode 请求回执状态码
statusDescription 请求回执状态描述
httpVersion HTTP版本号
*/
CFHTTPMessageRef CFHTTPMessageCreateResponse(CFAllocatorRef  __nullable alloc,CFIndex         statusCode,CFStringRef     __nullable statusDescription,CFStringRef     httpVersion);
//创建一个空的HTTP消息 
/*
isRequest 如果传入kCFBooleanTrue 则为请求类型 否则为回执类型
*/
CFHTTPMessageRef CFHTTPMessageCreateEmpty(CFAllocatorRef __nullable alloc, Boolean isRequest);
//复制一个HTTP消息
CFHTTPMessageRef CFHTTPMessageCreateCopy(CFAllocatorRef __nullable alloc, CFHTTPMessageRef message);
//判断一个HTTP消息是请求 还是 回执
Boolean CFHTTPMessageIsRequest(CFHTTPMessageRef message);
//获取HTTP版本
CFStringRef CFHTTPMessageCopyVersion(CFHTTPMessageRef message);
//获取消息体内容 请求体或者回执体
CFDataRef CFHTTPMessageCopyBody(CFHTTPMessageRef message);
//设置消息体内容
void CFHTTPMessageSetBody(CFHTTPMessageRef message, CFDataRef bodyData);
//获取某个消息头内容
CFStringRef CFHTTPMessageCopyHeaderFieldValue(CFHTTPMessageRef message, CFStringRef headerField);
//获取所有消息头字段
CFDictionaryRef CFHTTPMessageCopyAllHeaderFields(CFHTTPMessageRef message);
//设置消息头
void CFHTTPMessageSetHeaderFieldValue(CFHTTPMessageRef message, CFStringRef headerField, CFStringRef __nullable value);
//向空消息中追加序列化的数据
Boolean CFHTTPMessageAppendBytes(CFHTTPMessageRef message, const UInt8 *newBytes, CFIndex numBytes);
//返回消息头数据是否准备完成
Boolean CFHTTPMessageIsHeaderComplete(CFHTTPMessageRef message);
//将一个消息对象序列化成数据
CFDataRef CFHTTPMessageCopySerializedMessage(CFHTTPMessageRef message);
/*=================下面这些方法针对于请求类型的消息=====================*/
//获取消息中的url
CFURLRef CFHTTPMessageCopyRequestURL(CFHTTPMessageRef request);
//获取消息的请求方法
CFStringRef CFHTTPMessageCopyRequestMethod(CFHTTPMessageRef request);
//添加认证信息
Boolean CFHTTPMessageAddAuthentication(CFHTTPMessageRef   request,CFHTTPMessageRef   __nullable authenticationFailureResponse,CFStringRef        username,CFStringRef        password,CFStringRef        __nullable authenticationScheme,Boolean            forProxy);
/*=================下面这些方法针对于绘制类型的消息=====================*/
//获取回执状态码
CFIndex CFHTTPMessageGetResponseStatusCode(CFHTTPMessageRef response);
//获取回执状态行信息
CFStringRef CFHTTPMessageCopyResponseStatusLine(CFHTTPMessageRef response);
```

### 四、进行请求与回调处理

    CFHTTPMessageRef的主要用途是构建出HTTP的请求或回执对象，请求的相关发起与回调方法都封装在CFHTTPStream.h这个头文件中，解析如下：

```objectivec
//通过一个HTTP请求创建一个读取流对象
CFReadStreamRef CFReadStreamCreateForHTTPRequest(CFAllocatorRef __nullable alloc, CFHTTPMessageRef request);
//通过一个HTTP请求创建读取流对象 但是请求的body会被忽略 取requestBody作为请求体
CFReadStreamRef CFReadStreamCreateForStreamedHTTPRequest(CFAllocatorRef __nullable alloc, CFHTTPMessageRef requestHeaders, CFReadStreamRef requestBody);
//设置读取流是否自动重定向
void CFHTTPReadStreamSetRedirectsAutomatically(CFReadStreamRef httpStream, Boolean shouldAutoRedirect);
```

### 五、关于请求的证书验证

    有时，客户端在向服务端进行请求时收到状态为401的回执，这时往往表明需要客户端提供用户凭证，在CFNetWork框架中，用户凭证与证书验证相关方法封装在CFHTTPAuthentication.h头文件中。解析如下：

```objectivec
//获取CFHTTPAuthentication类ID
CFTypeID CFHTTPAuthenticationGetTypeID(void);
/*
通过一个401或者407的请求回执创建一个 用户认证对象
*/
CFHTTPAuthenticationRef CFHTTPAuthenticationCreateFromResponse(CFAllocatorRef __nullable alloc, CFHTTPMessageRef response);
//获取一个用户认证对象是否有效
Boolean CFHTTPAuthenticationIsValid(CFHTTPAuthenticationRef auth, CFStreamError * __nullable error);
//获取某个用户认证对象是否是某个请求的
Boolean CFHTTPAuthenticationAppliesToRequest(CFHTTPAuthenticationRef auth, CFHTTPMessageRef request);
//获取某个用户认证是否必须有序进行
Boolean CFHTTPAuthenticationRequiresOrderedRequests(CFHTTPAuthenticationRef auth);
//使用给定的用户名和密码执行证书验证方法
Boolean CFHTTPMessageApplyCredentials(CFHTTPMessageRef request,CFHTTPAuthenticationRef auth,CFStringRef  __nullable username,CFStringRef  __nullable    password,CFStreamError *   __nullable    error);
/*
此方法和上面方法作用一致 不同的是使用字典来进行用户名和密码的设置 字段的键如下：
kCFHTTPAuthenticationUsername 用户名键
kCFHTTPAuthenticationPassword 密码键
kCFHTTPAuthenticationAccountDomain 账户域
*/
Boolean CFHTTPMessageApplyCredentialDictionary(CFHTTPMessageRef request,CFHTTPAuthenticationRef   auth,CFDictionaryRef dict,CFStreamError * __nullable  error);
//返回用户凭证的账户域
CFStringRef CFHTTPAuthenticationCopyRealm(CFHTTPAuthenticationRef auth);
//返回一组账户域
CFArrayRef CFHTTPAuthenticationCopyDomains(CFHTTPAuthenticationRef auth);
//返回用户凭证的验证方法
CFStringRef CFHTTPAuthenticationCopyMethod(CFHTTPAuthenticationRef auth);
//获取用户凭证验证是否需要用户名和密码
Boolean CFHTTPAuthenticationRequiresUserNameAndPassword(CFHTTPAuthenticationRef auth);
//获取是否需要账户域
Boolean CFHTTPAuthenticationRequiresAccountDomain(CFHTTPAuthenticationRef auth);
```

### 六、进行FTP协议的数据交换

    CFNetWork框架也支持与FTP协议的服务端进行数据交互，方法解析如下：

```objectivec
//根据URL创建FTP读取流对象 用来进行文件下载
CFReadStreamRef CFReadStreamCreateWithFTPURL(CFAllocatorRef __nullable alloc, CFURLRef ftpURL);
//解析文件或目录的格式化数据
CFIndex CFFTPCreateParsedResourceListing(CFAllocatorRef __nullable alloc, const UInt8 *buffer, CFIndex bufferLength, CFDictionaryRef __nullable *  __nullable parsed);
//根据URL创建一个FTP写入流对象 用来进行文件上传
CFWriteStreamRef CFWriteStreamCreateWithFTPURL(CFAllocatorRef __nullable alloc, CFURLRef ftpURL);
```

对于FTP写入和读取流来说，可以使用CFReadStreamSetProperty()函数或者CFWriteStreamSetProperty()函数来进行属性的设置，可设置的属性列举如下：

```objectivec
kCFStreamPropertyFTPUserName //设置用户名
kCFStreamPropertyFTPPassword //设置密码
kCFStreamPropertyFTPUsePassiveMode //布尔值  设置是否被动模式
kCFStreamPropertyFTPResourceSize  //资源大小
kCFStreamPropertyFTPFileTransferOffset //记录文件位置 用来断点续传
kCFStreamPropertyFTPAttemptPersistentConnection //是否重用连接
kCFStreamPropertyFTPProxy  //设置代理字典
kCFStreamPropertyFTPFetchResourceInfo //资源详情字典
//下面为代理字典中可以定义的键
kCFStreamPropertyFTPProxyHost  //代理主机
kCFStreamPropertyFTPProxyPort  //代理端口
kCFStreamPropertyFTPProxyUser  //代理用户名
kCFStreamPropertyFTPProxyPassword //代理密码
//下面是资源详情字典中可以定义的键
kCFFTPResourceMode  //资源模式
kCFFTPResourceName //资源名
kCFFTPResourceOwne //资源所有者
kCFFTPResourceGroup //资源组
kCFFTPResourceLink  //资源链接
kCFFTPResourceSize  //资源尺寸
kCFFTPResourceType  //资源类型
kCFFTPResourceModDate //修改时间

```

### 七、主机地址相关操作

    CFNetWork中也封装了与主机地址域名相关的操作方法，例如，我们可以通过域名进行DNS解析出IP地址，示例代码如下：

```objectivec
#import <netinet/in.h>
#import <arpa/inet.h>
CFStringRef hostString = CFSTR("www.baidu.com");
CFHostRef host = CFHostCreateWithName(CFAllocatorGetDefault(), hostString);
CFHostStartInfoResolution(host, kCFHostAddresses, NULL);
CFArrayRef addresses = CFHostGetAddressing(host, NULL);
for (int i = 0; i<CFArrayGetCount(addresses); i++) {
    struct sockaddr_in * ip;
    ip = (struct sockaddr_in *)CFDataGetBytePtr(CFArrayGetValueAtIndex(addresses, i));
    printf("%s\n",inet_ntoa(ip->sin_addr));
}
```

CFHostRef对象操作相关方法解析如下：

```objectivec
//获取类型ID
CFTypeID CFHostGetTypeID(void);
//根据域名创建CFHostRef对象
CFHostRef CFHostCreateWithName(CFAllocatorRef __nullable allocator, CFStringRef hostname);
/*
根据地址创建CFHostRef对象
addr参数为sockaddr结构体数据
*/
CFHostRef CFHostCreateWithAddress(CFAllocatorRef __nullable allocator, CFDataRef addr);
//CFHostRef对象的复制
CFHostRef CFHostCreateCopy(CFAllocatorRef __nullable alloc, CFHostRef host);
//对指定主机进行信息预查找 返回值标明是否查找成功
Boolean CFHostStartInfoResolution(CFHostRef theHost, CFHostInfoType info, CFStreamError * __nullable error);
//获取主机的地址列表 数组中为sockaddr结构体数据
CFArrayRef CFHostGetAddressing(CFHostRef theHost, Boolean * __nullable hasBeenResolved);
//获取主机名列表
CFArrayRef CFHostGetNames(CFHostRef theHost, Boolean * __nullable hasBeenResolved);
//获取主机可达性信息
CFDataRef CFHostGetReachability(CFHostRef theHost, Boolean * __nullable hasBeenResolved);
//取消未完成的解析
/*
解析类型枚举
typedef CF_ENUM(int, CFHostInfoType) {
  //地址
  kCFHostAddresses              = 0,
  //主机名
  kCFHostNames                  = 1,
  //可达性信息
  kCFHostReachability           = 2
};
*/
void CFHostCancelInfoResolution(CFHostRef theHost, CFHostInfoType info);
//设置客户端回调
Boolean CFHostSetClient(CFHostRef theHost, CFHostClientCallBack __nullable clientCB, CFHostClientContext * __nullable clientContext);
//注册进Runloop
void CFHostScheduleWithRunLoop(CFHostRef theHost, CFRunLoopRef runLoop, CFStringRef runLoopMode);
//从Runloop中注销
void CFHostUnscheduleFromRunLoop(CFHostRef theHost, CFRunLoopRef runLoop, CFStringRef runLoopMode);
```

### 八、后续

    上面介绍的内容更多还是关于使用CFNetWork框架进行HTTP或FTP请求的相关方法，其实CFNetWork框架中还提供了复杂的Bonjour服务功能，其与CFNetService相关，这部分内容后面有时间再进行整理总结吧。

> 欢迎交流  珲少 QQ 316045346

---
title: 从AFNetworking源码分析到应用全解
date: 2017-11-24
categories: iOS第三方库
tags: []
---
## 从AFNetworking源码分析到应用全解

### 一、引言

    AFNetworking是iOS/OS开发中常用的一个第三方网络库，可以说它是目前最流行的网络库，但其代码结构其实并不复杂，也可以说非常简洁优美。在AFNetworking中，大量使用的线程安全的开发技巧，读此源码也是一次很好的多线程学习机会。本篇博客从主要结构和网络请求的主流程进行分享，解析了AFNetworking的设计思路与工作原理，后面还有其中提供的UI扩展包的接口应用总结。

    每次读优秀的代码都是一次深刻的学习，每一次模仿，都是创造的开始！

### 二、核心源码分析

    平时我们在使用AFNetworking框架时，大多只使用其中的请求管理功能。其实，这个有名的框架中还提供了许多其他的工具，除了可以方便的进行网络安全验证，请求数据与回执数据的序列化，网络状态茶台等基础应用外，还提供了UIKit工具包，其中提供有常用组件的扩展，图片下载器和缓存器等。

    对于AFNetworking框架的核心，无非AFURLSesstionManager类，这个类是基于系统的NSURLSesstion回话类进行的管理者包装，下图是AF框架一个整体的结构。

![](https://static.oschina.net/uploads/space/2017/1123/150005_OJqn_2340880.png)

把握这个结构，我们再来学习AF框架将变得十分容易上手，打开AFURLSesstionManager类，你会发现它有1200多行代码，但是AFURLSesstionManager类真正的实现确实从500多行开始的，之前的代码是内部的代理处理类，就像在MVVM模式中，我们总是喜欢将控制器的逻辑放入View-Model中一样，AFURLSesstionManager实例也会将通知，回调等操作交给这个代理实例处理。

#### 1.AFURLSesstionManager

    首先先来看AFURLSesstionManager类的初始化方法：

```objectivec
- (instancetype)initWithSessionConfiguration:(NSURLSessionConfiguration *)configuration {
    self = [super init];
    if (!self) {
        return nil;
    }
    if (!configuration) {
        //使用默认
        configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
    }
    self.sessionConfiguration = configuration;
    //代理方法执行做在的操作队列
    self.operationQueue = [[NSOperationQueue alloc] init];
    self.operationQueue.maxConcurrentOperationCount = 1;
    //创建sesstion会话
    self.session = [NSURLSession sessionWithConfiguration:self.sessionConfiguration delegate:self delegateQueue:self.operationQueue];
    //默认将返回数据按照json进行解析
    self.responseSerializer = [AFJSONResponseSerializer serializer];
    //使用默认的安全验证
    self.securityPolicy = [AFSecurityPolicy defaultPolicy];
#if !TARGET_OS_WATCH
    //使用默认的网络状态监测
    self.reachabilityManager = [AFNetworkReachabilityManager sharedManager];
#endif
    //每一个请求任务都对应有一个delegate 存在这里面
    self.mutableTaskDelegatesKeyedByTaskIdentifier = [[NSMutableDictionary alloc] init];
    //初始化锁
    self.lock = [[NSLock alloc] init];
    self.lock.name = AFURLSessionManagerLockName;
    //下面的做法是为了防止 被修改了sesstion的系统实现有初始任务
    [self.session getTasksWithCompletionHandler:^(NSArray *dataTasks, NSArray *uploadTasks, NSArray *downloadTasks) {
        for (NSURLSessionDataTask *task in dataTasks) {
            [self addDelegateForDataTask:task uploadProgress:nil downloadProgress:nil completionHandler:nil];
        }

        for (NSURLSessionUploadTask *uploadTask in uploadTasks) {
            [self addDelegateForUploadTask:uploadTask progress:nil completionHandler:nil];
        }

        for (NSURLSessionDownloadTask *downloadTask in downloadTasks) {
            [self addDelegateForDownloadTask:downloadTask progress:nil destination:nil completionHandler:nil];
        }
    }];

    return self;
}
```

用AFURLSesstionManager创建请求任务有三类，当然也对应NSURLSesstion中的数据请求任务，上传任务和下载任务。我们以数据请求任务为例，核心方法如下：

```objectivec
- (NSURLSessionDataTask *)dataTaskWithRequest:(NSURLRequest *)request
                               uploadProgress:(nullable void (^)(NSProgress *uploadProgress)) uploadProgressBlock
                             downloadProgress:(nullable void (^)(NSProgress *downloadProgress)) downloadProgressBlock
                            completionHandler:(nullable void (^)(NSURLResponse *response, id _Nullable responseObject,  NSError * _Nullable error))completionHandler {

    __block NSURLSessionDataTask *dataTask = nil;
    url_session_manager_create_task_safely(^{
        //创建任务
        dataTask = [self.session dataTaskWithRequest:request];
    });
    //进行代理设置
    [self addDelegateForDataTask:dataTask uploadProgress:uploadProgressBlock downloadProgress:downloadProgressBlock completionHandler:completionHandler];

    return dataTask;
}
```

上传任务和下载任务的创建源码和上面大同小异，只是创建出的任务类型不同，它们都要进行下一步代理的设置，还以数据请求任务的代理设置为例，源码如下：

```objectivec
- (void)addDelegateForDataTask:(NSURLSessionDataTask *)dataTask
                uploadProgress:(nullable void (^)(NSProgress *uploadProgress)) uploadProgressBlock
              downloadProgress:(nullable void (^)(NSProgress *downloadProgress)) downloadProgressBlock
             completionHandler:(void (^)(NSURLResponse *response, id responseObject, NSError *error))completionHandler
{
    //创建内部代理
    AFURLSessionManagerTaskDelegate *delegate = [[AFURLSessionManagerTaskDelegate alloc] init];
    delegate.manager = self;
    delegate.completionHandler = completionHandler;
    dataTask.taskDescription = self.taskDescriptionForSessionTasks;
    //设置代理 
    [self setDelegate:delegate forTask:dataTask];

    delegate.uploadProgressBlock = uploadProgressBlock;
    delegate.downloadProgressBlock = downloadProgressBlock;
}

```

需要注意，原生的NSURLSesstionTask的代理其实依然是AFURLSesstionManager类自身，这里Manager作为中介来进行方法的传递(它也会自己处理一些回调，与开发者相关的回调才会转给内部代理处理)。

    上面的流程就是AFURLSesstionManager创建的任务的主流程了，需要注意，它只创建出任务并不会执行，需要开发者手动调用resume才能激活任务。下面的流程就是系统的NSURLSesstionTaskDelagate相关的回调了：

```objectivec
//接收到URL重定向时回调
- (void)URLSession:(NSURLSession *)session
              task:(NSURLSessionTask *)task
willPerformHTTPRedirection:(NSHTTPURLResponse *)response
        newRequest:(NSURLRequest *)request
 completionHandler:(void (^)(NSURLRequest *))completionHandler
{
    NSURLRequest *redirectRequest = request;
    //这里会执行用户自定义的重定向block
    if (self.taskWillPerformHTTPRedirection) {
        redirectRequest = self.taskWillPerformHTTPRedirection(session, task, response, request);
    }
    //继续重定向后的请求
    if (completionHandler) {
        completionHandler(redirectRequest);
    }
}
//接收到安全认证挑战的回调
- (void)URLSession:(NSURLSession *)session
              task:(NSURLSessionTask *)task
didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge
 completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition, NSURLCredential *credential))completionHandler
{
    NSURLSessionAuthChallengeDisposition disposition = NSURLSessionAuthChallengePerformDefaultHandling;
    __block NSURLCredential *credential = nil;
    //如果开发者有提供block 交开发者处理 否则用默认的安全验证方案处理
    if (self.taskDidReceiveAuthenticationChallenge) {
        disposition = self.taskDidReceiveAuthenticationChallenge(session, task, challenge, &credential);
    } else {
        if ([challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust]) {
            if ([self.securityPolicy evaluateServerTrust:challenge.protectionSpace.serverTrust forDomain:challenge.protectionSpace.host]) {
                disposition = NSURLSessionAuthChallengeUseCredential;
                credential = [NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust];
            } else {
                disposition = NSURLSessionAuthChallengeCancelAuthenticationChallenge;
            }
        } else {
            disposition = NSURLSessionAuthChallengePerformDefaultHandling;
        }
    }
    if (completionHandler) {
        completionHandler(disposition, credential);
    }
}
//需要提供数据流传向服务器时调用
- (void)URLSession:(NSURLSession *)session
              task:(NSURLSessionTask *)task
 needNewBodyStream:(void (^)(NSInputStream *bodyStream))completionHandler
{
    NSInputStream *inputStream = nil;

    if (self.taskNeedNewBodyStream) {
        inputStream = self.taskNeedNewBodyStream(session, task);
    } else if (task.originalRequest.HTTPBodyStream && [task.originalRequest.HTTPBodyStream conformsToProtocol:@protocol(NSCopying)]) {
        inputStream = [task.originalRequest.HTTPBodyStream copy];
    }

    if (completionHandler) {
        completionHandler(inputStream);
    }
}
//已经发送数据后调用
- (void)URLSession:(NSURLSession *)session
              task:(NSURLSessionTask *)task
   didSendBodyData:(int64_t)bytesSent
    totalBytesSent:(int64_t)totalBytesSent
totalBytesExpectedToSend:(int64_t)totalBytesExpectedToSend
{

    int64_t totalUnitCount = totalBytesExpectedToSend;
    if(totalUnitCount == NSURLSessionTransferSizeUnknown) {
        NSString *contentLength = [task.originalRequest valueForHTTPHeaderField:@"Content-Length"];
        if(contentLength) {
            totalUnitCount = (int64_t) [contentLength longLongValue];
        }
    }

    if (self.taskDidSendBodyData) {
        self.taskDidSendBodyData(session, task, bytesSent, totalBytesSent, totalUnitCount);
    }
}
//请求任务完成时回调
- (void)URLSession:(NSURLSession *)session
              task:(NSURLSessionTask *)task
didCompleteWithError:(NSError *)error
{
    //拿到内部代理 交给它处理
    AFURLSessionManagerTaskDelegate *delegate = [self delegateForTask:task];

    // delegate may be nil when completing a task in the background
    if (delegate) {
        [delegate URLSession:session task:task didCompleteWithError:error];

        [self removeDelegateForTask:task];
    }

    if (self.taskDidComplete) {
        self.taskDidComplete(session, task, error);
    }
}
//接收到返回数据时回调
- (void)URLSession:(NSURLSession *)session
          dataTask:(NSURLSessionDataTask *)dataTask
didReceiveResponse:(NSURLResponse *)response
 completionHandler:(void (^)(NSURLSessionResponseDisposition disposition))completionHandler
{
    NSURLSessionResponseDisposition disposition = NSURLSessionResponseAllow;

    if (self.dataTaskDidReceiveResponse) {
        disposition = self.dataTaskDidReceiveResponse(session, dataTask, response);
    }

    if (completionHandler) {
        completionHandler(disposition);
    }
}
//此请求将要变更成下载任务时回调
- (void)URLSession:(NSURLSession *)session
          dataTask:(NSURLSessionDataTask *)dataTask
didBecomeDownloadTask:(NSURLSessionDownloadTask *)downloadTask
{   
    //重新设置内部代理
    AFURLSessionManagerTaskDelegate *delegate = [self delegateForTask:dataTask];
    if (delegate) {
        [self removeDelegateForTask:dataTask];
        [self setDelegate:delegate forTask:downloadTask];
    }

    if (self.dataTaskDidBecomeDownloadTask) {
        self.dataTaskDidBecomeDownloadTask(session, dataTask, downloadTask);
    }
}
//开始获取数据时调用
- (void)URLSession:(NSURLSession *)session
          dataTask:(NSURLSessionDataTask *)dataTask
    didReceiveData:(NSData *)data
{
    //交给内部代理处理
    AFURLSessionManagerTaskDelegate *delegate = [self delegateForTask:dataTask];
    [delegate URLSession:session dataTask:dataTask didReceiveData:data];

    if (self.dataTaskDidReceiveData) {
        self.dataTaskDidReceiveData(session, dataTask, data);
    }
}
//将要缓存回执数据时调用
- (void)URLSession:(NSURLSession *)session
          dataTask:(NSURLSessionDataTask *)dataTask
 willCacheResponse:(NSCachedURLResponse *)proposedResponse
 completionHandler:(void (^)(NSCachedURLResponse *cachedResponse))completionHandler
{
    NSCachedURLResponse *cachedResponse = proposedResponse;

    if (self.dataTaskWillCacheResponse) {
        cachedResponse = self.dataTaskWillCacheResponse(session, dataTask, proposedResponse);
    }

    if (completionHandler) {
        completionHandler(cachedResponse);
    }
}
//下载任务结束后回调
- (void)URLSession:(NSURLSession *)session
      downloadTask:(NSURLSessionDownloadTask *)downloadTask
didFinishDownloadingToURL:(NSURL *)location
{
    AFURLSessionManagerTaskDelegate *delegate = [self delegateForTask:downloadTask];
    if (self.downloadTaskDidFinishDownloading) {
        NSURL *fileURL = self.downloadTaskDidFinishDownloading(session, downloadTask, location);
        if (fileURL) {
            delegate.downloadFileURL = fileURL;
            NSError *error = nil;
            //数据位置移动
            [[NSFileManager defaultManager] moveItemAtURL:location toURL:fileURL error:&error];
            if (error) {
                [[NSNotificationCenter defaultCenter] postNotificationName:AFURLSessionDownloadTaskDidFailToMoveFileNotification object:downloadTask userInfo:error.userInfo];
            }

            return;
        }
    }
     //内部代理进行后续处理
    if (delegate) {
        [delegate URLSession:session downloadTask:downloadTask didFinishDownloadingToURL:location];
    }
}

```

到此，AFURLSesstionManager类的任务基本完成，头文件中的接口更多提供了上述回调的设置还有些通知的发送。下面我们要来看下内部代理AFURLSesstionManagerTaskDelegate类的作用，需要注意AFURLSesstionManagerTaskDelegate是一个类，并不是协议，其除了处理上面介绍的Manager转发过来的回到外，还会进行请求进度的控制。其配置方法和一些监听这里不再过多介绍，主要来看其对Manager转发过来的回到的处理：

```objectivec
//接收到数据后 将数据进行拼接
- (void)URLSession:(__unused NSURLSession *)session
          dataTask:(__unused NSURLSessionDataTask *)dataTask
    didReceiveData:(NSData *)data
{
    [self.mutableData appendData:data];
}
//请求完成后
- (void)URLSession:(__unused NSURLSession *)session
              task:(NSURLSessionTask *)task
didCompleteWithError:(NSError *)error
{
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wgnu"
    __strong AFURLSessionManager *manager = self.manager;

    __block id responseObject = nil;
    //配置一个信息字典
    __block NSMutableDictionary *userInfo = [NSMutableDictionary dictionary];
    userInfo[AFNetworkingTaskDidCompleteResponseSerializerKey] = manager.responseSerializer;
    //拿到请求结果数据
    NSData *data = nil;
    if (self.mutableData) {
        data = [self.mutableData copy];
        //We no longer need the reference, so nil it out to gain back some memory.
        self.mutableData = nil;
    }
    //下载信息配置
    if (self.downloadFileURL) {
        userInfo[AFNetworkingTaskDidCompleteAssetPathKey] = self.downloadFileURL;
    } else if (data) {
        userInfo[AFNetworkingTaskDidCompleteResponseDataKey] = data;
    }
    //错误信息配置
    if (error) {
        userInfo[AFNetworkingTaskDidCompleteErrorKey] = error;

        dispatch_group_async(manager.completionGroup ?: url_session_manager_completion_group(), manager.completionQueue ?: dispatch_get_main_queue(), ^{
            if (self.completionHandler) {
                self.completionHandler(task.response, responseObject, error);
            }

            dispatch_async(dispatch_get_main_queue(), ^{
                [[NSNotificationCenter defaultCenter] postNotificationName:AFNetworkingTaskDidCompleteNotification object:task userInfo:userInfo];
            });
        });
    } else {
        dispatch_async(url_session_manager_processing_queue(), ^{
            NSError *serializationError = nil;
            //用回执数据序列化类进行数据处理
            responseObject = [manager.responseSerializer responseObjectForResponse:task.response data:data error:&serializationError];

            if (self.downloadFileURL) {
                responseObject = self.downloadFileURL;
            }

            if (responseObject) {
                userInfo[AFNetworkingTaskDidCompleteSerializedResponseKey] = responseObject;
            }

            if (serializationError) {
                userInfo[AFNetworkingTaskDidCompleteErrorKey] = serializationError;
            }

            dispatch_group_async(manager.completionGroup ?: url_session_manager_completion_group(), manager.completionQueue ?: dispatch_get_main_queue(), ^{
                if (self.completionHandler) {
                    self.completionHandler(task.response, responseObject, serializationError);
                }

                dispatch_async(dispatch_get_main_queue(), ^{
                    [[NSNotificationCenter defaultCenter] postNotificationName:AFNetworkingTaskDidCompleteNotification object:task userInfo:userInfo];
                });
            });
        });
    }
#pragma clang diagnostic pop
}
//下载处理
- (void)URLSession:(NSURLSession *)session
      downloadTask:(NSURLSessionDownloadTask *)downloadTask
didFinishDownloadingToURL:(NSURL *)location
{
    NSError *fileManagerError = nil;
    self.downloadFileURL = nil;

    if (self.downloadTaskDidFinishDownloading) {
        self.downloadFileURL = self.downloadTaskDidFinishDownloading(session, downloadTask, location);
        if (self.downloadFileURL) {
            [[NSFileManager defaultManager] moveItemAtURL:location toURL:self.downloadFileURL error:&fileManagerError];

            if (fileManagerError) {
                [[NSNotificationCenter defaultCenter] postNotificationName:AFURLSessionDownloadTaskDidFailToMoveFileNotification object:downloadTask userInfo:fileManagerError.userInfo];
            }
        }
    }
}
```

#### 2.AFHTTPSesstionManager

    了解了AFURLSesstionManager，学习AFHTTPSesstionManager将十分轻松，这个类只是前者面向应用的一层封装。我们可以先从它的接口看起，这也是开发者最熟悉和常用的部分。

```objectivec
@interface AFHTTPSessionManager : AFURLSessionManager <NSSecureCoding, NSCopying
//基础URL
@property (readonly, nonatomic, strong, nullable) NSURL *baseURL;
//请求序列化对象
@property (nonatomic, strong) AFHTTPRequestSerializer <AFURLRequestSerialization> * requestSerializer;
//回执序列化对象
@property (nonatomic, strong) AFHTTPResponseSerializer <AFURLResponseSerialization> * responseSerializer;
//创建一个默认的Manager实例  注意不是单例
+ (instancetype)manager;
//初始化方法
- (instancetype)initWithBaseURL:(nullable NSURL *)url;
- (instancetype)initWithBaseURL:(nullable NSURL *)url
           sessionConfiguration:(nullable NSURLSessionConfiguration *)configuration NS_DESIGNATED_INITIALIZER;
//进行get请求
- (nullable NSURLSessionDataTask *)GET:(NSString *)URLString
                   parameters:(nullable id)parameters
                      success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                      failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure DEPRECATED_ATTRIBUTE;

- (nullable NSURLSessionDataTask *)GET:(NSString *)URLString
                            parameters:(nullable id)parameters
                              progress:(nullable void (^)(NSProgress *downloadProgress))downloadProgress
                               success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                               failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure;

//进行HEAD请求
- (nullable NSURLSessionDataTask *)HEAD:(NSString *)URLString
                    parameters:(nullable id)parameters
                       success:(nullable void (^)(NSURLSessionDataTask *task))success
                       failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure;

//进行POST请求
- (nullable NSURLSessionDataTask *)POST:(NSString *)URLString
                    parameters:(nullable id)parameters
                       success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                       failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure DEPRECATED_ATTRIBUTE;

- (nullable NSURLSessionDataTask *)POST:(NSString *)URLString
                             parameters:(nullable id)parameters
                               progress:(nullable void (^)(NSProgress *uploadProgress))uploadProgress
                                success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                                failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure;

- (nullable NSURLSessionDataTask *)POST:(NSString *)URLString
                    parameters:(nullable id)parameters
     constructingBodyWithBlock:(nullable void (^)(id <AFMultipartFormData> formData))block
                       success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                       failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure DEPRECATED_ATTRIBUTE;

- (nullable NSURLSessionDataTask *)POST:(NSString *)URLString
                             parameters:(nullable id)parameters
              constructingBodyWithBlock:(nullable void (^)(id <AFMultipartFormData> formData))block
                               progress:(nullable void (^)(NSProgress *uploadProgress))uploadProgress
                                success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                                failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure;

//进行PUT请求
- (nullable NSURLSessionDataTask *)PUT:(NSString *)URLString
                   parameters:(nullable id)parameters
                      success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                      failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure;

//进行PATCH请求
- (nullable NSURLSessionDataTask *)PATCH:(NSString *)URLString
                     parameters:(nullable id)parameters
                        success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                        failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure;

//进行DELETE请求
- (nullable NSURLSessionDataTask *)DELETE:(NSString *)URLString
                      parameters:(nullable id)parameters
                         success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                         failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure;
```

上面所有的请求，一旦创建就会自动resume，开发则不需要额外开启。

#### 3.请求序列化AFURLRequestSerialization

    AFURLRequestSerialization是一个协议，它的作用其实就是来将请求进行配置，其中只定义了一个接口：

```objectivec
@protocol AFURLRequestSerialization <NSObject, NSSecureCoding, NSCopying>
//将请求和参数进行配置后返回
- (nullable NSURLRequest *)requestBySerializingRequest:(NSURLRequest *)request
                               withParameters:(nullable id)parameters
                                        error:(NSError * _Nullable __autoreleasing *)error NS_SWIFT_NOTHROW;

@end
```

实现这个接口的类主要是AFHTTPRequestSerizlizaer类，其还有两个子类，分别为AFJSONRequestSerizlizaer和AFPropertyListRequestSerializer类。

    在使用AFNetworking进行网络请求时，如果你有过抓包，你一定会发现，在发送的普通HTTP请求的HEAD中默认包含了许多信息，其实这些都是AFHTTPRequestSerizlizaer类做的，他默认会向请求头中添加Accept-Language，User-Agent以及其他可配置的头部信息(缓存，Cookie，超时时间，用户名密码等)。在进行请求参数配置的时候，AFHTTPRequestSerizlizaer会根据请求方法来选择配置到url后面或者添加到请求body中(HEAD，DELETE，GET会追加URL，其他添加body)。AFJSONRequestSerizlizaer的作用与AFHTTPRequestSerizlizaer一致，不同的是会将请求头中的Content-Type设置为application/json并且将参数格式化成JSON数据放置在请求体中，AFPropertyListRequestSerializer类则是将Content-Type设置为application/x-plist，并且将参数格式化成Plist数据放入请求体。

#### 4.回执数据序列化AFURLResponseSerialization

    AFNetworking进行网络请求有一个十分方便的地方在于它可以直接将返回数据进行解析。其中AFHTTPResponseSerializer是最基础的解析类，它只会根据返回头信息来校验返回数据的有效性，整理后直接将原数据返回。AFJSONResponseSerializer类用来解析返回数据为JSON数据的回执，用这个类进行解析时，返回头信息中的MIMEType必须为application/json，text/json或text/javascript。AFXMLParserResponseSerializer类用来解析XML数据，其会返回一个XML解析器，使用它时，返回头信息中的MIMEType必须为application/xml或text/xml。AFXMLDocumentResponseSerializer类将返回数据解析成XML文档。AFPropertyListResponseSerializer用来将返回数据解析成Plist数据。AFImageResponseSerializer类用来将返回数据解析成UIImage图片，其支持的MIMEType类型为image/tiff，image/jpeg，image/gif，image/png，image/ico，image/x-icon，image/bmp，image/x-bmp，image/x-xbitmap，image/x-win-bitmap。

    除了上面列出的这些类外，还有一个AFCompoundResponseSerializer类，这个类示例中可以配置多个ResponseSerializer实例，解析的时候会进行遍历尝试找到可以解析的模式，这种模式也叫混合解析模式。

### 三、UI工具包源码分析

#### 1.AFAutoPurgingImageCache图片缓存源码分析

    AFAutoPurgingImageCache类是AF框架中提供的图片缓存器，需要注意，它并不是一个持久化的缓存工具，只做临时性的缓存。其中封装了自动清缓存和按时间命中的逻辑。

    每一个AFAutoPurgingImageCache类实例中都有一个缓存池，缓存池有两个临界值，最大容量与期望容量。当实际使用的内存超过最大容量时，缓存池会自动清理到期望容量。在缓存池中，存放的实际上是AFCacheImage对象，这个内部类对UIImage进行了包装，如下：

```objectivec
@interface AFCachedImage : NSObject
//关联的UIImage
@property (nonatomic, strong) UIImage *image;
//id
@property (nonatomic, strong) NSString *identifier;
//图片大小
@property (nonatomic, assign) UInt64 totalBytes;
//最后使用时间
@property (nonatomic, strong) NSDate *lastAccessDate;
//这个属性没有使用到
@property (nonatomic, assign) UInt64 currentMemoryUsage;
@end

```

清缓存的逻辑十分简单，这里就不做代码解析，流程是每次进行图片缓存时，判断是否超出缓存池最大容量，如果超出，将AFCacheImage对象按照lastAccessDate属性进行排序后进行按顺序删除直到到达期望容量。当收到系统的内存警告时，也会唤起清除内存操作。

#### 2.AFImageDownloader图片下载器源码解析

    AFImageDownloader类专门用来下载图片，其但单独的线程的进行图片的下载处理，并且内置了多任务挂起等待和图片数据缓存的特性。AFImageDownloder类的核心有两个线程：

```objectivec
@property (nonatomic, strong) dispatch_queue_t synchronizationQueue;
@property (nonatomic, strong) dispatch_queue_t responseQueue;
```

其中synchronizationQueue是一个同步线程，用来创建与开始下载任务，也可以理解这个串行线程为这个下载器类的主要代码执行所在的线程，responseQueue是一个并行线程，其用来当请求完成后处理数据。默认情况下，下载器可以同时下载4张图片，如果图片的请求大于4，多出的请求会被暂时挂起，等待其他请求完成在进行激活。其核心流程如下图所示：

![](https://static.oschina.net/uploads/space/2017/1122/172019_WRIu_2340880.png)

如上图所示，AFImageDownloader类中有大量的操作任务池和修改激活任务数的操作，为了保证数据的安全，这也就是为何AFImageDownloader的主题操作要在其自建的串行线程中执行。核心方法代码解析如下：

```objectivec
- (nullable AFImageDownloadReceipt *)downloadImageForURLRequest:(NSURLRequest *)request
                                                  withReceiptID:(nonnull NSUUID *)receiptID
                                                        success:(nullable void (^)(NSURLRequest *request, NSHTTPURLResponse  * _Nullable response, UIImage *responseObject))success
                                                        failure:(nullable void (^)(NSURLRequest *request, NSHTTPURLResponse * _Nullable response, NSError *error))failure {
    __block NSURLSessionDataTask *task = nil;
    //在串行线程中执行
    dispatch_sync(self.synchronizationQueue, ^{
        //取当前图片的url作为标识
        NSString *URLIdentifier = request.URL.absoluteString;
        //检查url的可用性
        if (URLIdentifier == nil) {
            if (failure) {
                NSError *error = [NSError errorWithDomain:NSURLErrorDomain code:NSURLErrorBadURL userInfo:nil];
                dispatch_async(dispatch_get_main_queue(), ^{
                    failure(request, nil, error);
                });
            }
            return;
        }

        //检查任务池中是否已经有此任务
        AFImageDownloaderMergedTask *existingMergedTask = self.mergedTasks[URLIdentifier];
        if (existingMergedTask != nil) {
            //已经存在此任务 则追加回调 之后返回  这样做的目的是 先后两次对相同图片的请求 可以只进行一次请求，并且执行不同的两次回调
            AFImageDownloaderResponseHandler *handler = [[AFImageDownloaderResponseHandler alloc] initWithUUID:receiptID success:success failure:failure];
            [existingMergedTask addResponseHandler:handler];
            task = existingMergedTask.task;
            return;
        }

        //进行缓存的检查
        switch (request.cachePolicy) {
            case NSURLRequestUseProtocolCachePolicy:
            case NSURLRequestReturnCacheDataElseLoad:
            case NSURLRequestReturnCacheDataDontLoad: {
                UIImage *cachedImage = [self.imageCache imageforRequest:request withAdditionalIdentifier:nil];
                if (cachedImage != nil) {
                    if (success) {
                        dispatch_async(dispatch_get_main_queue(), ^{
                            success(request, nil, cachedImage);
                        });
                    }
                    return;
                }
                break;
            }
            default:
                break;
        }

        //创建数据请求任务
        NSUUID *mergedTaskIdentifier = [NSUUID UUID];
        NSURLSessionDataTask *createdTask;
        __weak __typeof__(self) weakSelf = self;

        createdTask = [self.sessionManager
                       dataTaskWithRequest:request
                       completionHandler:^(NSURLResponse * _Nonnull response, id  _Nullable responseObject, NSError * _Nullable error) {
                           dispatch_async(self.responseQueue, ^{
                               __strong __typeof__(weakSelf) strongSelf = weakSelf;
                               AFImageDownloaderMergedTask *mergedTask = self.mergedTasks[URLIdentifier];
                               if ([mergedTask.identifier isEqual:mergedTaskIdentifier]) {
                                   mergedTask = [strongSelf safelyRemoveMergedTaskWithURLIdentifier:URLIdentifier];
                                   if (error) {
                                       for (AFImageDownloaderResponseHandler *handler in mergedTask.responseHandlers) {
                                           if (handler.failureBlock) {
                                               dispatch_async(dispatch_get_main_queue(), ^{
                                                   handler.failureBlock(request, (NSHTTPURLResponse*)response, error);
                                               });
                                           }
                                       }
                                   } else {
                                       [strongSelf.imageCache addImage:responseObject forRequest:request withAdditionalIdentifier:nil];

                                       for (AFImageDownloaderResponseHandler *handler in mergedTask.responseHandlers) {
                                           if (handler.successBlock) {
                                               dispatch_async(dispatch_get_main_queue(), ^{
                                                   handler.successBlock(request, (NSHTTPURLResponse*)response, responseObject);
                                               });
                                           }
                                       }
                                       
                                   }
                               }
                               [strongSelf safelyDecrementActiveTaskCount];
                               [strongSelf safelyStartNextTaskIfNecessary];
                           });
                       }];

        //创建处理回调
        AFImageDownloaderResponseHandler *handler = [[AFImageDownloaderResponseHandler alloc] initWithUUID:receiptID
                                                                                                   success:success
                                                                                                   failure:failure];
        //创建图片任务 追加回调
        AFImageDownloaderMergedTask *mergedTask = [[AFImageDownloaderMergedTask alloc]
                                                   initWithURLIdentifier:URLIdentifier
                                                   identifier:mergedTaskIdentifier
                                                   task:createdTask];
        [mergedTask addResponseHandler:handler];
        self.mergedTasks[URLIdentifier] = mergedTask;

        //进行激活或挂起数据请求任务
        if ([self isActiveRequestCountBelowMaximumLimit]) {
            [self startMergedTask:mergedTask];
        } else {
            [self enqueueMergedTask:mergedTask];
        }

        task = mergedTask.task;
    });
    if (task) {
        //将回执返回 用来取消任务
        return [[AFImageDownloadReceipt alloc] initWithReceiptID:receiptID task:task];
    } else {
        return nil;
    }
}
```

#### 3.UIImageView与UIButton两个类别的设计分析

    UIImageView+AFNetworking类别与UIButton+AFNetworking类别是AF中提供了两个加载网络图片的工具类别。这两个类别都为其实例对象关联了一个图片下载器，开发者可以自定义这个下载器也可以使用默认提供的，例如：

```objectivec
+ (AFImageDownloader *)sharedImageDownloader {

#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wgnu"
    return objc_getAssociatedObject(self, @selector(sharedImageDownloader)) ?: [AFImageDownloader defaultInstance];
#pragma clang diagnostic pop
}

+ (void)setSharedImageDownloader:(AFImageDownloader *)imageDownloader {
    objc_setAssociatedObject(self, @selector(sharedImageDownloader), imageDownloader, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
```

虽然头文件中提供了多个设置图片的接口方法，但实际核心方法只有一个，其他方法都是基于这个核心方法时候封装的遍历方法，以UIImageView类别为例，UIButton类别也类似，如图：

![](https://static.oschina.net/uploads/space/2017/1120/154308_FHsH_2340880.png)

进行网络图片设置的过程如下图：

![](https://static.oschina.net/uploads/space/2017/1120/160836_CVJ1_2340880.png)

核心方法代码分析：

```objectivec
- (void)setImageWithURLRequest:(NSURLRequest *)urlRequest
              placeholderImage:(UIImage *)placeholderImage
                       success:(void (^)(NSURLRequest *request, NSHTTPURLResponse * _Nullable response, UIImage *image))success
                       failure:(void (^)(NSURLRequest *request, NSHTTPURLResponse * _Nullable response, NSError *error))failure
{
    //检查请求是否合法
    if ([urlRequest URL] == nil) {
        [self cancelImageDownloadTask];
        self.image = placeholderImage;
        return;
    }
    //检查此请求是否正在进行中
    if ([self isActiveTaskURLEqualToURLRequest:urlRequest]){
        return;
    }
    //取消当前UI组件的下载任务
    [self cancelImageDownloadTask];
    //拿到图片下载器
    AFImageDownloader *downloader = [[self class] sharedImageDownloader];
    //拿到缓存器
    id <AFImageRequestCache> imageCache = downloader.imageCache;
    //检查缓存图片是否存在
    UIImage *cachedImage = [imageCache imageforRequest:urlRequest withAdditionalIdentifier:nil];
    if (cachedImage) {
        if (success) {
            success(urlRequest, nil, cachedImage);
        } else {
            self.image = cachedImage;
        }
        [self clearActiveDownloadInformation];
    } else {
        if (placeholderImage) {
            self.image = placeholderImage;
        }

        __weak __typeof(self)weakSelf = self;
        //取一个随机的UUID作为标志
        NSUUID *downloadID = [NSUUID UUID];
        //任务控制对象
        AFImageDownloadReceipt *receipt;
        receipt = [downloader
                   downloadImageForURLRequest:urlRequest
                   withReceiptID:downloadID
                   success:^(NSURLRequest * _Nonnull request, NSHTTPURLResponse * _Nullable response, UIImage * _Nonnull responseObject) {
                       __strong __typeof(weakSelf)strongSelf = weakSelf;
                       //检查当前的任务id和请求的任务id是否一致
                       if ([strongSelf.af_activeImageDownloadReceipt.receiptID isEqual:downloadID]) {
                           if (success) {
                               success(request, response, responseObject);
                           } else if(responseObject) {
                               strongSelf.image = responseObject;
                           }
                           //清空任务信息
                           [strongSelf clearActiveDownloadInformation];
                       }

                   }
                   failure:^(NSURLRequest * _Nonnull request, NSHTTPURLResponse * _Nullable response, NSError * _Nonnull error) {
                       __strong __typeof(weakSelf)strongSelf = weakSelf;
                        if ([strongSelf.af_activeImageDownloadReceipt.receiptID isEqual:downloadID]) {
                            if (failure) {
                                failure(request, response, error);
                            }
                            [strongSelf clearActiveDownloadInformation];
                        }
                   }];
        //存储任务控制对象
        self.af_activeImageDownloadReceipt = receipt;
    }
}
```

#### 4.AFNetworkActivityIndicatorManager类设计解析

    AFNetworkActivityIndicatorManager类用来管理系统在状态栏上的指示器显示或隐藏。系统状态栏上的指示器提供的接口非常简单，只有一个：

```objectivec
[[UIApplication sharedApplication] setNetworkActivityIndicatorVisible:YES];
```

上面方法的参数决定指示器是否显示。AFNetworkActivityIndicatorManager从两个方向来管理这个指示器，可以开发者手动设置，同时他也会对所有AFNetworking发出的请求状态进行监听，来自动适应决定是否显示指示器。

    以前我在设计全局Loading时，通常直接为他暴漏显隐两个接口，当处理多个并行请求的时候就很尴尬了，因为你无法保证Loading在最后完成的请求结束后再隐藏。 AFNetworkActivityIndicatorManager采用了触发器的设计模式(其实有些像引用计数)，请求来对触发器进行加或减的操作，触发器决定是否触发显示指示器。后面的应用解析中会有具体的接口解释。

### 四、应用全解

#### 1.检测网络状态

    在AFNetworking中提供了管理类AFNetworkReachabilityManager类，这个类专门用于网络状态的检测。其代码量很少，接口设计也十分简单。AFNetworkReachabilityManager类解析如下：

```objectivec
//返回默认的单例对象
+ (instancetype)sharedManager;
//创建一个新的管理对象
+ (instancetype)manager;
+ (instancetype)managerForDomain:(NSString *)domain;
+ (instancetype)managerForAddress:(const void *)address;
- (instancetype)initWithReachability:(SCNetworkReachabilityRef)reachability;
//获取当前的网络状态
/*
typedef NS_ENUM(NSInteger, AFNetworkReachabilityStatus) {
    AFNetworkReachabilityStatusUnknown          = -1, //未知
    AFNetworkReachabilityStatusNotReachable     = 0,  //未连接
    AFNetworkReachabilityStatusReachableViaWWAN = 1,  //移动网路
    AFNetworkReachabilityStatusReachableViaWiFi = 2,  //WIFI
};
*/
@property (readonly, nonatomic, assign) AFNetworkReachabilityStatus networkReachabilityStatus;
//获取当前网络是否连接
@property (readonly, nonatomic, assign, getter = isReachable) BOOL reachable;
//获取当前是否是移动网络
@property (readonly, nonatomic, assign, getter = isReachableViaWWAN) BOOL reachableViaWWAN;
//获取当前是否是WIFI网络
@property (readonly, nonatomic, assign, getter = isReachableViaWiFi) BOOL reachableViaWiFi;
//开启网络状态监听 当网络状态变化时会有回调
- (void)startMonitoring;
//停止进行网络状态监听
- (void)stopMonitoring;
//设置网络状态发生变化后的回调
- (void)setReachabilityStatusChangeBlock:(nullable void (^)(AFNetworkReachabilityStatus status))block;
//格式化的网络状态字符串
- (NSString *)localizedNetworkReachabilityStatusString;
//如果开启了网络状态监听 则网络状态发生变化时也会 发这个通知
FOUNDATION_EXPORT NSString * const AFNetworkingReachabilityDidChangeNotification;
//通知中userinfo字典中对应网络状态的键名
FOUNDATION_EXPORT NSString * const AFNetworkingReachabilityNotificationStatusItem;
```

#### 2.UIButton+AFNetworking类别

    这个类别是AFNetworking中UI部分所提供了一个工具，用来创建远程图片按钮。

```objectivec
//设置共享的下载器对象 用来进行网络图片的下载
+ (void)setSharedImageDownloader:(AFImageDownloader *)imageDownloader;
//获取共享的下载器对象
+ (AFImageDownloader *)sharedImageDownloader;

//下面这些方法用来设置按钮图片
- (void)setImageForState:(UIControlState)state
                 withURL:(NSURL *)url;
- (void)setImageForState:(UIControlState)state
                 withURL:(NSURL *)url
        placeholderImage:(nullable UIImage *)placeholderImage;
- (void)setImageForState:(UIControlState)state
          withURLRequest:(NSURLRequest *)urlRequest
        placeholderImage:(nullable UIImage *)placeholderImage
                 success:(nullable void (^)(NSURLRequest *request, NSHTTPURLResponse * _Nullable response, UIImage *image))success
                 failure:(nullable void (^)(NSURLRequest *request, NSHTTPURLResponse * _Nullable response, NSError *error))failure;
//下面这些方法用来设置按钮背景图片
- (void)setBackgroundImageForState:(UIControlState)state
                           withURL:(NSURL *)url;
- (void)setBackgroundImageForState:(UIControlState)state
                           withURL:(NSURL *)url
                  placeholderImage:(nullable UIImage *)placeholderImage;
- (void)setBackgroundImageForState:(UIControlState)state
                    withURLRequest:(NSURLRequest *)urlRequest
                  placeholderImage:(nullable UIImage *)placeholderImage
                           success:(nullable void (^)(NSURLRequest *request, NSHTTPURLResponse * _Nullable response, UIImage *image))success
                           failure:(nullable void (^)(NSURLRequest *request, NSHTTPURLResponse * _Nullable response, NSError *error))failure;

//取消按钮图片下载任务
- (void)cancelImageDownloadTaskForState:(UIControlState)state;
//取消按钮背景图片下载任务
- (void)cancelBackgroundImageDownloadTaskForState:(UIControlState)state;
```

#### 3.UIImageView+AFNetworking类别

    这个类别是AFNetworking框架提供了UIImageView加载异步图片的方案(更多时候可能会用SDWebImage)。

```objectivec
//设置共享的图片下载器
+ (void)setSharedImageDownloader:(AFImageDownloader *)imageDownloader;
//获取共享的图片下载器
+ (AFImageDownloader *)sharedImageDownloader;

//设置图片
- (void)setImageWithURL:(NSURL *)url;
- (void)setImageWithURL:(NSURL *)url
       placeholderImage:(nullable UIImage *)placeholderImage;
- (void)setImageWithURLRequest:(NSURLRequest *)urlRequest
              placeholderImage:(nullable UIImage *)placeholderImage
                       success:(nullable void (^)(NSURLRequest *request, NSHTTPURLResponse * _Nullable response, UIImage *image))success
                       failure:(nullable void (^)(NSURLRequest *request, NSHTTPURLResponse * _Nullable response, NSError *error))failure;
//取消图片下载任务
- (void)cancelImageDownloadTask;
```

#### 4.状态栏的网络状态指示器

    AFNetworking的UI工具包中提供了AFNetworkActivityIndicatorManager类，这个管理类用来对iOS设备状态栏上活动指示器的显示隐藏进行管理。其提供的接口十分简单，解析如下：

```objectivec
//设置是否有效 
/*
如果设置为YES，则可以手动进行控制器的控制
如果设置为NO，则控制器只会根据网络通知来绝对是否显示
*/
@property (nonatomic, assign, getter = isEnabled) BOOL enabled;
//指示器当前是否显示
@property (readonly, nonatomic, assign, getter=isNetworkActivityIndicatorVisible) BOOL networkActivityIndicatorVisible;
//指示器的延时显示时间 当多久后如果网络请求还没有完成 再进行显示
@property (nonatomic, assign) NSTimeInterval activationDelay;
//设置当请求结束多久后 活动指示器隐藏 默认为0.17s
@property (nonatomic, assign) NSTimeInterval completionDelay;
//获取单例对象
+ (instancetype)sharedManager;
//手动对触发器加1 
//当计数大于0 则显示指示器
- (void)incrementActivityCount;
//手动对触发器减1
//当计数不大于0 则隐藏指示器
- (void)decrementActivityCount;
//设置指示器状态改变的回调
- (void)setNetworkingActivityActionWithBlock:(nullable void (^)(BOOL networkActivityIndicatorVisible))block;
```

#### 5.UIActivityIndicatorView+AFNetworking与UIRefreshControl+AFNetworking类别

    这两个类别可以将一个请求任务绑定到活动指示器或原生刷新组件上，任务的完成与否来决定控件是否展示动画。

```objectivec
//UIActivityIndicatorView+AFNetworking
//将任务绑定到活动指示器上
- (void)setAnimatingWithStateOfTask:(nullable NSURLSessionTask *)task;

//UIRefresh+AFNetworing
//将任务绑定到刷新组件上
- (void)setRefreshingWithStateOfTask:(NSURLSessionTask *)task;
```

#### 6.UIProgressView+AFNetworking类别

    这个类别可以将上传和下载任务绑定到进度条组件上。

```objectivec
//将上传任务绑定
- (void)setProgressWithUploadProgressOfTask:(NSURLSessionUploadTask *)task
                                   animated:(BOOL)animated;
//将下载任务绑定
- (void)setProgressWithDownloadProgressOfTask:(NSURLSessionDownloadTask *)task
                                     animated:(BOOL)animated;
```

#### 7.UIWebView+AFNetworking类别

    这个类别是对UIWebView的一种扩展(由于WebKit，这个类别很少会用到了)，其主要作用是将WebView的直接加载改为先下载本地数据，然后进行本地数据的加载，并可以提供一个进度。

```objectivec
//这个方法是下面方法的封装
- (void)loadRequest:(NSURLRequest *)request
           progress:(NSProgress * _Nullable __autoreleasing * _Nullable)progress
            success:(nullable NSString * (^)(NSHTTPURLResponse *response, NSString *HTML))success
            failure:(nullable void (^)(NSError *error))failure;
//进行网页数据请求，完成后会将网页数据返回 并且自动的进行加载
- (void)loadRequest:(NSURLRequest *)request
           MIMEType:(nullable NSString *)MIMEType
   textEncodingName:(nullable NSString *)textEncodingName
           progress:(NSProgress * _Nullable __autoreleasing * _Nullable)progress
            success:(nullable NSData * (^)(NSHTTPURLResponse *response, NSData *data))success
            failure:(nullable void (^)(NSError *error))failure;
```

#### 8.图片下载器AFImageDownloader的应用

    AFImageDownloader是AF框架中提供的图片下载管理类。其内部封装了一个AFImageDownloadReceipt回执类，这个类实例用来进行正在下载图片任务的取消。AFImageDownloader解析如下：

```objectivec
//图片缓存器
@property (nonatomic, strong, nullable) id <AFImageRequestCache> imageCache;
//请求会话管理类
@property (nonatomic, strong) AFHTTPSessionManager *sessionManager;
//设置下载器属性
/*
下载器可以设置同时下载图片的最大数量 如果超出数量的请求会被暂时挂起
typedef NS_ENUM(NSInteger, AFImageDownloadPrioritization) {
    AFImageDownloadPrioritizationFIFO,//被挂起的请求遵守先挂起的先开始
    AFImageDownloadPrioritizationLIFO//被挂起的请求遵守后挂起的先开始
};
*/
@property (nonatomic, assign) AFImageDownloadPrioritization downloadPrioritizaton;
//获取单例对象
+ (instancetype)defaultInstance;
//获取默认的url缓存器
+ (NSURLCache *)defaultURLCache;
//使用初始化方法新建对象
- (instancetype)initWithSessionManager:(AFHTTPSessionManager *)sessionManager
                downloadPrioritization:(AFImageDownloadPrioritization)downloadPrioritization
                maximumActiveDownloads:(NSInteger)maximumActiveDownloads
                            imageCache:(nullable id <AFImageRequestCache>)imageCache;
//进行图片请求 随机生成uuid
- (nullable AFImageDownloadReceipt *)downloadImageForURLRequest:(NSURLRequest *)request
                                                        success:(nullable void (^)(NSURLRequest *request, NSHTTPURLResponse  * _Nullable response, UIImage *responseObject))success
                                                        failure:(nullable void (^)(NSURLRequest *request, NSHTTPURLResponse * _Nullable response, NSError *error))failure;
//和上面方法一样，使用自定义的uuid
- (nullable AFImageDownloadReceipt *)downloadImageForURLRequest:(NSURLRequest *)request
                                                 withReceiptID:(NSUUID *)receiptID
                                                        success:(nullable void (^)(NSURLRequest *request, NSHTTPURLResponse  * _Nullable response, UIImage *responseObject))success
                                                        failure:(nullable void (^)(NSURLRequest *request, NSHTTPURLResponse * _Nullable response, NSError *error))failure;
```

#### 9.图片缓存器应用

    AF框架中UIKit工具包里的ADAutoPurgingImageCache类用来进行图片缓存，其还封装了自动清缓存的逻辑，缓存的命中则是完全按照使用时间先后为标准。类接口解析如下：

```objectivec
//图片缓存协议
@protocol AFImageCache <NSObject>
//根据id来缓存图片
- (void)addImage:(UIImage *)image withIdentifier:(NSString *)identifier;
//根据id来删除图片
- (BOOL)removeImageWithIdentifier:(NSString *)identifier;
//删除所有图片缓存
- (BOOL)removeAllImages;
//根据id来取缓存
- (nullable UIImage *)imageWithIdentifier:(NSString *)identifier;
@end

//网络图片缓存协议
@protocol AFImageRequestCache <AFImageCache>
//缓存某个url的图片 用url+id的方式
- (void)addImage:(UIImage *)image forRequest:(NSURLRequest *)request withAdditionalIdentifier:(nullable NSString *)identifier;
//移除某个缓存图片
- (BOOL)removeImageforRequest:(NSURLRequest *)request withAdditionalIdentifier:(nullable NSString *)identifier;
//获取缓存
- (nullable UIImage *)imageforRequest:(NSURLRequest *)request withAdditionalIdentifier:(nullable NSString *)identifier;
@end

//缓存器实例类
@interface AFAutoPurgingImageCache : NSObject <AFImageRequestCache>
//缓存空间大小 默认为100M
@property (nonatomic, assign) UInt64 memoryCapacity;
//当缓存超过最大限制时进行清缓存 清缓存后的缓存底线大小设置 默认60M
@property (nonatomic, assign) UInt64 preferredMemoryUsageAfterPurge;
//已用空间大小
@property (nonatomic, assign, readonly) UInt64 memoryUsage;
//初始化方法
- (instancetype)init;
- (instancetype)initWithMemoryCapacity:(UInt64)memoryCapacity preferredMemoryCapacity:(UInt64)preferredMemoryCapacity;

@end

```

>     每次读优秀的代码都是一次深刻的学习，每一次模仿，都是创造的开始！
> 
> ——QQ 316045346 欢迎交流

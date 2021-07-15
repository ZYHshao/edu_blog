---
title: SDWebImage源码分析
date: 2017-12-29
categories: iOS第三方库
tags: []
---
## SDWebImage源码分析 

>     每次读优秀的代码都是一次深刻的学习，每一次模仿，都是创造的开始！
> 
> ——QQ 316045346 欢迎交流

     SDWebImage是iOS开发中非常流行的一个网络图片加载库，如果你观察其源码，会发现其中的文件非常多，虽然文件数很多，但是作者的代码结构和条理却是非清晰。SDWebImage的代码结构基本可以分为3块：应用层类别、核心功能类、工具类与类别。其中我们最常使用的是应用层的类别。例如UIImageView的图片加载，UIButton的图片加载等。

### 一、帮助类与类别的解析

#### 1.NSData+ImageContentType

    这个类别是一个图片数据的格式帮助类，使用它可以方便的获取图片数据的图片格式，其中枚举了常用的图片格式如下：

```objectivec
typedef NS_ENUM(NSInteger, SDImageFormat) {
    SDImageFormatUndefined = -1, //未知格式
    SDImageFormatJPEG = 0,   //jpeg
    SDImageFormatPNG,   //png
    SDImageFormatGIF,   //gif
    SDImageFormatTIFF,  //tiff
    SDImageFormatWebP,  //webp
    SDImageFormatHEIC   //heic
};
```

其原理是根据图片数据的第1个字节码进行分析，不同格式的图像数据在开头都会有一部分的用来表明图像信息的数据块，通过它可以获取图片的具体格式。这个类别中只提供了两个方法：

```objectivec
//获取图像数据格式
+ (SDImageFormat)sd_imageFormatForImageData:(nullable NSData *)data;
//将SDImageFormat转换成CFStringRef
+ (nonnull CFStringRef)sd_UTTypeFromSDImageFormat:(SDImageFormat)format;
```

#### 2、SDWebImageFrame

    这个类是SDWebImage中封装的图像帧类，主要用来创建动画图像。

```objectivec
//当前帧图像
@property (nonatomic, strong, readonly, nonnull) UIImage *image;
//时间
@property (nonatomic, readonly, assign) NSTimeInterval duration;
//初始化方法
+ (instancetype _Nonnull)frameWithImage:(UIImage * _Nonnull)image duration:(NSTimeInterval)duration;
```

#### 3.UIImage的编码与解码

    SDWebImageCoder中定义了一个协议，其中约定了方法来对图像数据进行解码与编码，实现这个协议的主要有SDWebImageIOCoder和SDWebImageGIFCoder。

```objectivec
//数据是否可以进行解码 除了webp类型的 其他类型的图像都可以解码
- (BOOL)canDecodeFromData:(nullable NSData *)data;
//进行图片数据解码
- (nullable UIImage *)decodedImageWithData:(nullable NSData *)data;
//进行增量解码
- (nullable UIImage *)decompressedImageWithImage:(nullable UIImage *)image
                                            data:(NSData * _Nullable * _Nonnull)data
                                         options:(nullable NSDictionary<NSString*, NSObject*>*)optionsDict;
//获取此类型图像是否可以编码
- (BOOL)canEncodeToFormat:(SDImageFormat)format;
//将图片编码为数据
- (nullable NSData *)encodedDataWithImage:(nullable UIImage *)image format:(SDImageFormat)format;
//获取此图像数据是否可以增量解码
- (BOOL)canIncrementallyDecodeFromData:(nullable NSData *)data;
//进行增量解码
- (nullable UIImage *)incrementallyDecodedImageWithData:(nullable NSData *)data finished:(BOOL)finished;
```

#### 4.图像数据预加载

    SDWebImagePrefetcher类提供了图像数据的预加载功能，在进行用户体验优化，需要预加载某些常态图像时，可以用使用这个类。

```objectivec
@interface SDWebImagePrefetcher : NSObject
//管理中心
@property (strong, nonatomic, readonly, nonnull) SDWebImageManager *manager;
//设置最大的同时下载数
@property (nonatomic, assign) NSUInteger maxConcurrentDownloads;
//配置 枚举 设置优先级等
@property (nonatomic, assign) SDWebImageOptions options;
//单例对象
+ (nonnull instancetype)sharedImagePrefetcher;
//构造方法
- (nonnull instancetype)initWithImageManager:(nonnull SDWebImageManager *)manager NS_DESIGNATED_INITIALIZER;
//进行预下载
- (void)prefetchURLs:(nullable NSArray<NSURL *> *)urls;
- (void)prefetchURLs:(nullable NSArray<NSURL *> *)urls
            progress:(nullable SDWebImagePrefetcherProgressBlock)progressBlock
           completed:(nullable SDWebImagePrefetcherCompletionBlock)completionBlock;
//取消预下载行为
- (void)cancelPrefetching;
```

SDWebImagePrefetcher还提供了代理来对预下载过程进行监听，如下：

```objectivec
//当一张图片被下载完后调用
- (void)imagePrefetcher:(nonnull SDWebImagePrefetcher *)imagePrefetcher didPrefetchURL:(nullable NSURL *)imageURL finishedCount:(NSUInteger)finishedCount totalCount:(NSUInteger)totalCount;
//与下载任务全部结束后调用
- (void)imagePrefetcher:(nonnull SDWebImagePrefetcher *)imagePrefetcher didFinishWithTotalCount:(NSUInteger)totalCount skippedCount:(NSUInteger)skippedCount;
```

### 二、核心功能

    SDWebImage的核心功能类结构如下图所示：

![](https://static.oschina.net/uploads/space/2017/1229/135005_K3TI_2340880.png)

#### 1.缓存管理类SDImageCache

    SDImageCache类负责所有网络图片数据的缓存，其从逻辑上分为两级缓存，内存缓存和硬盘缓存。开发者可以使用单例方法来获取默认的SDImageCache实例，也可以使用特殊的Name值来创建缓存实例，常用函数列举如下：

```objectivec
//缓存图片到内存和磁盘
- (void)storeImage:(nullable UIImage *)image
            forKey:(nullable NSString *)key
        completion:(nullable SDWebImageNoParamsBlock)completionBlock;
//缓存图片到磁盘
- (void)storeImage:(nullable UIImage *)image
            forKey:(nullable NSString *)key
            toDisk:(BOOL)toDisk
        completion:(nullable SDWebImageNoParamsBlock)completionBlock;
- (void)storeImage:(nullable UIImage *)image
         imageData:(nullable NSData *)imageData
            forKey:(nullable NSString *)key
            toDisk:(BOOL)toDisk
        completion:(nullable SDWebImageNoParamsBlock)completionBlock;
- (void)storeImageDataToDisk:(nullable NSData *)imageData forKey:(nullable NSString *)key;
//异步检查磁盘缓存是否存在
- (void)diskImageExistsWithKey:(nullable NSString *)key completion:(nullable SDWebImageCheckCacheCompletionBlock)completionBlock;
//检查内存缓存是否存在
- (nullable UIImage *)imageFromMemoryCacheForKey:(nullable NSString *)key;
//获取磁盘缓存数据
- (nullable UIImage *)imageFromDiskCacheForKey:(nullable NSString *)key;
//获取内存和磁盘缓存数据
- (nullable UIImage *)imageFromCacheForKey:(nullable NSString *)key;
//异步删除缓存
- (void)removeImageForKey:(nullable NSString *)key withCompletion:(nullable SDWebImageNoParamsBlock)completion;
- (void)removeImageForKey:(nullable NSString *)key fromDisk:(BOOL)fromDisk withCompletion:(nullable SDWebImageNoParamsBlock)completion;
//清除所有内存缓存
- (void)clearMemory;
//删除所有过期数据
- (void)deleteOldFilesWithCompletionBlock:(nullable SDWebImageNoParamsBlock)completionBlock;
//获取磁盘缓存大小
- (NSUInteger)getSize;
//获取磁盘缓存图片数
- (NSUInteger)getDiskCount;

```

SDImageCacheConfig用来对缓存进行配置，如下：

```objectivec
//是否允许缓存到内存
@property (assign, nonatomic) BOOL shouldCacheImagesInMemory;
//缓存生命
@property (assign, nonatomic) NSInteger maxCacheAge;
//最大缓存容量
@property (assign, nonatomic) NSUInteger maxCacheSize;
```

#### 2.下载器SDWebImageDownloader

    SDWebImageDownloader提供对图片下载的支持管理，其可以配置同时最大下载数量，下载超时等：

```objectivec
//同时最大下载数量
@property (assign, nonatomic) NSInteger maxConcurrentDownloads;
//当前正在下载的任务数量
@property (readonly, nonatomic) NSUInteger currentDownloadCount;
//设置超时时间
@property (assign, nonatomic) NSTimeInterval downloadTimeout;
//证书
@property (strong, nonatomic, nullable) NSURLCredential *urlCredential;
//用户名
@property (strong, nonatomic, nullable) NSString *username;
//密码
@property (strong, nonatomic, nullable) NSString *password;
//设置请求头
- (void)setValue:(nullable NSString *)value forHTTPHeaderField:(nullable NSString *)field;
//获取请求头
- (nullable NSString *)valueForHTTPHeaderField:(nullable NSString *)field;
//开始下载任务
- (nullable SDWebImageDownloadToken *)downloadImageWithURL:(nullable NSURL *)url
                                                   options:(SDWebImageDownloaderOptions)options
                                                  progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                                                 completed:(nullable SDWebImageDownloaderCompletedBlock)completedBlock;
//取消下载任务
- (void)cancel:(nullable SDWebImageDownloadToken *)token;
//取消所有下载任务
- (void)cancelAllDownloads;
```

### 三、应用类别

#### 1.UIButton+WebCache

    这个类别用来对按钮设置网络图片。

```objectivec
//当前状态的图片URL
- (nullable NSURL *)sd_currentImageURL;
//获取指定状态的图片URL
- (nullable NSURL *)sd_imageURLForState:(UIControlState)state;
//为某个状态设置网络图片
- (void)sd_setImageWithURL:(nullable NSURL *)url
                  forState:(UIControlState)state;
- (void)sd_setImageWithURL:(nullable NSURL *)url
                  forState:(UIControlState)state
          placeholderImage:(nullable UIImage *)placeholder;
- (void)sd_setImageWithURL:(nullable NSURL *)url
                  forState:(UIControlState)state
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options;
- (void)sd_setImageWithURL:(nullable NSURL *)url
                  forState:(UIControlState)state
                 completed:(nullable SDExternalCompletionBlock)completedBlock;
- (void)sd_setImageWithURL:(nullable NSURL *)url
                  forState:(UIControlState)state
          placeholderImage:(nullable UIImage *)placeholder
                 completed:(nullable SDExternalCompletionBlock)completedBlock;
- (void)sd_setImageWithURL:(nullable NSURL *)url
                  forState:(UIControlState)state
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                 completed:(nullable SDExternalCompletionBlock)completedBlock;
//下面这些方法设置按钮的背景图
- (nullable NSURL *)sd_currentBackgroundImageURL;
- (nullable NSURL *)sd_backgroundImageURLForState:(UIControlState)state;
- (void)sd_setBackgroundImageWithURL:(nullable NSURL *)url
                            forState:(UIControlState)state;
- (void)sd_setBackgroundImageWithURL:(nullable NSURL *)url
                            forState:(UIControlState)state
                    placeholderImage:(nullable UIImage *)placeholder;
- (void)sd_setBackgroundImageWithURL:(nullable NSURL *)url
                            forState:(UIControlState)state
                    placeholderImage:(nullable UIImage *)placeholder
                             options:(SDWebImageOptions)options;
- (void)sd_setBackgroundImageWithURL:(nullable NSURL *)url
                            forState:(UIControlState)state
                           completed:(nullable SDExternalCompletionBlock)completedBlock;
- (void)sd_setBackgroundImageWithURL:(nullable NSURL *)url
                            forState:(UIControlState)state
                    placeholderImage:(nullable UIImage *)placeholder
                           completed:(nullable SDExternalCompletionBlock)completedBlock;
- (void)sd_setBackgroundImageWithURL:(nullable NSURL *)url
                            forState:(UIControlState)state
                    placeholderImage:(nullable UIImage *)placeholder
                             options:(SDWebImageOptions)options
                           completed:(nullable SDExternalCompletionBlock)completedBlock;
//取消图片下载
- (void)sd_cancelImageLoadForState:(UIControlState)state;
- (void)sd_cancelBackgroundImageLoadForState:(UIControlState)state;
```

#### 2.UIImageView+WebCache与UIImageView+HighlightedWebCache

    这两个类别的作用都是对UIImageView实例进行图片设置，分别设置正常状态的图片和高亮状态的图片。只举例UIImageView+WebCache中方法如下：

```objectivec
//设置网络图片
- (void)sd_setImageWithURL:(nullable NSURL *)url;
- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder;
- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options;
- (void)sd_setImageWithURL:(nullable NSURL *)url
                 completed:(nullable SDExternalCompletionBlock)completedBlock;
- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                 completed:(nullable SDExternalCompletionBlock)completedBlock;
- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                 completed:(nullable SDExternalCompletionBlock)completedBlock;
- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                  progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                 completed:(nullable SDExternalCompletionBlock)completedBlock;
- (void)sd_setImageWithPreviousCachedImageWithURL:(nullable NSURL *)url
                                 placeholderImage:(nullable UIImage *)placeholder
                                          options:(SDWebImageOptions)options
                                         progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                                        completed:(nullable SDExternalCompletionBlock)completedBlock;
//对一组url进行下载并以动画方式显示
- (void)sd_setAnimationImagesWithURLs:(nonnull NSArray<NSURL *> *)arrayOfURLs;
//取消当前图片下载
- (void)sd_cancelCurrentAnimationImagesLoad;

```

关于SDWebImageOptions，它是一个配置枚举：

```objectivec
typedef NS_OPTIONS(NSUInteger, SDWebImageOptions) {
    //配置这个参数下载失败的url将会重试下载
    SDWebImageRetryFailed = 1 << 0,
    //低优先级
    SDWebImageLowPriority = 1 << 1,
    //仅仅进行内存缓存
    SDWebImageCacheMemoryOnly = 1 << 2,
    //进行分步下载
    SDWebImageProgressiveDownload = 1 << 3,
    //刷新缓存
    SDWebImageRefreshCached = 1 << 4,
    //支持后台
    SDWebImageContinueInBackground = 1 << 5,
    //保持cookie
    SDWebImageHandleCookies = 1 << 6,
    //允许证书
    SDWebImageAllowInvalidSSLCertificates = 1 << 7,
    //高优先级 
    SDWebImageHighPriority = 1 << 8,
    //配置此参数 当图片加载结束后才会显示placeholder
    SDWebImageDelayPlaceholder = 1 << 9,
    //执行图片变换
    SDWebImageTransformAnimatedImage = 1 << 10,
    SDWebImageScaleDownLargeImages = 1 << 12
};
```

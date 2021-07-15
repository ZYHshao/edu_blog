---
title: iOS开发之AssetsLibrary框架使用
date: 2018-09-11
categories: iOS逻辑初窥
tags: []
---
## iOS开发之AssetsLibrary框架使用

### 一、引言

    AssetsLibrary框架是专门用来操作相册相关资源的一个框架，其是iOS4到iOS9之间常使用的一个框架，在iOS9之后，系统系统了Photos框架代替了AssetsLibrary框架，但是AssetsLibrary框架依然可以使用，并且其结构和设计思路依然值得我们进行分析学习。

### 二、概述

    AssetsLibrary框架会操作系统的相册，因此首先需要进行权限的申请，在使用之前，首先需要在Info.plist文件中添加如下键值：

Privacy - Photo Library Usage Description

AssetsLibrary框架中核心的类关系如下图所示：

![](https://oscimg.oschina.net/oscnet/d6a95f1c782f0c09265fda33372c65ee032.jpg)

### 三、ALAssetsLibrary资源库对象

    ALAssetsLibrary类用来构建资源库对象，这个对象用来整体操作系统的相册资源，在使用它之前我们可以使用下面的方法来获取用户的授权情况：

```objectivec
+ (ALAuthorizationStatus)authorizationStatus;
```

ALAuthorizationStatus枚举定义了用户的授权情况，定义如下：

```objectivec
typedef NS_ENUM(NSInteger, ALAuthorizationStatus) {
    ALAuthorizationStatusNotDetermined, // 用户尚未选择是否授权
    ALAuthorizationStatusRestricted,    //应用尚未授权
    ALAuthorizationStatusDenied),       // 用户拒绝授权
    ALAuthorizationStatusAuthorized     // 用户已经授权
}
```

如果用户尚未授权过，那么任何访问操作都将触发授权机制。

    资源库中的资源数据是以组的方式进行存储，下面代码示例了获取资源组的方式：

```objectivec
    _library = [[ALAssetsLibrary alloc]init];
    [_library enumerateGroupsWithTypes:ALAssetsGroupAll usingBlock:^(ALAssetsGroup *group, BOOL *stop) {
        if (group) { // 遍历相册还未结束
            // 设置过滤器
            [group setAssetsFilter:[ALAssetsFilter allPhotos]];
            if (group.numberOfAssets) {
                NSLog(@"%@",group);
            }
        } else { // 遍历结束（当group为空的时候就意味着结束）
            
                NSLog(@"没有相册列表了");
            
        }
        
    } failureBlock:^(NSError *error) {
        NSLog(@"失败");
    }];
```

上面示例的枚举函数用来根据参数类型获取资源组，ALAssetsGroupType参数决定获取组的类型，可选值枚举如下：

```objectivec
enum {
    ALAssetsGroupLibrary     ,// 编辑库
    ALAssetsGroupAlbum       ,//相册库
    ALAssetsGroupEvent       ,//事件库  
    ALAssetsGroupFaces       ,// iTunes同步
    ALAssetsGroupSavedPhotos ,// 保存的相片
    ALAssetsGroupPhotoStream ,// The PhotoStream album.
    ALAssetsGroupAll         ,//所有库
};
```

枚举过程中，我们可以过去到ALAssetsGroup类型的对象，这个对象中封装了相片资源信息，后面会介绍。

    下面列举了ALAssetsLibrary中其他常用的方法：

```objectivec
//直接通过URL来获取资源
- (void)assetForURL:(NSURL *)assetURL resultBlock:(ALAssetsLibraryAssetForURLResultBlock)resultBlock failureBlock:(ALAssetsLibraryAccessFailureBlock)failureBlock;
//直接通过URL来获取资源组
- (void)groupForURL:(NSURL *)groupURL resultBlock:(ALAssetsLibraryGroupResultBlock)resultBlock failureBlock:(ALAssetsLibraryAccessFailureBlock)failureBlock;
//向相册库中添加一个新的资源组 可以自定义名称
- (void)addAssetsGroupAlbumWithName:(NSString *)name resultBlock:(ALAssetsLibraryGroupResultBlock)resultBlock failureBlock:(ALAssetsLibraryAccessFailureBlock)failureBlock;
//向相册中写入一张图片 orientation参数设置图片的方向
/*
typedef NS_ENUM(NSInteger, ALAssetOrientation) {
    ALAssetOrientationUp ,            // 向上 默认的
    ALAssetOrientationDown ,          // 向下
    ALAssetOrientationLeft ,          // 向左
    ALAssetOrientationRight ,         // 向右
    ALAssetOrientationUpMirrored ,    //
    ALAssetOrientationDownMirrored ,  // horizontal flip
    ALAssetOrientationLeftMirrored ,  // vertical flip
    ALAssetOrientationRightMirrored , // vertical flip
};
*/
- (void)writeImageToSavedPhotosAlbum:(CGImageRef)imageRef orientation:(ALAssetOrientation)orientation completionBlock:(ALAssetsLibraryWriteImageCompletionBlock)completionBlock;
//向相册中写入一张图片 并可以设置图片的元数据
- (void)writeImageToSavedPhotosAlbum:(CGImageRef)imageRef metadata:(NSDictionary *)metadata completionBlock:(ALAssetsLibraryWriteImageCompletionBlock)completionBlock;
//向相册中写入图片数据 并可以设置元数据
- (void)writeImageDataToSavedPhotosAlbum:(NSData *)imageData metadata:(NSDictionary *)metadata completionBlock:(ALAssetsLibraryWriteImageCompletionBlock)completionBlock;
//将某个路径的视频写入相册中
- (void)writeVideoAtPathToSavedPhotosAlbum:(NSURL *)videoPathURL completionBlock:(ALAssetsLibraryWriteVideoCompletionBlock)completionBlock;
//检查路径中的视频是否和相册相兼容
- (BOOL)videoAtPathIsCompatibleWithSavedPhotosAlbum:(NSURL *)videoPathURL;
```

当资源库改变时，系统会发出如下通知：

```objectivec
//资源库改变的通知
extern NSString *const ALAssetsLibraryChangedNotification;
```

通知中传递的信息中包含如下字段：

```objectivec
//资源库更新
extern NSString *const ALAssetLibraryUpdatedAssetsKey;
//插入组
extern NSString *const ALAssetLibraryInsertedAssetGroupsKey;
//更新组
extern NSString *const ALAssetLibraryUpdatedAssetGroupsKey;
//删除组
extern NSString *const ALAssetLibraryDeletedAssetGroupsKey;
```

下面列举了操作过程中的一些异常定义：

```objectivec
enum {
    ALAssetsLibraryUnknownError =                 -1,      // 未知错误
    ALAssetsLibraryWriteFailedError =           -3300,      //写入错误
    ALAssetsLibraryWriteBusyError =             -3301,      // 写入繁忙 可以重试
    ALAssetsLibraryWriteInvalidDataError =      -3302,      // 无效数据
    ALAssetsLibraryWriteIncompatibleDataError = -3303,      // 不兼容的数据
    ALAssetsLibraryWriteDataEncodingError =     -3304,      // 数据编码错误
    ALAssetsLibraryWriteDiskSpaceError =        -3305,      // 内存不足
    ALAssetsLibraryDataUnavailableError =       -3310,      // 数据不可用
    ALAssetsLibraryAccessUserDeniedError =      -3311,      // 权限错误
    ALAssetsLibraryAccessGloballyDeniedError =  -3312,      // 权限错误
};
```

### 四、ALAssetsGroup资源组对象

    资源组其实就是对应与我们相册中的一组资源，我们可以通过如下的方便遍历出其中的所有资源：

```objectivec
    _library = [[ALAssetsLibrary alloc]init];
    [_library enumerateGroupsWithTypes:ALAssetsGroupAll usingBlock:^(ALAssetsGroup *group, BOOL *stop) {
        if (group) { // 遍历相册还未结束
            // 设置过滤器
            [group setAssetsFilter:[ALAssetsFilter allPhotos]];
            if (group.numberOfAssets) {
                [group enumerateAssetsUsingBlock:^(ALAsset *result, NSUInteger index, BOOL *stop) {
                    NSLog(@"%d:%@",index,result);
                }];
            }
        } else { // 遍历结束（当group为空的时候就意味着结束）

                NSLog(@"没有相册列表了");
        }

    } failureBlock:^(NSError *error) {
        NSLog(@"失败");
    }];
```

    ALAssetsGroup中相关方法解析如下：

```objectivec
//获取相关属性
/*
extern NSString *const ALAssetsGroupPropertyName;//组名字
extern NSString *const ALAssetsGroupPropertyType;//组类型
extern NSString *const ALAssetsGroupPropertyPersistentID; //ID
extern NSString *const ALAssetsGroupPropertyURL;//组URL
*/
- (id)valueForProperty:(NSString *)property;
//获取当前组的缩略图海报
- (CGImageRef)posterImage;
//设置过滤器
- (void)setAssetsFilter:(ALAssetsFilter *)filter;
//获取组中资源个数
- (NSInteger)numberOfAssets;
//进行资源枚举
- (void)enumerateAssetsUsingBlock:(ALAssetsGroupEnumerationResultsBlock)enumerationBlock;
/*
typedef NS_OPTIONS(NSUInteger, NSEnumerationOptions) {
    NSEnumerationConcurrent = (1UL << 0),//顺序枚举
    NSEnumerationReverse = (1UL << 1),   //逆序枚举
};
*/
- (void)enumerateAssetsWithOptions:(NSEnumerationOptions)options usingBlock:(ALAssetsGroupEnumerationResultsBlock)enumerationBlock;
- (void)enumerateAssetsAtIndexes:(NSIndexSet *)indexSet options:(NSEnumerationOptions)options usingBlock:(ALAssetsGroupEnumerationResultsBlock)enumerationBlock;
//获取当前组是否允许编辑
@property (nonatomic, readonly, getter=isEditable) BOOL editable;
//向组中添加一个资源
- (BOOL)addAsset:(ALAsset *)asset;
```

上面有提到资源过滤器，资源过滤器用来设置过滤组中的资源，有3个类方法可以直接获取系统提供的过滤器：

```objectivec
@interface ALAssetsFilter : NSObject {
//所有图片资源
+ (ALAssetsFilter *)allPhotos;
// 所有视频资源
+ (ALAssetsFilter *)allVideos;
// 所有资源
+ (ALAssetsFilter *)allAssets;
@end
```

### 五、ALAsset资源对象

    ALAsset是封装好的资源对象类，如下方法可以获取到资源中封装的属性：

```objectivec
- (id)valueForProperty:(NSString *)property;
```

属性名的定义如下：

```objectivec
//获取资源类型
/*
这个属性将返回一个字符串
extern NSString *const ALAssetTypePhoto//照片类型
extern NSString *const ALAssetTypeVideo//视频类型
extern NSString *const ALAssetTypeUnknown//未知类型
*/
extern NSString *const ALAssetPropertyType;
//会返回一个CLLocation对象 图片的地址信息
extern NSString *const ALAssetPropertyLocation;
//视频资源的时长 NSNumber对象
extern NSString *const ALAssetPropertyDuration;
//资源方向
extern NSString *const ALAssetPropertyOrientation;
//资源日期 会返回NSDate对象
extern NSString *const ALAssetPropertyDate;
```

下面列举了ALAsset中常用方法：

```objectivec
//获取默认的Representation对象
- (ALAssetRepresentation *)defaultRepresentation;
//获取指定的Representation对象
- (ALAssetRepresentation *)representationForUTI:(NSString *)representationUTI;
//获取资源缩略图
- (CGImageRef)thumbnail;
- (CGImageRef)aspectRatioThumbnail;
//写入图片数据
- (void)writeModifiedImageDataToSavedPhotosAlbum:(NSData *)imageData metadata:(NSDictionary *)metadata completionBlock:(ALAssetsLibraryWriteImageCompletionBlock)completionBlock;
//写入视频数据
- (void)writeModifiedVideoAtPathToSavedPhotosAlbum:(NSURL *)videoPathURL completionBlock:(ALAssetsLibraryWriteVideoCompletionBlock)completionBlock;
//原始资源对象
@property (nonatomic, readonly) ALAsset *originalAsset;
//是否允许编辑
@property (nonatomic, readonly, getter=isEditable) BOOL editable;
//替换图片数据
- (void)setImageData:(NSData *)imageData metadata:(NSDictionary *)metadata completionBlock:(ALAssetsLibraryWriteImageCompletionBlock)completionBlock;
//替换视频数据
- (void)setVideoAtPath:(NSURL *)videoPathURL completionBlock:(ALAssetsLibraryWriteVideoCompletionBlock)completionBlock;
```

### 六、关于ALAssetRepresentation类

    每一个ALAsset对象中都封装了一个ALAssetRepresentation对象，这个对象的作用是获取资源的详细信息，解析如下：

```objectivec
//获取UTI
- (NSString *)UTI;
//获取资源的尺寸
- (CGSize)dimensions;
//获取资源的大小
- (long long)size;
//读取数据
- (NSUInteger)getBytes:(uint8_t *)buffer fromOffset:(long long)offset length:(NSUInteger)length error:(NSError **)error;
//获取图片数据
- (CGImageRef)fullResolutionImage;
- (CGImageRef)CGImageWithOptions:(NSDictionary *)options;
//获取全屏图片
- (CGImageRef)fullScreenImage;
//获取资源URL
- (NSURL *)url;
//获取资源元数据
- (NSDictionary *)metadata;
//获取资源方向
- (ALAssetOrientation)orientation;
//缩放比
- (float)scale;
//获取资源名称
- (NSString *)filename;
```

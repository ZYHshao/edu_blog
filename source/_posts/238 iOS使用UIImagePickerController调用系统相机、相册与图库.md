---
title: iOS使用UIImagePickerController调用系统相机、相册与图库
date: 2016-07-12
categories: iOS之UI控件
tags: []
---
## iOS使用UIImagePickerController调用系统相机、相册与图库

### 一、引言

        UIImagePickerController是系统封装好的一个导航视图控制器，使用其开发者可以十分方便的进行相机相册相关功能的调用。UIImagePickerController继承于UINavigationController，其通过代理的方式将用户获取的图片或者视频文件传入给开发者。

### 二、UIImagePickerController中属性与方法的应用

        在使用UIImagePickerController之前，应该先判断设备做支持的媒体文件获取类型，使用如下方法进行判断：

```objectivec
//判断是否支持某个数据提供类型
/*
UIImagePickerControllerSourceType枚举定义如下:
typedef NS_ENUM(NSInteger, UIImagePickerControllerSourceType) {
    //系统图库
    UIImagePickerControllerSourceTypePhotoLibrary,
    //相机
    UIImagePickerControllerSourceTypeCamera,
    //系统相册
    UIImagePickerControllerSourceTypeSavedPhotosAlbum
} __TVOS_PROHIBITED;
*/
+ (BOOL)isSourceTypeAvailable:(UIImagePickerControllerSourceType)sourceType;

//判断某个数据提供者所支持的文件格式
/*
文件格式定义在<MobileCoreServices/MobileCoreServices.h>框架中
*/
+ (nullable NSArray<NSString *> *)availableMediaTypesForSourceType:(UIImagePickerControllerSourceType)sourceType; 

//判断所支持的相机设备
/*
typedef NS_ENUM(NSInteger, UIImagePickerControllerCameraDevice) {
    //前置摄像头
    UIImagePickerControllerCameraDeviceRear,
    //后置摄像头
    UIImagePickerControllerCameraDeviceFront
} __TVOS_PROHIBITED;
*/
+ (BOOL)isCameraDeviceAvailable:(UIImagePickerControllerCameraDevice)cameraDevice                   NS_AVAILABLE_IOS(4_0); 

//判断对闪光灯的支持
+ (BOOL)isFlashAvailableForCameraDevice:(UIImagePickerControllerCameraDevice)cameraDevice           NS_AVAILABLE_IOS(4_0);

//判断相机设备支持的媒体模式
/*
返回值为如下枚举：
typedef NS_ENUM(NSInteger, UIImagePickerControllerCameraCaptureMode) {
    //照片模式
    UIImagePickerControllerCameraCaptureModePhoto,
    //视频模式
    UIImagePickerControllerCameraCaptureModeVideo
} __TVOS_PROHIBITED;
*/
+ (nullable NSArray<NSNumber *> *)availableCaptureModesForCameraDevice:(UIImagePickerControllerCameraDevice)cameraDevice NS_AVAILABLE_IOS(4_0); 
```

上面提到的定义于<MobileCoreServices/MobileCoreServices.h>框架中的文件类型，列举如下：

```objectivec
//图片类型
extern const CFStringRef kUTTypeImage                                __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
//JPEG格式
extern const CFStringRef kUTTypeJPEG                                 __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
//JPEG2000格式
extern const CFStringRef kUTTypeJPEG2000                             __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeTIFF                                 __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypePICT                                 __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeGIF                                  __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypePNG                                  __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeQuickTimeImage                       __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeAppleICNS                            __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeBMP                                  __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeICO                                  __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeRawImage                             __OSX_AVAILABLE_STARTING(__MAC_10_10,__IPHONE_8_0);
extern const CFStringRef kUTTypeScalableVectorGraphics               __OSX_AVAILABLE_STARTING(__MAC_10_10,__IPHONE_8_0);
extern const CFStringRef kUTTypeLivePhoto                            __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_9_1);



//视频格式
extern const CFStringRef kUTTypeAudiovisualContent                   __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeMovie                                __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeVideo                                __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeAudio                                __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeQuickTimeMovie                       __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeMPEG                                 __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeMPEG2Video                           __OSX_AVAILABLE_STARTING(__MAC_10_10,__IPHONE_8_0);
extern const CFStringRef kUTTypeMPEG2TransportStream                 __OSX_AVAILABLE_STARTING(__MAC_10_10,__IPHONE_8_0);
extern const CFStringRef kUTTypeMP3                                  __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeMPEG4                                __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeMPEG4Audio                           __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeAppleProtectedMPEG4Audio             __OSX_AVAILABLE_STARTING(__MAC_10_4,__IPHONE_3_0);
extern const CFStringRef kUTTypeAppleProtectedMPEG4Video             __OSX_AVAILABLE_STARTING(__MAC_10_10,__IPHONE_8_0);
extern const CFStringRef kUTTypeAVIMovie                             __OSX_AVAILABLE_STARTING(__MAC_10_10,__IPHONE_8_0);
extern const CFStringRef kUTTypeAudioInterchangeFileFormat           __OSX_AVAILABLE_STARTING(__MAC_10_10,__IPHONE_8_0);
extern const CFStringRef kUTTypeWaveformAudio                        __OSX_AVAILABLE_STARTING(__MAC_10_10,__IPHONE_8_0);
extern const CFStringRef kUTTypeMIDIAudio                            __OSX_AVAILABLE_STARTING(__MAC_10_10,__IPHONE_8_0);
```

CFStringRef与NSString类型的转换，可以使用如下方法：

```objectivec
NSString * str = (__bridge NSString*)kUTTypeMovie;
```

UIImagePickerController中更多属性与方法解析如下：

```objectivec
//设置代理
@property(nullable,nonatomic,weak)      id <UINavigationControllerDelegate, UIImagePickerControllerDelegate> delegate;
//设置书体提供者类型 默认为图库
@property(nonatomic)           UIImagePickerControllerSourceType     sourceType; 
//设置所需要的数据类型，需要设置为系统定义的文件类型字符串数组 默认为kUTTypeImage
@property(nonatomic,copy)      NSArray<NSString *>                   *mediaTypes;
//设置是否允许编辑图片 设置为YES，则用户选择图片时可以编辑裁剪图片
@property(nonatomic)           BOOL                                  allowsEditing;
//设置媒体文件的最大时长 默认为10分钟
@property(nonatomic)           NSTimeInterval                        videoMaximumDuration; 
//设置媒体文件的质量 枚举如下：
/*
typedef NS_ENUM(NSInteger, UIImagePickerControllerQualityType) {
    UIImagePickerControllerQualityTypeHigh = 0,       // 高质量
    UIImagePickerControllerQualityTypeMedium = 1,     // 中等质量
    UIImagePickerControllerQualityTypeLow = 2,         // 低质量
    UIImagePickerControllerQualityType640x480 NS_ENUM_AVAILABLE_IOS(4_0) = 3,    
    UIImagePickerControllerQualityTypeIFrame1280x720 NS_ENUM_AVAILABLE_IOS(5_0) = 4,
    UIImagePickerControllerQualityTypeIFrame960x540 NS_ENUM_AVAILABLE_IOS(5_0) = 5,
} __TVOS_PROHIBITED;
*/
@property(nonatomic)           UIImagePickerControllerQualityType    videoQuality;
//设置是否显示相机控制界面
@property(nonatomic)           BOOL                                  showsCameraControls;
//自定义的拍照界面 其会覆盖在原拍照界面上
@property(nullable, nonatomic,strong) __kindof UIView                *cameraOverlayView  NS_AVAILABLE_IOS(3_1);  
//设置拍照界面的transform
@property(nonatomic)           CGAffineTransform                     cameraViewTransform ;
//拍照
- (void)takePicture NS_AVAILABLE_IOS(3_1);  
//进行视频捕获
- (BOOL)startVideoCapture NS_AVAILABLE_IOS(4_0);
//停止视频捕获
- (void)stopVideoCapture  NS_AVAILABLE_IOS(4_0);
//设置相机捕获模式 照片或视频
@property(nonatomic) UIImagePickerControllerCameraCaptureMode cameraCaptureMode;
//设置相机设备 前置或后置摄像头
@property(nonatomic) UIImagePickerControllerCameraDevice      cameraDevice;
//设置闪光灯模式
/*
typedef NS_ENUM(NSInteger, UIImagePickerControllerCameraFlashMode) {
    UIImagePickerControllerCameraFlashModeOff  = -1, //关闭
    UIImagePickerControllerCameraFlashModeAuto = 0,  //自动
    UIImagePickerControllerCameraFlashModeOn   = 1   //开启
} __TVOS_PROHIBITED;
*/
@property(nonatomic) UIImagePickerControllerCameraFlashMode   cameraFlashMode;

```

### 三、UIImagePickerControllerDelegate中方法解析

```objectivec
//相机拍照完成或者从图库相册选择相片完成后触发的回调方法 editingInfo字典中将传入编辑信息
- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingImage:(UIImage *)image editingInfo:(nullable NSDictionary<NSString *,id> *)editingInfo NS_DEPRECATED_IOS(2_0, 3_0);
//相机录像或者从图库相册选择视频完成后触发的回调方法 info字典中是具体信息
- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary<NSString *,id> *)info;
//ImagePickerController取消选择是回调的方法
- (void)imagePickerControllerDidCancel:(UIImagePickerController *)picker;
```

上面方法中的编辑字典与信息字典中，约定好了一些键值对，开发者可以通过相应的键获取需要的信息，规则如下：

```objectivec
//这个键对应NSString类型的值 意义为媒体文件的格式
UIKIT_EXTERN NSString *const UIImagePickerControllerMediaType;
//这个键对应UIImage类型的值 意义为获取的原始图片
UIKIT_EXTERN NSString *const UIImagePickerControllerOriginalImage;
//这个件对应UIIImage类型的值 意义为获取编辑后的图片
UIKIT_EXTERN NSString *const UIImagePickerControllerEditedImage;
//这个键对应一个NSValue值 可以转为CGRect类型 意义为编辑的图片范围
UIKIT_EXTERN NSString *const UIImagePickerControllerCropRect;
//这个键对应媒体文件的URL
UIKIT_EXTERN NSString *const UIImagePickerControllerMediaURL;
//这个键对应图库中的URL
UIKIT_EXTERN NSString *const UIImagePickerControllerReferenceURL;
//这个键对应一个NSDictionary 里面存放媒体数据
UIKIT_EXTERN NSString *const UIImagePickerControllerMediaMetadata;
//现场图片数据 相机捕捉图片时会记录声音
UIKIT_EXTERN NSString *const UIImagePickerControllerLivePhoto;

```

### 四、对捕获的图片与视频进行持久化

        系统也提供了对相机照片和视频进行存储的方式，列举如下：

```objectivec
//将图片数据存储到相册
void UIImageWriteToSavedPhotosAlbum(UIImage *image, __nullable id completionTarget, __nullable SEL completionSelector, void * __nullable contextInfo);
//将视频保存到相册
BOOL UIVideoAtPathIsCompatibleWithSavedPhotosAlbum(NSString *videoPath);
void UISaveVideoAtPathToSavedPhotosAlbum(NSString *videoPath, __nullable id completionTarget, __nullable SEL completionSelector, void * __nullable contextInfo);
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

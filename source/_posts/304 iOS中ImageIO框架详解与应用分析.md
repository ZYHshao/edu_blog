---
title: iOS中ImageIO框架详解与应用分析
date: 2017-02-14
categories: iOS逻辑初窥
tags: []
---
## iOS中ImageIO框架详解与应用分析

### 一、引言

    ImageIO框架提供了读取与写入图片数据的基本方法，使用它可以直接获取到图片文件的内容数据，ImageIO框架中包含6个头文件，其中完成主要功能的是前两个头文件中定义的方法：

1.CGImageSource.h:负责读取图片数据。

2.CGImageDestination.h:负责写入图片数据。

3.CGImageMetadata.h:图片文件元数据类。

4.CGImageProperties:定义了框架中使用的字符串常量和宏。

5.ImageIOBase.h:预处理逻辑，无需关心。

### 二、CGImageSource详解

    CGImageSource类的主要作用是用来读取图片数据，在平时开发中，关于图片我们使用的最多的可能是UIImage类，UIImage是iOS系统UI系统中用于构建图像对象的类，但是其中只有图像数据，实际上一个图片文件中存储的除了图片数据外，还有一些地理位置、设备类型、时间等信息，除此之外，一个图片文件中可能存储的也不只一张图像(例如gif文件)。CGImageSource就是这样的一个抽象图片数据示例，从其中可以获取到我们所关心的所有数据。

    读取图片文件数据，并将其展示在视图的简单代码示例如下：

```objectivec
//获取图片文件路径
NSString * path = [[NSBundle mainBundle]pathForResource:@"timg" ofType:@"jpeg"];
NSURL * url = [NSURL fileURLWithPath:path];
CGImageRef myImage = NULL;
CGImageSourceRef myImageSource;
//通过文件路径创建CGImageSource对象
myImageSource = CGImageSourceCreateWithURL((CFURLRef)url, NULL);
//获取第一张图片
myImage = CGImageSourceCreateImageAtIndex(myImageSource,
                                          0,
                                          NULL);
CFRelease(myImageSource);
UIImageView * image = [[UIImageView alloc]initWithFrame:CGRectMake(0, 0, 200, 200)];
image.image = [UIImage imageWithCGImage:myImage];
[self.view addSubview:image];
```

上面的示例代码采用的是本地的一个素材文件，当然通过网络图片链接也是可以创建CGImageSource独享的。除了通过URL链接的方式创建对象，ImageIO框架中还提供了两种方法，解析如下：

```objectivec
//通过数据提供器创建CGImageSource对象
/*
CGDataProviderRef是CoreGraphics框架中的一个数据读取类，其也可以通过Data数据，URL和文件名来创建
*/
CGImageSourceRef __nullable CGImageSourceCreateWithDataProvider(CGDataProviderRef __nonnull provider, CFDictionaryRef __nullable options);
//通过Data数据创建CGImageSource对象
CGImageSourceRef __nullable CGImageSourceCreateWithData(CFDataRef __nonnull data, CFDictionaryRef __nullable options);
```

需要注意，上面所提到的所有创建CGImageSource的方法中都可以传入一个CFDictionaryRef类型的字典，可以配置的键值意义如下：

```objectivec
/*
设置一个预期的图片文件格式，需要设置为字符串类型的值
*/
const CFStringRef kCGImageSourceTypeIdentifierHint;
/*
设置是否以解码的方式读取图片数据 默认为kCFBooleanTrue
如果设置为true，在读取数据时就进行解码 如果为false 则在渲染时才进行解码
*/
const CFStringRef kCGImageSourceShouldCache;
/*
返回CGImage对象时是否允许使用浮点值 默认为kCFBooleanFalse
*/
const CFStringRef kCGImageSourceShouldAllowFloa;
/*
设置如果不存在缩略图则创建一个缩略图，缩略图的尺寸受开发者设置影响，如果不设置尺寸极限，则为图片本身大小
默认为kCFBooleanFalse
*/
const CFStringRef kCGImageSourceCreateThumbnailFromImageIfAbsent;
/*
设置是否创建缩略图，无论原图像有没有包含缩略图kCFBooleanFalse
*/
const CFStringRef kCGImageSourceCreateThumbnailFromImageAlways;
/*
设置缩略图的宽高尺寸 需要设置为CFNumber值
*/
const CFStringRef kCGImageSourceThumbnailMaxPixelSize;
/*
设置缩略图是否进行Transfrom变换
*/
const CFStringRef kCGImageSourceCreateThumbnailWithTransform;
```

CGImageSource类中其他方法解析如下：

```objectivec
//获取CGImageSource类在CoreFundation框架中的id
CFTypeID CGImageSourceGetTypeID (void);
//获取所支持的图片格式数组
CFArrayRef __nonnull CGImageSourceCopyTypeIdentifiers(void);
//获取CGImageSource对象的图片格式
CFStringRef __nullable CGImageSourceGetType(CGImageSourceRef __nonnull isrc);
//获取CGImageSource中的图片张数 不包括缩略图
size_t CGImageSourceGetCount(CGImageSourceRef __nonnull isrc);
//获取CGImageSource的文件信息
/*
字典参数可配置的键值对与创建CGImageSource所传参数意义一致
返回的字典中的键值意义后面介绍
*/
CFDictionaryRef __nullable CGImageSourceCopyProperties(CGImageSourceRef __nonnull isrc, CFDictionaryRef __nullable options);
//获取CGImageSource中某个图像的附加数据
/*
index参数设置获取第几张图像 options参数可配置的键值对与创建CGImageSource所传参数意义一致
返回的字典中的键值意义后面介绍
*/
CFDictionaryRef __nullable CGImageSourceCopyPropertiesAtIndex(CGImageSourceRef __nonnull isrc, size_t index, CFDictionaryRef __nullable options);
//获取图片的元数据信息 CGImageMetadataRef类是图像原数据的抽象
CGImageMetadataRef __nullable CGImageSourceCopyMetadataAtIndex (CGImageSourceRef __nonnull isrc, size_t index, CFDictionaryRef __nullable options);
//获取CGImageSource中的图片数据
CGImageRef __nullable CGImageSourceCreateImageAtIndex(CGImageSourceRef __nonnull isrc, size_t index, CFDictionaryRef __nullable options);
//删除一个指定索引图像的缓存
void CGImageSourceRemoveCacheAtIndex(CGImageSourceRef __nonnull isrc, size_t index);
//获取某一帧图片的缩略图
CGImageRef __nullable CGImageSourceCreateThumbnailAtIndex(CGImageSourceRef __nonnull isrc, size_t index, CFDictionaryRef __nullable options);
//创建一个空的CGImageSource容器，逐步加载大图片
CGImageSourceRef __nonnull CGImageSourceCreateIncremental(CFDictionaryRef __nullable options);
//使用新的数据更新CGImageSource容器
void CGImageSourceUpdateData(CGImageSourceRef __nonnull isrc, CFDataRef __nonnull data, bool final);
//更新数据提供器来填充CGImageSource容器
void CGImageSourceUpdateDataProvider(CGImageSourceRef __nonnull isrc, CGDataProviderRef __nonnull provider, bool final);
//获取当前CGImageSource的状态
/*
CGImageSourceStatus枚举意义：
typedef CF_ENUM(int32_t, CGImageSourceStatus) {
    kCGImageStatusUnexpectedEOF = -5, //文件结尾出错
    kCGImageStatusInvalidData = -4,   //数据无效
    kCGImageStatusUnknownType = -3,   //未知的图片类型
    kCGImageStatusReadingHeader = -2, //读标题过程中
    kCGImageStatusIncomplete = -1,    //操作不完整
    kCGImageStatusComplete = 0        //操作完整
};
*/
CGImageSourceStatus CGImageSourceGetStatus(CGImageSourceRef __nonnull isrc);
//同上，获取某一个图片的状态
CGImageSourceStatus CGImageSourceGetStatusAtIndex(CGImageSourceRef __nonnull isrc, size_t index);
```

### 三、CGImageDestination详解

    CGImageSource是图片文件数据的抽象对象，而CGImageDestination的作用则是将抽象的图片数据写入指定的目标中。将图片写成文件示例如下：

```objectivec
//创建存储路径
NSArray *paths=NSSearchPathForDirectoriesInDomains(NSDocumentDirectory,NSUserDomainMask,YES);
NSString *newPath = [paths.firstObject stringByAppendingPathComponent:[NSString stringWithFormat:@"image.png"]];
CFURLRef URL =  CFURLCreateWithFileSystemPath (
                                   kCFAllocatorDefault,
                                   (CFStringRef)newPath,
                                   kCFURLPOSIXPathStyle, 
                                   false);
//创建CGImageDestination对象
CGImageDestinationRef myImageDest = CGImageDestinationCreateWithURL(URL,CFSTR("public.png"), 1, NULL);
UIImage * image = [UIImage imageNamed:@"timg.jpeg"];
//写入图片
CGImageDestinationAddImage(myImageDest, image.CGImage, NULL);
CGImageDestinationFinalize(myImageDest);
CFRelease(myImageDest);
```

同样，除了可以直接将图片数据写入url外，也可以Data数据或数据消费器，方法如下：

```objectivec
//将图片数据写入数据消费者
CGImageDestinationRef __nullable CGImageDestinationCreateWithDataConsumer(CGDataConsumerRef __nonnull consumer, CFStringRef __nonnull type, size_t count, CFDictionaryRef __nullable options);
//将图片数据写入Data
CGImageDestinationRef __nullable CGImageDestinationCreateWithData(CFMutableDataRef __nonnull data, CFStringRef __nonnull type, size_t count, CFDictionaryRef __nullable options);
```

需要注意，上面方法的type参数设置写入数据的文件格式，必须为ImageIO框架所支持的格式，前面有方法可以获取所有支持的格式，还有一点，这3个写入方法的中options参数目前并没有什么作用，其是留给未来使用的，目前传入NULL即可。

CGImageDestination类中的其他方法解析如下：

```objectivec
//获取CGImageDestination的CFTypeID
CFTypeID CGImageDestinationGetTypeID(void);
//获取CGImageDestination所支持的图片文件类型
/*
目前支持如下：iOS10.1
 (
    "public.jpeg",
    "public.png",
    "com.compuserve.gif",
    "public.tiff",
    "public.jpeg-2000",
    "com.microsoft.ico",
    "com.microsoft.bmp",
    "com.adobe.photoshop-image",
    "com.adobe.pdf",
    "com.truevision.tga-image",
    "com.ilm.openexr-image",
    "public.pbm",
    "public.pvr",
    "org.khronos.astc",
    "org.khronos.ktx",
    "com.microsoft.dds",
    "com.apple.rjpeg"
)
*/
CFArrayRef __nonnull CGImageDestinationCopyTypeIdentifiers(void);
//设置图片文件属性
/*
可以设置的键值对意义如下：
const CFStringRef kCGImageDestinationLossyCompressionQuality; //设置压缩质量 0-1之间的cfnumberref值
const CFStringRef kCGImageDestinationBackgroundColor;  //将图片数据写为无alpha通道时的默认背景色 cgcolor值
*/
void CGImageDestinationSetProperties(CGImageDestinationRef __nonnull idst, CFDictionaryRef __nullable properties);
//向CGImageDestination中添加一张图片 其中的option参数意义和上面一致，设置此图片的质量与无alpha默认背景色
void CGImageDestinationAddImage(CGImageDestinationRef __nonnull idst, CGImageRef __nonnull image, CFDictionaryRef __nullable properties);
//通过CGImageSource对象来向CGImageDestination中添加图片
void CGImageDestinationAddImageFromSource(CGImageDestinationRef __nonnull idst, CGImageSourceRef __nonnull isrc, size_t index, CFDictionaryRef __nullable properties);
//进行写入操作 执行此方法后 不可以在写入其他信息
bool CGImageDestinationFinalize(CGImageDestinationRef __nonnull idst);
//添加图片元信息
void CGImageDestinationAddImageAndMetadata(CGImageDestinationRef __nonnull idst, CGImageRef __nonnull image, CGImageMetadataRef __nullable metadata, CFDictionaryRef __nullable options);
//将CGImageSource信息拷贝进CGImageDestination
/*
options参数可以用来添加元信息
*/
bool CGImageDestinationCopyImageSource(CGImageDestinationRef __nonnull idst, CGImageSourceRef __nonnull isrc, CFDictionaryRef __nullable options, __nullable CFErrorRef * __nullable err);
```

上面列举的方法中，CGImageDestinationCopyImageSource()方法中的options参数可以添加一些图片的元信息，可以设置的键值对意义如下：

```objectivec
//设置元信息 需要设置为CGImageMetadataRef对象
const CFStringRef kCGImageDestinationMetadata;
//是否将CGImageSource的元信息信息合并操作 默认为kCFBooleanFalse
const CFStringRef kCGImageDestinationMergeMetadata;
//XMP数据是否不被写入 默认为kCFBooleanFalse
const CFStringRef kCGImageMetadataShouldExcludeXMP;
//GPS信息是否不被写入 默认为kCFBooleanFalse
const CFStringRef kCGImageMetadataShouldExcludeGPS;
//更新元数据的时间值 需要设置为CFStringRef或者CFDateRef
const CFStringRef kCGImageDestinationDateTime;
//更新元数据的方向值 需要设置为NSNumber1-8
const CFStringRef kCGImageDestinationOrientation;
```

### 四、关于CGImageMetadata

    前面我们很多次提到元数据，CGImageMetadata类就是元数据的抽象，其中封装了一些方法供开发者读取或写入元数据信息。奇怪的是Apple的官方文档与API文档中并没有CGImageMetadata的介绍与解释，博客中本部分的内容，多出自我的理解，有疏漏和不对的地方，清楚的朋友可以指点与建议。

    前边介绍，CGImageSource中有获取图片元数据的方法，CGImageDestination中也有写入图片元数据的方法，元数据中抽象出的CGImageMetadataTag是对具体数据内容的封装。CGImageMetadata解析如下：

```objectivec
//获取CGImageMetadata类的CFTypeID
CFTypeID CGImageMetadataGetTypeID(void);
//创建一个空的可变的CGImageMetadata对象
CGMutableImageMetadataRef __nonnull CGImageMetadataCreateMutable(void);
//拷贝一个可变的CGImageMetadata对象
CGMutableImageMetadataRef __nullable CGImageMetadataCreateMutableCopy(CGImageMetadataRef __nonnull metadata);
//获取CGImageMetadataTag类的CFTypeID
CFTypeID CGImageMetadataTagGetTypeID(void);
//创建一个CGImageMetadataTag对象
/*
这个方法比较复杂
xmlns参数设置命名空间
prefix参数设置命名空间的缩写或前缀
name参数设置CGImageMetadataTag的名称
type参数设置CGImageMetadataTag对应值的类型
value参数设置CGImageMetadataTag的对应值
*/
CGImageMetadataTagRef __nullable CGImageMetadataTagCreate (CFStringRef __nonnull xmlns, CFStringRef __nullable prefix, CFStringRef __nonnull name, CGImageMetadataType type, CFTypeRef __nonnull value);
```

上面创建CGImageMetadataTag的方法中，xmlns设置命名空间，必须使用一个预定义的命名空间或者自定义的命名空间，对于自定义的命名空间，必须遵守Adobe的XMP规范。一些共用的命名空间定义如下：

```objectivec
//Exif命名空间
const CFStringRef  kCGImageMetadataNamespaceExif;
//ExifAux命名空间
const CFStringRef  kCGImageMetadataNamespaceExifAux;
//ExifEX命名空间
const CFStringRef  kCGImageMetadataNamespaceExifEX;
//DublineCore命名空间
const CFStringRef  kCGImageMetadataNamespaceDublinCore;
//IPTCCore命名空间
const CFStringRef  kCGImageMetadataNamespaceIPTCCore;
//Photoshop命名空间
const CFStringRef  kCGImageMetadataNamespacePhotoshop;
//TIFF命名空间
const CFStringRef  kCGImageMetadataNamespaceTIFF;
//XMPBasic命名空间
const CFStringRef  kCGImageMetadataNamespaceXMPBasic;
//XMPRights命名空间
const CFStringRef  kCGImageMetadataNamespaceXMPRights;
```

上面创建CGImageMetadataTag的方法中prefix设置命名空间缩写或前缀，同样一些公用的前缀定义如下：

```objectivec
//Exif命名空间前缀
const CFStringRef  kCGImageMetadataPrefixExif;
//ExifAux命名空间前缀
const CFStringRef  kCGImageMetadataPrefixExifAux;
//ExifEX命名空间前缀
const CFStringRef  kCGImageMetadataPrefixExifEX;
//DublinCore命名空间前缀
const CFStringRef  kCGImageMetadataPrefixDublinCore;
//IPCCore命名空间前缀
const CFStringRef  kCGImageMetadataPrefixIPTCCore;
//Photoshop命名空间前缀
const CFStringRef  kCGImageMetadataPrefixPhotoshop;
//TIFF命名空间前缀
const CFStringRef  kCGImageMetadataPrefixTIFF;
//XMPBasic命名空间前缀
const CFStringRef  kCGImageMetadataPrefixXMPBasic;
//XMPRights命名空间前缀
const CFStringRef  kCGImageMetadataPrefixXMPRights;
```

上面创建CGImageMetadataTag的方法中type设置对应值的类型，其是一个CGImageMetadataType类型的枚举，意义如下：

```objectivec
typedef CF_ENUM(int32_t, CGImageMetadataType) {
    //无效的数据类型
    kCGImageMetadataTypeInvalid = -1,
    //基本的CFType类型
    kCGImageMetadataTypeDefault = 0,
    //字符串类型
    kCGImageMetadataTypeString = 1,
    //无需集合类型
    kCGImageMetadataTypeArrayUnordered = 2,
    //有序集合类型
    kCGImageMetadataTypeArrayOrdered = 3,
    //有序阵列
    kCGImageMetadataTypeAlternateArray = 4,
    //特殊的数组 其中元素进行不同的本地化
    kCGImageMetadataTypeAlternateText = 5,
    //结构类型 如字典
    kCGImageMetadataTypeStructure = 6
};
```

获取到CGImageMetadataTag后，可以通过如下方法来获取其中封装的信息：

```objectivec
//获取标签的命名空间
CFStringRef __nullable CGImageMetadataTagCopyNamespace(CGImageMetadataTagRef __nonnull tag);
//获取标签的命名空间前缀
CFStringRef __nullable CGImageMetadataTagCopyPrefix(CGImageMetadataTagRef __nonnull tag);
//获取标签名称
CFStringRef __nullable CGImageMetadataTagCopyName(CGImageMetadataTagRef __nonnull tag);
//获取标签的值
CFTypeRef __nullable CGImageMetadataTagCopyValue(CGImageMetadataTagRef __nonnull tag);
//获取标签值的类型
CGImageMetadataType CGImageMetadataTagGetType(CGImageMetadataTagRef __nonnull tag);
//获取标签的Qualifier数组
CFArrayRef __nullable CGImageMetadataTagCopyQualifiers(CGImageMetadataTagRef __nonnull tag);
```

下面这些方法用于向CGImageMetadata中添加标签或者获取标签：

```objectivec
//获取CGImageMetadata中的所有标签
CFArrayRef __nullable CGImageMetadataCopyTags(CGImageMetadataRef __nonnull metadata);
//通过路径查找特殊的标签
CGImageMetadataTagRef __nullable CGImageMetadataCopyTagWithPath(CGImageMetadataRef __nonnull metadata, CGImageMetadataTagRef __nullable parent, CFStringRef __nonnull path);
//通过路径查找特殊标签的值
 CFStringRef __nullable CGImageMetadataCopyStringValueWithPath(CGImageMetadataRef __nonnull metadata, CGImageMetadataTagRef __nullable parent, CFStringRef __nonnull path);
//为一个前缀注册一个命名空间
bool CGImageMetadataRegisterNamespaceForPrefix(CGMutableImageMetadataRef __nonnull metadata, CFStringRef __nonnull xmlns, CFStringRef __nonnull prefix, __nullable CFErrorRef * __nullable err);
//通过路径为CGImageMetadata设置标签
bool CGImageMetadataSetTagWithPath(CGMutableImageMetadataRef __nonnull metadata, CGImageMetadataTagRef __nullable parent, CFStringRef __nonnull path, CGImageMetadataTagRef __nonnull tag);
//通过路径为CGImageMetadata设置标签的值
bool CGImageMetadataSetValueWithPath(CGMutableImageMetadataRef __nonnull metadata, CGImageMetadataTagRef __nullable parent, CFStringRef __nonnull path, CFTypeRef __nonnull value);
//通过路径移除一个标签
bool CGImageMetadataRemoveTagWithPath(CGMutableImageMetadataRef __nonnull metadata,  CGImageMetadataTagRef __nullable parent, CFStringRef __nonnull path);
//对标签进行枚举
void CGImageMetadataEnumerateTagsUsingBlock(CGImageMetadataRef __nonnull metadata, CFStringRef __nullable rootPath, CFDictionaryRef __nullable options, CGImageMetadataTagBlock __nonnull block);
```

### 五、CGImageProperties中定义的字典意义

    前面提到的CGImageSourceCopyProperties方法与CGImageSourceCopyPropertiesAtIndex方法都会返回一个字典，字典中可能包含如下有意义的键：

```objectivec
//TIFF信息字典
const CFStringRef kCGImagePropertyTIFFDictionary;
/GIF信息字典
const CFStringRef kCGImagePropertyGIFDictionary;
//JFIF信息字典
const CFStringRef kCGImagePropertyJFIFDictionary;
//EXif信息字典
const CFStringRef kCGImagePropertyExifDictionary;
//PNG信息字典
const CFStringRef kCGImagePropertyPNGDictionary;
//IPTC信息字典
const CFStringRef kCGImagePropertyIPTCDictionary;
//GPS信息字典
const CFStringRef kCGImagePropertyGPSDictionary;
//原始信息字典
const CFStringRef kCGImagePropertyRawDictionary;
//CIFF信息字典
const CFStringRef kCGImagePropertyCIFFDictionary;
//佳能相机信息字典
const CFStringRef kCGImagePropertyMakerCanonDictionary;
//尼康相机信息字典
const CFStringRef kCGImagePropertyMakerNikonDictionary;
//柯尼卡相机信息字典
const CFStringRef kCGImagePropertyMakerMinoltaDictionary;
//富士相机信息字典
const CFStringRef kCGImagePropertyMakerFujiDictionary;
//奥林巴斯相机信息字典
const CFStringRef kCGImagePropertyMakerOlympusDictionary;
//宾得相机信息字典
const CFStringRef kCGImagePropertyMakerPentaxDictionary;
//对应Photoshop相片的信息字典
const CFStringRef kCGImageProperty8BIMDictionary;
//NDG信息字典
const CFStringRef kCGImagePropertyDNGDictionary ;
//ExifAux信息字典
const CFStringRef kCGImagePropertyExifAuxDictionary;
//OpenEXR信息字典
const CFStringRef kCGImagePropertyOpenEXRDictionary;
//Apple相机信息字典
const CFStringRef kCGImagePropertyMakerAppleDictionary ;

```

CGImageSourceCopyProperties方法返回的字典中还可能会有如下一个特殊的键：

```objectivec
//对应文件大小
const CFStringRef kCGImagePropertyFileSize;
```

CGImageSourceCopyPropertiesAtIndex方法中可能包含的特殊键：

```objectivec
//像素高度
const CFStringRef kCGImagePropertyPixelHeight;
//像素宽度
const CFStringRef kCGImagePropertyPixelWidth;
//DPI高度
const CFStringRef kCGImagePropertyDPIHeight;
//DPI宽度
const CFStringRef kCGImagePropertyDPIWidth;
//颜色位数
const CFStringRef kCGImagePropertyDepth;
//图片的显示方向
/*
对应Number值
 *   1  =  左上到右下.  
 *   2  =  右上到左下.  
 *   3  =  右下到左上.
 *   4  =  左下到右上.  
 *   5  =  行列置换 左上到右下.  
 *   6  =  行列置换 右上到左下.  
 *   7  =  行列置换 右下到左上.  
 *   8  =  行列置换 左下到右上.
*/
const CFStringRef kCGImagePropertyOrientation;
//颜色是否支持浮点数
const CFStringRef kCGImagePropertyIsFloat;
//图像是否包含像素样本
const CFStringRef kCGImagePropertyIsIndexed;
//图像是否包含alpha通道
const CFStringRef kCGImagePropertyHasAlpha;
//图像的颜色模式
const CFStringRef kCGImagePropertyColorModel;
//嵌入图片的ICC配置文件名称
const CFStringRef kCGImagePropertyProfileName;

```

kCGImagePropertyColorModel键可返回的值有如下几种定义：

```objectivec
//RBG模式
const CFStringRef kCGImagePropertyColorModelRGB;
//Gray模式
const CFStringRef kCGImagePropertyColorModelGray;
//CMYK模式
const CFStringRef kCGImagePropertyColorModelCMYK;
//Lab模式
const CFStringRef kCGImagePropertyColorModelLab;
```

kCGImagePropertyTIFFDictionary键可返回的值定义如下：

```objectivec
//图片数据压缩方案
const CFStringRef kCGImagePropertyTIFFCompression;
//图片数据的色彩空间
const CFStringRef kCGImagePropertyTIFFPhotometricInterpretation;
//文档名称
const CFStringRef kCGImagePropertyTIFFDocumentName;
//图片描述
const CFStringRef kCGImagePropertyTIFFImageDescription;
//相机设备名
const CFStringRef kCGImagePropertyTIFFMake;
//相机设备模式
const CFStringRef kCGImagePropertyTIFFModel;
//图片方向
const CFStringRef kCGImagePropertyTIFFOrientation;
//横向每个分辨位的像素数
const CFStringRef kCGImagePropertyTIFFXResolution;
//纵向每个分辨位的像素数
const CFStringRef kCGImagePropertyTIFFYResolution;
//分辨率单位
const CFStringRef kCGImagePropertyTIFFResolutionUnit;
//创建图像的软件名称和版本
const CFStringRef kCGImagePropertyTIFFSoftware;
//transform函数
const CFStringRef kCGImagePropertyTIFFTransferFunction;
//日期时间
const CFStringRef kCGImagePropertyTIFFDateTime;
//作者
const CFStringRef kCGImagePropertyTIFFArtist;
//创建图片的电脑系统
const CFStringRef kCGImagePropertyTIFFHostComputer;
//公司信息
const CFStringRef kCGImagePropertyTIFFCopyright;
//图片的白点
const CFStringRef kCGImagePropertyTIFFWhitePoint;
//图像的原色色度
const CFStringRef kCGImagePropertyTIFFPrimaryChromaticities;
//图片的瓦片宽度
const CFStringRef kCGImagePropertyTIFFTileWidth;
//图片的瓦片高度
const CFStringRef kCGImagePropertyTIFFTileLength;
```

kCGImagePropertyJFIFDictionary对应的字典中可能包含如下意义的键：

```objectivec
//JFIF版本
const CFStringRef kCGImagePropertyJFIFVersion;
//横向像素密度
const CFStringRef kCGImagePropertyJFIFXDensity;
//纵向像素密度
const CFStringRef kCGImagePropertyJFIFYDensity;
//像素密度单元
const CFStringRef kCGImagePropertyJFIFDensityUnit;
//是否是高质量图像版本
const CFStringRef kCGImagePropertyJFIFIsProgressive;
```

kCGImagePropertyExifDictionary对应的字典中可能包含如下意义的键 ：

```objectivec
//曝光时间
const CFStringRef kCGImagePropertyExifExposureTime;
//ExifNumber
const CFStringRef kCGImagePropertyExifFNumber;
//曝光程序
const CFStringRef kCGImagePropertyExifExposureProgram;
//每个通道的光谱灵敏度
const CFStringRef kCGImagePropertyExifSpectralSensitivity;
//ISO速度等级
const CFStringRef kCGImagePropertyExifISOSpeedRatings;
//ExifOECF
const CFStringRef kCGImagePropertyExifOECF;
//灵敏类型
const CFStringRef kCGImagePropertyExifSensitivityType;
//输出灵敏标准
const CFStringRef kCGImagePropertyExifStandardOutputSensitivity;
//推荐曝光指数
const CFStringRef kCGImagePropertyExifRecommendedExposureIndex;
//ISO速率
const CFStringRef kCGImagePropertyExifISOSpeed;
const CFStringRef kCGImagePropertyExifISOSpeedLatitudeyyy;
const CFStringRef kCGImagePropertyExifISOSpeedLatitudezzz;
//Exif版本
const CFStringRef kCGImagePropertyExifVersion;
//原始日期时间
const CFStringRef kCGImagePropertyExifDateTimeOriginal;
//数字化日期时间
const CFStringRef kCGImagePropertyExifDateTimeDigitized;
//压缩配置
const CFStringRef kCGImagePropertyExifComponentsConfiguration;
//压缩模式像素位
const CFStringRef kCGImagePropertyExifCompressedBitsPerPixel;
//快门速度值
const CFStringRef kCGImagePropertyExifShutterSpeedValue;
//孔径值
const CFStringRef kCGImagePropertyExifApertureValue;
//亮度值
const CFStringRef kCGImagePropertyExifBrightnessValue;
//曝光偏差值
const CFStringRef kCGImagePropertyExifExposureBiasValue;
//最大光圈值
const CFStringRef kCGImagePropertyExifMaxApertureValue;
//距离
const CFStringRef kCGImagePropertyExifSubjectDistance;
//测光模式
const CFStringRef kCGImagePropertyExifMeteringMode;
//光源
const CFStringRef kCGImagePropertyExifLightSource;
//拍摄时的闪光状态
const CFStringRef kCGImagePropertyExifFlash;
//焦距
const CFStringRef kCGImagePropertyExifFocalLength;
//主体区域
const CFStringRef kCGImagePropertyExifSubjectArea;
//相机制造商指定的信息
const CFStringRef kCGImagePropertyExifMakerNote;
//用户信息
const CFStringRef kCGImagePropertyExifUserComment;
//日期和时间标记的秒分数
const CFStringRef kCGImagePropertyExifSubsecTime;
//原始时间
const CFStringRef kCGImagePropertyExifSubsecTimeOriginal;
//数字时间
const CFStringRef kCGImagePropertyExifSubsecTimeDigitized;
//FlashPix版本信息
const CFStringRef kCGImagePropertyExifFlashPixVersion;
//色彩空间
const CFStringRef kCGImagePropertyExifColorSpace;
//X方向像素
const CFStringRef kCGImagePropertyExifPixelXDimension;
//Y方向像素
const CFStringRef kCGImagePropertyExifPixelYDimension;
//与图像相关的声音文件
const CFStringRef kCGImagePropertyExifRelatedSoundFile;
//FlashEnergy
const CFStringRef kCGImagePropertyExifFlashEnergy;
//FrequencyResponse
const CFStringRef kCGImagePropertyExifSpatialFrequencyResponse;
//像素数目
const CFStringRef kCGImagePropertyExifFocalPlaneXResolution;
const CFStringRef kCGImagePropertyExifFocalPlaneYResolution;
const CFStringRef kCGImagePropertyExifFocalPlaneResolutionUnit;
//图像主体的位置
const CFStringRef kCGImagePropertyExifSubjectLocation;
//选择的曝光指数
const CFStringRef kCGImagePropertyExifExposureIndex;
//传感器类型
const CFStringRef kCGImagePropertyExifSensingMethod;
//图像文件源
const CFStringRef kCGImagePropertyExifFileSource;
//场景类型
const CFStringRef kCGImagePropertyExifSceneType;
//CFA模块
const CFStringRef kCGImagePropertyExifCFAPattern;
//对图像数据进行特殊渲染
const CFStringRef kCGImagePropertyExifCustomRendered;
//曝光模式设置
const CFStringRef kCGImagePropertyExifExposureMode;
//白平衡模式
const CFStringRef kCGImagePropertyExifWhiteBalance;
//数字变焦比
const CFStringRef kCGImagePropertyExifDigitalZoomRatio;
//35毫米胶片的等效焦距
const CFStringRef kCGImagePropertyExifFocalLenIn35mmFilm;
//场景捕捉类型（标准，景观，肖像，夜晚）
const CFStringRef kCGImagePropertyExifSceneCaptureType;
//图像增益
const CFStringRef kCGImagePropertyExifGainControl;
//图像对比度
const CFStringRef kCGImagePropertyExifContrast;
//图像饱和度
const CFStringRef kCGImagePropertyExifSaturation;
//图像锐度
const CFStringRef kCGImagePropertyExifSharpness;
//拍摄条件
const CFStringRef kCGImagePropertyExifDeviceSettingDescription;
//主体距离
const CFStringRef kCGImagePropertyExifSubjectDistRange;
//图像的唯一标识
const CFStringRef kCGImagePropertyExifImageUniqueID;
//相机所有者
const CFStringRef kCGImagePropertyExifCameraOwnerName;
//相机序列号
const CFStringRef kCGImagePropertyExifBodySerialNumber;
//透镜规格信息
const CFStringRef kCGImagePropertyExifLensSpecification;
//透镜制造商名称
const CFStringRef kCGImagePropertyExifLensMake;
//透镜模式
const CFStringRef kCGImagePropertyExifLensModel;
//透镜序列号
const CFStringRef kCGImagePropertyExifLensSerialNumber;
//伽马设置
const CFStringRef kCGImagePropertyExifGamma;
```

kCGImagePropertyExifAuxDictionary对应的字典中可能包含的键定义如下：

```objectivec
//镜头信息
const CFStringRef kCGImagePropertyExifAuxLensInfo;
//镜头模式
const CFStringRef kCGImagePropertyExifAuxLensModel;
//序列号
const CFStringRef kCGImagePropertyExifAuxSerialNumber;
//镜头ID
const CFStringRef kCGImagePropertyExifAuxLensID;
//镜头序列号
const CFStringRef kCGImagePropertyExifAuxLensSerialNumber;
//图片编号
const CFStringRef kCGImagePropertyExifAuxImageNumber;
//闪光补偿
const CFStringRef kCGImagePropertyExifAuxFlashCompensation;
//所有者名称
const CFStringRef kCGImagePropertyExifAuxOwnerName;
//固件信息
const CFStringRef kCGImagePropertyExifAuxFirmware;
```

kCGImagePropertyGIFDictionary对应的字典中可能包含的键定义如下：

```objectivec
//动画循环次数
const CFStringRef kCGImagePropertyGIFLoopCount;
//两帧之间的延时
const CFStringRef kCGImagePropertyGIFDelayTime;
//颜色Map
const CFStringRef kCGImagePropertyGIFImageColorMap;
const CFStringRef kCGImagePropertyGIFHasGlobalColorMap;
//两帧之间的延时
const CFStringRef kCGImagePropertyGIFUnclampedDelayTime;
```

kCGImagePropertyPNGDictionary对应的字典中可能包含的键定义如下：

```objectivec
//PNG伽马值
const CFStringRef kCGImagePropertyPNGGamma;
//混合类型
const CFStringRef kCGImagePropertyPNGInterlaceType;
//X方向像素数
const CFStringRef kCGImagePropertyPNGXPixelsPerMeter;
//Y方向像素数
const CFStringRef kCGImagePropertyPNGYPixelsPerMeter;
//RGB意图
const CFStringRef kCGImagePropertyPNGsRGBIntent;
//色度
const CFStringRef kCGImagePropertyPNGChromaticities;
//作者
const CFStringRef kCGImagePropertyPNGAuthor;
//公司
const CFStringRef kCGImagePropertyPNGCopyright;
//创建时间
const CFStringRef kCGImagePropertyPNGCreationTime;
//描述
const CFStringRef kCGImagePropertyPNGDescription;
//最后修改日期时间
const CFStringRef kCGImagePropertyPNGModificationTime;
//软件
const CFStringRef kCGImagePropertyPNGSoftware;
//标题
const CFStringRef kCGImagePropertyPNGTitle;
//动画循环次数
const CFStringRef kCGImagePropertyAPNGLoopCount;
//两帧之间的延时
const CFStringRef kCGImagePropertyAPNGDelayTime;
const CFStringRef kCGImagePropertyAPNGUnclampedDelayTime;
```

kCGImagePropertyGPSDictionary对应的字典中可能包含的键定义如下：

```objectivec
//GPS版本
const CFStringRef kCGImagePropertyGPSVersion;
//纬度是南纬或北纬
const CFStringRef kCGImagePropertyGPSLatitudeRef;
//纬度
const CFStringRef kCGImagePropertyGPSLatitude;
//经度是东经或西经
const CFStringRef kCGImagePropertyGPSLongitudeRef;
//经度
const CFStringRef kCGImagePropertyGPSLongitude;
//海拔标准
const CFStringRef kCGImagePropertyGPSAltitudeRef;
//海拔高度
const CFStringRef kCGImagePropertyGPSAltitude;
//时间戳
const CFStringRef kCGImagePropertyGPSTimeStamp;
//测量GPS的卫星
const CFStringRef kCGImagePropertyGPSSatellites;
//GPS状态
const CFStringRef kCGImagePropertyGPSStatus;
//测量模式
const CFStringRef kCGImagePropertyGPSMeasureMode;
//精度数据
const CFStringRef kCGImagePropertyGPSDOP;
//速度标准
const CFStringRef kCGImagePropertyGPSSpeedRef;
//速度
const CFStringRef kCGImagePropertyGPSSpeed;
//运动方向参考
const CFStringRef kCGImagePropertyGPSTrackRef;
//运动方向
const CFStringRef kCGImagePropertyGPSTrack;
//位置方向参考
const CFStringRef kCGImagePropertyGPSImgDirectionRef;
//位置方向
const CFStringRef kCGImagePropertyGPSImgDirection;
//地图测量数据
const CFStringRef kCGImagePropertyGPSMapDatum;
//地理纬度南纬或北纬
const CFStringRef kCGImagePropertyGPSDestLatitudeRef;
//地理纬度
const CFStringRef kCGImagePropertyGPSDestLatitude;
//地理经度 东经或西经
const CFStringRef kCGImagePropertyGPSDestLongitudeRef;
//地理经度
const CFStringRef kCGImagePropertyGPSDestLongitude;
//方位参照
const CFStringRef kCGImagePropertyGPSDestBearingRef;
//地理方位
const CFStringRef kCGImagePropertyGPSDestBearing;
//距离参照
const CFStringRef kCGImagePropertyGPSDestDistanceRef;
//距离
const CFStringRef kCGImagePropertyGPSDestDistance;
//查找地理位置的方法
const CFStringRef kCGImagePropertyGPSProcessingMethod;
//GPS地区名
const CFStringRef kCGImagePropertyGPSAreaInformation;
//日期时间
const CFStringRef kCGImagePropertyGPSDateStamp;
//校正信息
const CFStringRef kCGImagePropertyGPSDifferental;
//错误信息
const CFStringRef kCGImagePropertyGPSHPositioningError;
```

kCGImagePropertyIPTCDictionary对应的字典中可能包含的键定义如下：

```objectivec
//对象类型
const CFStringRef kCGImagePropertyIPTCObjectTypeReference;
//对象属性
const CFStringRef kCGImagePropertyIPTCObjectAttributeReference;
//对象名称
const CFStringRef kCGImagePropertyIPTCObjectName;
//编辑状态
const CFStringRef kCGImagePropertyIPTCEditStatus;
//更新状态
const CFStringRef kCGImagePropertyIPTCEditorialUpdate;
//紧急等级
const CFStringRef kCGImagePropertyIPTCUrgency;
//主体
const CFStringRef kCGImagePropertyIPTCSubjectReference;
//类别
const CFStringRef kCGImagePropertyIPTCCategory;
//补充类别
const CFStringRef kCGImagePropertyIPTCSupplementalCategory;
//Fixture标识
const CFStringRef kCGImagePropertyIPTCFixtureIdentifier;
//关键字
const CFStringRef kCGImagePropertyIPTCKeywords;
//内容定位码
const CFStringRef kCGImagePropertyIPTCContentLocationCode;
//内容位置名称
const CFStringRef kCGImagePropertyIPTCContentLocationName;
//图像使用的最早日期
const CFStringRef kCGImagePropertyIPTCReleaseDate;
//图像使用的最早时间
const CFStringRef kCGImagePropertyIPTCReleaseTime;
//最后一次使用日期
const CFStringRef kCGImagePropertyIPTCExpirationDate;
//最后一次使用时间
const CFStringRef kCGImagePropertyIPTCExpirationTime;
//图像使用的特别说明
const CFStringRef kCGImagePropertyIPTCSpecialInstructions;
//建议行为
const CFStringRef kCGImagePropertyIPTCActionAdvised;
//服务参考
const CFStringRef kCGImagePropertyIPTCReferenceService;
//日期参考
const CFStringRef kCGImagePropertyIPTCReferenceDate;
//参考码
const CFStringRef kCGImagePropertyIPTCReferenceNumber;
//创建日期
const CFStringRef kCGImagePropertyIPTCDateCreated;
//创建时间
const CFStringRef kCGImagePropertyIPTCTimeCreated;
//数字创建日期
const CFStringRef kCGImagePropertyIPTCDigitalCreationDate;
//数字创建时间
const CFStringRef kCGImagePropertyIPTCDigitalCreationTime;
//原始程序
const CFStringRef kCGImagePropertyIPTCOriginatingProgram;
//程序版本
const CFStringRef kCGImagePropertyIPTCProgramVersion;
图像的编辑周期（早晨，晚上或两者）。
const CFStringRef kCGImagePropertyIPTCObjectCycle;
//不想创建者名称
const CFStringRef kCGImagePropertyIPTCByline;
//图像创建标题
const CFStringRef kCGImagePropertyIPTCBylineTitle;
//城市信息
const CFStringRef kCGImagePropertyIPTCCity;
//城市内位置
const CFStringRef kCGImagePropertyIPTCSubLocation;
//省份
const CFStringRef kCGImagePropertyIPTCProvinceState;
//国家编码
const CFStringRef kCGImagePropertyIPTCCountryPrimaryLocationCode;
//国家名称
const CFStringRef kCGImagePropertyIPTCCountryPrimaryLocationName;
//OriginalTransmission参考
const CFStringRef kCGImagePropertyIPTCOriginalTransmissionReference;
//图像内容摘要
const CFStringRef kCGImagePropertyIPTCHeadline;
//提供图像服务的名称
const CFStringRef kCGImagePropertyIPTCCredit;
//图像源
const CFStringRef kCGImagePropertyIPTCSource;
//公司提示
const CFStringRef kCGImagePropertyIPTCCopyrightNotice;
//联系人
const CFStringRef kCGImagePropertyIPTCContact;
//描述
const CFStringRef kCGImagePropertyIPTCCaptionAbstract;
//图像编辑者
const CFStringRef kCGImagePropertyIPTCWriterEditor;
//图像类型
const CFStringRef kCGImagePropertyIPTCImageType;
//方向信息
const CFStringRef kCGImagePropertyIPTCImageOrientation;
//语言信息
const CFStringRef kCGImagePropertyIPTCLanguageIdentifier;
//星级
const CFStringRef kCGImagePropertyIPTCStarRating;
//联系人详细信息
const CFStringRef kCGImagePropertyIPTCCreatorContactInfo;
//图像使用权限
const CFStringRef kCGImagePropertyIPTCRightsUsageTerms;
//场景代码
const CFStringRef kCGImagePropertyIPTCScene;
```

上面的kCGImagePropertyIPTCCreatorContactInfo对应的字典中键的定义如下：

```objectivec
//联系人城市
const CFStringRef kCGImagePropertyIPTCContactInfoCity;
//联系人国家
const CFStringRef kCGImagePropertyIPTCContactInfoCountry;
//联系人地址
const CFStringRef kCGImagePropertyIPTCContactInfoAddress;
//邮编
const CFStringRef kCGImagePropertyIPTCContactInfoPostalCode;
//省份
const CFStringRef kCGImagePropertyIPTCContactInfoStateProvince;
//电子邮件
const CFStringRef kCGImagePropertyIPTCContactInfoEmails;
//电话
const CFStringRef kCGImagePropertyIPTCContactInfoPhones;
//网址
const CFStringRef kCGImagePropertyIPTCContactInfoWebURLs;
```

kCGImageProperty8BIMDictionary对应的字典中可能包含的键定义如下：

```objectivec
//Photoshop文件的图层名
const CFStringRef  kCGImageProperty8BIMLayerNames;
//版本
const CFStringRef  kCGImageProperty8BIMVersion;
```

kCGImagePropertyDNGDictionary对应的字典中可能包含的键定义如下：

```objectivec
//DNG版本
const CFStringRef  kCGImagePropertyDNGVersion;
//兼容的最老版本
const CFStringRef  kCGImagePropertyDNGBackwardVersion;
//摄像机模型
const CFStringRef  kCGImagePropertyDNGUniqueCameraModel;
const CFStringRef  kCGImagePropertyDNGLocalizedCameraModel;
//相机序列码
const CFStringRef  kCGImagePropertyDNGCameraSerialNumber;
//镜头信息
const CFStringRef  kCGImagePropertyDNGLensInfo;
//黑度等级
const CFStringRef  kCGImagePropertyDNGBlackLevel;
//白度等级
const CFStringRef  kCGImagePropertyDNGWhiteLevel;

const CFStringRef  kCGImagePropertyDNGCalibrationIlluminant1;
const CFStringRef  kCGImagePropertyDNGCalibrationIlluminant2;
const CFStringRef  kCGImagePropertyDNGColorMatrix1;
const CFStringRef  kCGImagePropertyDNGColorMatrix2;
const CFStringRef  kCGImagePropertyDNGCameraCalibration1;
const CFStringRef  kCGImagePropertyDNGCameraCalibration2;
const CFStringRef  kCGImagePropertyDNGAsShotNeutral;
const CFStringRef  kCGImagePropertyDNGAsShotWhiteXY;
const CFStringRef  kCGImagePropertyDNGBaselineExposure;
const CFStringRef  kCGImagePropertyDNGBaselineNoise;
const CFStringRef  kCGImagePropertyDNGBaselineSharpness;
const CFStringRef  kCGImagePropertyDNGPrivateData;
const CFStringRef  kCGImagePropertyDNGCameraCalibrationSignature;
const CFStringRef  kCGImagePropertyDNGProfileCalibrationSignature;
const CFStringRef  kCGImagePropertyDNGNoiseProfile;
const CFStringRef  kCGImagePropertyDNGWarpRectilinear;
const CFStringRef  kCGImagePropertyDNGWarpFisheye;
const CFStringRef  kCGImagePropertyDNGFixVignetteRadial;
```

kCGImagePropertyCIFFDictionary对应的字典中可能包含的键定义如下：

```objectivec
//相机信息
const CFStringRef  kCGImagePropertyCIFFDescription;
//固件版本
const CFStringRef  kCGImagePropertyCIFFFirmware;
//所有者名称
const CFStringRef  kCGImagePropertyCIFFOwnerName;
//图片名
const CFStringRef  kCGImagePropertyCIFFImageName;
//图片文件名
const CFStringRef  kCGImagePropertyCIFFImageFileName;
//曝光方式
const CFStringRef  kCGImagePropertyCIFFReleaseMethod;
//曝光时间
const CFStringRef  kCGImagePropertyCIFFReleaseTiming;
//RecordID
const CFStringRef  kCGImagePropertyCIFFRecordID;
//曝光时间
const CFStringRef  kCGImagePropertyCIFFSelfTimingTime;
//相机序列号
const CFStringRef  kCGImagePropertyCIFFCameraSerialNumber;
//图片编码
const CFStringRef  kCGImagePropertyCIFFImageSerialNumber;
//驱动模式
const CFStringRef  kCGImagePropertyCIFFContinuousDrive);
//焦点模式
const CFStringRef  kCGImagePropertyCIFFFocusMode;
//测量模式
const CFStringRef  kCGImagePropertyCIFFMeteringMode;
//曝光模式
const CFStringRef  kCGImagePropertyCIFFShootingMode;
//透镜模式
const CFStringRef  kCGImagePropertyCIFFLensModel;
//最长镜头长度
const CFStringRef  kCGImagePropertyCIFFLensMaxMM;
//最短镜头长度
const CFStringRef  kCGImagePropertyCIFFLensMinMM;
//白平衡等级
const CFStringRef  kCGImagePropertyCIFFWhiteBalanceIndex;
//曝光补偿
const CFStringRef  kCGImagePropertyCIFFFlashExposureComp;
//实测曝光值
const CFStringRef  kCGImagePropertyCIFFMeasuredEV);

```

### 六、ImageIO框架在实际开发中的几个应用

#### 1.显示特殊格式的图片

    在平时开发中，我们通常使用UIImage来读取图片，UIImage支持的图片包括png与jpg等，但是类似windows系统的ico图标，UIImage默认是无法显示的，可以通过ImageIO框架来在iOS系统中使用ico图标，示例如下：

```objectivec
    NSString * path = [[NSBundle mainBundle]pathForResource:@"image" ofType:@"ico"];
    NSURL * url = [NSURL fileURLWithPath:path];
    CGImageRef myImage = NULL;
    CGImageSourceRef myImageSource;
    CFDictionaryRef myOptions = NULL;
    myImageSource = CGImageSourceCreateWithURL((CFURLRef)url, NULL);
    myImage = CGImageSourceCreateImageAtIndex(myImageSource,
                                              0,
                                              NULL);
    CFRelease(myImageSource);
    UIImageView * image = [[UIImageView alloc]initWithFrame:CGRectMake(0, 0, 200, 200)];
    image.image = [UIImage imageWithCGImage:myImage];
```

#### 2.读取数码相机拍摄图片的地理位置、时间等信息

#### 3.对相册中图片的地理位置，时间等信息进行自定义修改。

#### 4.将自定义格式的图片数据写入本地文件。

#### 5.展示GIF动图

    详情见博客：[https://my.oschina.net/u/2340880/blog/608560](https://my.oschina.net/u/2340880/blog/608560)。

#### 6.渐进渲染大图

    渐进渲染技术在对加载大图片时特别重要，你应该使用过地图软件，地图视图在加载时是局部进行加载，当移动或者放大时，地图会一部分一部分的渐进进行加载，使用ImageIO框架可以实现大图渐进渲染的效果，一般在对大图片进行网络请求时，可以获取一部分数据就加载一部分数据，为了便于演示，博客中使用定时器来默认网络返回数据，代码示例如下：

```objectivec
@interface ViewController ()
{
    NSMutableData * _data;
    NSData * _allData;
    NSUInteger length;
    UIImageView * _imageView;
    NSTimer * timer;
    NSInteger le;
}
@end
@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    _data = [[NSMutableData alloc]init];
    NSString * path = [[NSBundle mainBundle]pathForResource:@"Default-Portrait-ns@2x" ofType:@"png"];
    _allData = [NSData dataWithContentsOfFile:path];
    length = _allData.length;
    le = length/10;
    timer = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(updateImage) userInfo:nil repeats:YES];
    _imageView = [[UIImageView alloc]initWithFrame:self.view.frame];
    [self.view addSubview:_imageView];
}


-(void)updateImage{
    static int index = 0;
    if (index==10) {
        return;
    }
    NSUInteger l;
    if (index==9) {
        l=length-le*9;
    }else{
        l= le;
    }
    
    Byte by[l];
    [_allData getBytes:by range:NSMakeRange(index*le, l)];
    [_data appendBytes:by length:l];
    CGImageSourceRef myImageSource = CGImageSourceCreateWithData((CFDataRef)_data, NULL);
    CGImageRef myImage = CGImageSourceCreateImageAtIndex(myImageSource,
                                              0,
                                              NULL);
    CFRelease(myImageSource);
    
    _imageView.image = [UIImage imageWithCGImage:myImage];
    //    image.image = [UIImage imageNamed:@"image.ico"];
    index++;
}
@end
```

效果如下：

![](https://static.oschina.net/uploads/space/2017/0214/185328_kOrD_2340880.gif)

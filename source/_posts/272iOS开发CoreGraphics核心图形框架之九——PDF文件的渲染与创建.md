---
title: iOS开发CoreGraphics核心图形框架之九——PDF文件的渲染与创建
date: 2016-12-07
categories: iOS逻辑初窥
tags: []
---
## iOS开发CoreGraphics核心图形框架之九——PDF文件的渲染与创建

### 一、渲染已有的PDF文档

    在CoreGraphics框架中，有两个类型与PDF文档的渲染有关，分别为CGPDFDocumentRef与CGPDFPageRef。其中，CGPDFDocumentRef对应整个PDF文档，里面封装了许多文档相关的信息，CGPDFPageRef对应PDF文档中某一页的内容，通过它开发者可以将PDF内容通过CGContext上下文渲染到指定目标上。

    如下代码演示了在自定义View的drawRect:方法中进行PDF文档的绘制：

```objectivec
-(void)drawRect:(CGRect)rect{
    //由于坐标系不同，需要进行翻转
    CGContextRef contextRef = UIGraphicsGetCurrentContext();
    //进行坐标系的翻转
    CGContextTranslateCTM(contextRef, 0, rect.size.height);
    CGContextScaleCTM(contextRef, 1.0, -1.0);
    //获取pdf文件的路径
    NSString * path = [[NSBundle mainBundle] pathForResource:@"MyText" ofType:@"pdf"];
    CFStringRef pathString = CFStringCreateWithCString(NULL, [path cStringUsingEncoding:NSUTF8StringEncoding], kCFStringEncodingUTF8);
    //创建url
    CFURLRef url = CFURLCreateWithFileSystemPath(NULL, pathString, kCFURLPOSIXPathStyle, 0);
    CFRelease(pathString);
    //进行CGPDFDocumentRef引用的创建
    CGPDFDocumentRef document = CGPDFDocumentCreateWithURL(url);
    CFRelease(url);
    //获取文档的第1页
    CGPDFPageRef page1 = CGPDFDocumentGetPage(document, 1);
    //进行绘制
    CGContextDrawPDFPage(contextRef, page1);
    CGPDFPageRelease(page1);
    CGPDFDocumentRelease(document);
}
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1207/105816_K9P1_2340880.png)

CGPDFDocument中提供的方法解析如下：

```objectivec
//通过数据提供者类来创建PDF文档对象
CGPDFDocumentRef  CGPDFDocumentCreateWithProvider(CGDataProviderRef cg_nullable provider);
//通过url来创建PDF文档
CGPDFDocumentRef  CGPDFDocumentCreateWithURL(CFURLRef cg_nullable url);
//进行引用计数+1
CGPDFDocumentRef  CGPDFDocumentRetain(CGPDFDocumentRef cg_nullable document);
//进行引用计数-1,需要注意，其作用和CFRelease()相似，不同的是如果document为NULL，不是发生crash
void CGPDFDocumentRelease(CGPDFDocumentRef cg_nullable document);
//获取PDF文档的版本
void CGPDFDocumentGetVersion(CGPDFDocumentRef cg_nullable document, int *  majorVersion, int *  minorVersion);
//判断文档是否是加密的
bool CGPDFDocumentIsEncrypted(CGPDFDocumentRef cg_nullable document);
//使用密码对PDF文档进行解密 返回值为1表示解密成功
bool CGPDFDocumentUnlockWithPassword(CGPDFDocumentRef cg_nullable document, const char *  password);
//判断PDF文档是否已经解锁
bool CGPDFDocumentIsUnlocked(CGPDFDocumentRef cg_nullable document);
//获取此PDF文档是否允许绘制
bool CGPDFDocumentAllowsPrinting(CGPDFDocumentRef cg_nullable document);
//获取此文档是否允许拷贝
bool CGPDFDocumentAllowsCopying(CGPDFDocumentRef cg_nullable document);
//获取PDF文档的总页数
size_t CGPDFDocumentGetNumberOfPages(CGPDFDocumentRef cg_nullable document);
//获取文档中某页数据
CGPDFPageRef __nullable CGPDFDocumentGetPage(CGPDFDocumentRef cg_nullable document, size_t pageNumber);
//获取文档的目录信息
CGPDFDictionaryRef __nullable CGPDFDocumentGetCatalog(CGPDFDocumentRef cg_nullable document);
//获取文档详情信息
CGPDFDictionaryRef __nullable CGPDFDocumentGetInfo(CGPDFDocumentRef cg_nullable document);
//获取文档id
CGPDFArrayRef __nullable CGPDFDocumentGetID(CGPDFDocumentRef cg_nullable document);
//获取CGPDFDocument类在CoreGraphics框架中的id
CFTypeID CGPDFDocumentGetTypeID(void);


```

CGPDFDocument中还有一些已经弃用的方法，这些方法现在封装在CGPDFPage中，弃用的方法如下：

```objectivec
CGRect CGPDFDocumentGetMediaBox(CGPDFDocumentRef cg_nullable document,int page);
CGRect CGPDFDocumentGetCropBox(CGPDFDocumentRef cg_nullable document, int page);
CGRect CGPDFDocumentGetBleedBox(CGPDFDocumentRef cg_nullable document, int page);
CGRect CGPDFDocumentGetTrimBox(CGPDFDocumentRef cg_nullable document, int page);
CGRect CGPDFDocumentGetArtBox(CGPDFDocumentRef cg_nullable document, int page);
int CGPDFDocumentGetRotationAngle(CGPDFDocumentRef cg_nullable document, int page);
```

CGPDFPage中的主要方法列举如下：

```objectivec
//进行引用计数+1
CGPDFPageRef CGPDFPageRetain(CGPDFPageRef cg_nullable page);
//进行引用计数-1
void CGPDFPageRelease(CGPDFPageRef cg_nullable page);
//获取对应的PDF文档对象
CGPDFDocumentRef __nullable CGPDFPageGetDocument(CGPDFPageRef cg_nullable page);
//获取当前页是文档中的第几页
size_t CGPDFPageGetPageNumber(CGPDFPageRef cg_nullable page);
//获取与文档此页相关联的媒体区域
/*
typedef CF_ENUM (int32_t, CGPDFBox) {
  kCGPDFMediaBox = 0,
  kCGPDFCropBox = 1,
  kCGPDFBleedBox = 2,
  kCGPDFTrimBox = 3,
  kCGPDFArtBox = 4
};
*/
CGRect CGPDFPageGetBoxRect(CGPDFPageRef cg_nullable page, CGPDFBox box);
//获取此页的旋转角度
int CGPDFPageGetRotationAngle(CGPDFPageRef cg_nullable page);
//transform变换
CGAffineTransform CGPDFPageGetDrawingTransform(CGPDFPageRef cg_nullable page, CGPDFBox box, CGRect rect, int rotate, bool preserveAspectRatio);
```

### 二、使用代码创建PDF文件

    如下示例代码演示了创建PDF文档的过程：

```objectivec
-(void)creatPDF{
    //绘图上下文
    CGContextRef pdfContext;
    CFStringRef path;
    CFURLRef url;
    CFDataRef boxData = NULL;
    CFMutableDictionaryRef myDictionary = NULL;
    CFMutableDictionaryRef pageDictionary = NULL;
    //文件存放的路径
    NSString * filePath = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, true).firstObject;
    NSLog(@"%@",filePath);
    const char * filename = [[NSString stringWithFormat:@"%@/MyText",filePath] cStringUsingEncoding:kCFStringEncodingUTF8];
    path = CFStringCreateWithCString (NULL, filename,kCFStringEncodingUTF8);
    url = CFURLCreateWithFileSystemPath (NULL, path,kCFURLPOSIXPathStyle, 0);
    CFRelease (path);
    //文档信息字典
    myDictionary = CFDictionaryCreateMutable(NULL, 0,
                                             &kCFTypeDictionaryKeyCallBacks,
                                             &kCFTypeDictionaryValueCallBacks);
    //设置文档名称 
    CFDictionarySetValue(myDictionary, kCGPDFContextTitle, CFSTR("My PDF File"));
    //设置创建者
    CFDictionarySetValue(myDictionary, kCGPDFContextCreator, CFSTR("My Name"));
    //设置文档尺寸
    CGRect pageRect = CGRectMake(0, 0, 200, 200);
    //创建文档
    pdfContext = CGPDFContextCreateWithURL (url, &pageRect, myDictionary);
    CFRelease(myDictionary);
    CFRelease(url);
    //设置内容信息字典
    pageDictionary = CFDictionaryCreateMutable(NULL, 0,
                                               &kCFTypeDictionaryKeyCallBacks,
                                               &kCFTypeDictionaryValueCallBacks);
    boxData = CFDataCreate(NULL,(const UInt8 *)&pageRect, sizeof (CGRect));
    CFDictionarySetValue(pageDictionary, kCGPDFContextMediaBox, boxData);
    //开始渲染一页
    CGPDFContextBeginPage (pdfContext, pageDictionary);
    CGFloat  colors[4] = {1,0,0,1};
    CGContextSetFillColorSpace(pdfContext, CGColorSpaceCreateWithName(kCGColorSpaceGenericRGB));
    CGContextSetFillColor(pdfContext, colors);
    CGContextFillRect(pdfContext, CGRectMake(0, 0, 100, 100));
    //结束此页的渲染
    CGPDFContextEndPage (pdfContext);
    //开始新一页内容的渲染
    CGPDFContextBeginPage (pdfContext, pageDictionary);
    CGContextSetFillColorSpace(pdfContext, CGColorSpaceCreateWithName(kCGColorSpaceGenericRGB));
    CGContextSetFillColor(pdfContext, colors);
    CGContextFillRect(pdfContext, CGRectMake(0, 0, 100, 100));
    CGPDFContextEndPage (pdfContext);
    CGContextRelease (pdfContext);
    CFRelease(pageDictionary);
    CFRelease(boxData);

}

```

上面代码创建出的PDF文件如下图所示：

![](https://static.oschina.net/uploads/space/2016/1207/142450_sqm6_2340880.png)

在创建PDF文档时，开发者还可以使用如下列举的方法来对文档进行超链接添加，内容信息设置等：

```objectivec
//关闭文档上下文，关闭后将不能再次写入
void CGPDFContextClose(CGContextRef cg_nullable context);
//开启新一页内容的绘制
void CGPDFContextBeginPage(CGContextRef cg_nullable context, CFDictionaryRef __nullable pageInfo);
//结束当前页内容的绘制
void CGPDFContextEndPage(CGContextRef cg_nullable context);
//添加元数据
void CGPDFContextAddDocumentMetadata(CGContextRef cg_nullable context, CFDataRef __nullable metadata);
//为某个区域添加超链接
void CGPDFContextSetURLForRect(CGContextRef cg_nullable context, CFURLRef  url, CGRect rect);
//在文档的某个点添加一个目标
void CGPDFContextAddDestinationAtPoint(CGContextRef cg_nullable context, CFStringRef  name, CGPoint point);
//为某个区域添加跳转目标功能
void CGPDFContextSetDestinationForRect(CGContextRef cg_nullable context, CFStringRef  name, CGRect rect);
```

在设置文档信息字典时，支持的常用键如下：

```objectivec
//设置文档标题 可选设置
const CFStringRef  kCGPDFContextTitle;
//设置文档的作者 可选设置
const CFStringRef  kCGPDFContextAuthor;
//设置文档的副标题 可选设置
const CFStringRef  kCGPDFContextSubject;
//为文档设置关键字 可选设置 可以设置为一个数组 设置多个关键字
const CFStringRef  kCGPDFContextKeywords;
//设置文档的创建者
const CFStringRef  kCGPDFContextCreator;
//为文档设置所有者密码
const CFStringRef  kCGPDFContextOwnerPassword;
//为文档设置用户密码
const CFStringRef  kCGPDFContextUserPassword;
//设置加密密钥长度
const CFStringRef  kCGPDFContextEncryptionKeyLength;
//设置是否允许绘制
const CFStringRef  kCGPDFContextAllowsPrinting;
//设置是否允许复制
const CFStringRef  kCGPDFContextAllowsCopying;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

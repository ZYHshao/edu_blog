---
title: iOS开发之CoreSpotlight框架的应用
date: 2019-02-25
categories: iOS逻辑初窥
tags: []
---
## iOS开发之CoreSpotlight框架的应用

    CoreSpotlight是iOS提供的一套本地检索推荐功能。开发者可以为自己的应用添加本地索引，用户通过索引中定义的关键字可以搜索并定位到应用程序内的指定功能。

### 一、一个简单的添加索引示例

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    //创建索引属性对象
    CSSearchableItemAttributeSet *set = [[CSSearchableItemAttributeSet alloc] initWithItemContentType:(NSString*)kUTTypeText];
    //设置索引属性
    set.title = @"哈哈哈";
    set.displayName = @"Hello";
    set.alternateNames = @[@"aaa",@"bbb"];
    set.keywords = @[@"333",@"444"];
    set.version = @"1.1";
    set.path = @"path";
    set.thumbnailURL = [NSURL fileURLWithPath:[[NSBundle mainBundle] pathForResource:@"image" ofType:@"png"]];
    //创建索引
    CSSearchableItem *item = [[CSSearchableItem alloc] initWithUniqueIdentifier:@"1111" domainIdentifier:@"huishao" attributeSet:set];
    //添加索引
    [[CSSearchableIndex defaultSearchableIndex] indexSearchableItems:@[item] completionHandler:^(NSError * _Nullable error) {
        if (error) {
            NSLog(@"buildSearchableItem Error:%@",error.localizedDescription);
        }
    }];
}
```

在搜索栏中搜索索引的关键字，标题，名称、路径都可以搜索到当前应用程序。例如：

![](https://oscimg.oschina.net/oscnet/b229f3f87d18985012fa96e5228e787c0ac.jpg)

### 二、CSSearchableIndex索引管理类

      CSSearchableIndex类提供了对索引的操作功能，例如添加索引，查找索引，删除索引等等，解析如下：

```objectivec
//代理对象
@property (weak,nullable) id<CSSearchableIndexDelegate> indexDelegate; 
//获取索引检索是否可用
+ (BOOL)isIndexingAvailable;
//获取默认提供的索引管理对象
+ (instancetype)defaultSearchableIndex;
//创建一个索引管理对象
- (instancetype)initWithName:(NSString *)name;
- (instancetype)initWithName:(NSString *)name protectionClass:(nullable NSFileProtectionType)protectionClass;
//获取索引管理对象中的所有索引
- (void)indexSearchableItems:(NSArray<CSSearchableItem *> *)items completionHandler:(void (^ __nullable)(NSError * __nullable error))completionHandler;
//通过标识符来删除索引
- (void)deleteSearchableItemsWithIdentifiers:(NSArray<NSString *> *)identifiers completionHandler:(void (^ __nullable)(NSError * __nullable error))completionHandler;
//通过域名来删除索引
- (void)deleteSearchableItemsWithDomainIdentifiers:(NSArray<NSString *> *)domainIdentifiers completionHandler:(void (^ __nullable)(NSError * __nullable error))completionHandler;
//删除所有索引
- (void)deleteAllSearchableItemsWithCompletionHandler:(void (^ __nullable)(NSError * __nullable error))completionHandler;
```

### 三、CSSearchableIndexDelegate详解

    CSSearchableIndexDelegate提供了索引查找的相关回调方法，解析如下：

```objectivec
//这个代理重新索引所有可搜索的数据，并且清除任何本地状态（可能该状态已经被持久化），因为索引已经丢失了
- (void)searchableIndex:(CSSearchableIndex *)searchableIndex reindexAllSearchableItemsWithAcknowledgementHandler:(void (^)(void))acknowledgementHandler;
//根据id重新索引所有可搜索的数据
- (void)searchableIndex:(CSSearchableIndex *)searchableIndex reindexSearchableItemsWithIdentifiers:(NSArray <NSString *> *)identifiers
 acknowledgementHandler:(void (^)(void))acknowledgementHandler;
//已经进入节能模式调用的方法
- (void)searchableIndexDidThrottle:(CSSearchableIndex *)searchableIndex;
// 结束节能模式调用的方法
- (void)searchableIndexDidFinishThrottle:(CSSearchableIndex *)searchableIndex;
//用来提供数据
- (nullable NSData *)dataForSearchableIndex:(CSSearchableIndex *)searchableIndex itemIdentifier:(NSString *)itemIdentifier typeIdentifier:(NSString *)typeIdentifier error:(out NSError ** __nullable)outError;
//用来提供文件地址
- (nullable NSURL *)fileURLForSearchableIndex:(CSSearchableIndex *)searchableIndex itemIdentifier:(NSString *)itemIdentifier typeIdentifier:(NSString *)typeIdentifier inPlace:(BOOL)inPlace error:(out NSError ** __nullable)outError;
```

### 四、CSSearchableItem索引类

      CSSearchableItem用来进行索引的定义，解析如下：

```objectivec
//通过设置唯一标识符、域名和属性来定义索引
- (instancetype)initWithUniqueIdentifier:(nullable NSString *)uniqueIdentifier 
                        domainIdentifier:(nullable NSString *)domainIdentifier
                            attributeSet:(CSSearchableItemAttributeSet *)attributeSet;
//唯一标识符
@property (copy) NSString *uniqueIdentifier;
//域名标识符
@property (copy, nullable) NSString *domainIdentifier;
//过期时间
@property (copy, null_resettable) NSDate * expirationDate;
//属性
@property (strong) CSSearchableItemAttributeSet *attributeSet;
```

### 五、CSSearchableItemAttributeSet类

      这个类的主要作用是进行索引信息的配置，CSSearchableItemAttributeSet并没定义太多的属性，CoreSpotlight中提供了这个类的大量Category来补充这个类的功能，解析如下：

```objectivec
//---------------标准---------------------------------
//通过类型创建一个索引信息类
- (instancetype)initWithItemContentType:(nonnull NSString *)itemContentType;
//设置索引的展示名称  可以进行搜索
@property(nullable, copy) NSString *displayName;
//设置可交互的名称数组 可以进行搜索
@property(nullable, copy) NSArray<NSString*> *alternateNames;
//设置索引的完整路径 可以进行搜索
@property(nullable, copy) NSString *path;
//设置索引关联的文件URL
@property(nullable, strong) NSURL *contentURL;
//设置索引缩略图的URL
@property(nullable, strong) NSURL *thumbnailURL;
//设置索引缩略图数据
@property(nullable, copy) NSData *thumbnailData;
//数据最后更改日期
@property(nullable, strong) NSDate *metadataModificationDate;
//内容类型
@property(nullable, copy) NSString *contentType;
//关键字数组
@property(nullable, copy) NSArray<NSString*> *keywords;
//设置标题
@property(nullable, copy) NSString *title;
//设置版本
@property(nullable, copy) NSString *version;
//设置搜索权重 0-100之间
@property(nullable, strong) NSNumber *rankingHint;
//域名标识
@property(nullable, copy) NSString *domainIdentifier;

//------------动作相关--------------------------------
//是否支持拨打电话  必须先设置phoneNumbers属性
@property(nullable, strong) NSNumber *supportsPhoneCall;
//是否支持导航 必须先设置经纬度信息
@property(nullable, strong) NSNumber *supportsNavigation;

//------------容器相关--------------------------------
//设置容器标题
@property(nullable, copy) NSString *containerTitle;
//设置容器显示名
@property(nullable, copy) NSString *containerDisplayName;
//设置容器标识
@property(nullable, copy) NSString *containerIdentifier;
//设置容器顺序
@property(nullable, strong) NSNumber *containerOrder;

//-----------图片相关-------------------------------
//图片高度
@property(nullable, strong) NSNumber *pixelHeight;
//图片高度
@property(nullable, strong) NSNumber *pixelWidth;
//像素总数
@property(nullable, strong) NSNumber *pixelCount;
//颜色空间
@property(nullable, copy) NSString *colorSpace;
//每一帧的bit数
@property(nullable, strong) NSNumber *bitsPerSample;
//拍照时是否开启闪光灯
@property(nullable, strong, getter=isFlashOn) NSNumber *flashOn;
//焦距是否35毫米
@property(nullable, strong, getter=isFocalLength35mm) NSNumber *focalLength35mm;
//设备制造商信息
@property(nullable, copy) NSString *acquisitionMake;
//设备模型信息
@property(nullable, copy) NSString *acquisitionModel;
//... 更多图片信息

//----------媒体相关-------------------------------
//编辑者
@property(nullable, copy) NSArray<NSString*> *editors;
//下载时间
@property(nullable, strong) NSDate *downloadedDate;
//文件描述
@property(nullable, copy) NSString *comment;
//内容版权
@property(nullable, copy) NSString *copyright;
//最后使用时间
@property(nullable, strong) NSDate *lastUsedDate;
//添加时间
@property(nullable, strong) NSDate *addedDate;
//时长
@property(nullable, strong) NSNumber *duration;
//联系人关键字
@property(nullable, copy) NSArray<NSString*> *contactKeywords;
//媒体类型
@property(nullable, copy) NSArray<NSString*> *mediaTypes;
//总比特率
@property(nullable, strong) NSNumber *totalBitRate;
//视频比特币
@property(nullable, strong) NSNumber *videoBitRate;
//音频比特率
@property(nullable, strong) NSNumber *audioBitRate;
//组织信息
@property(nullable, copy) NSArray<NSString*> *organizations;
//创建者
@property(nullable, copy) NSString *role;
//语言信息
@property(nullable, copy) NSArray<NSString*> *languages;
//发布者
@property(nullable, copy) NSArray<NSString*> *publishers;
//地址
@property(nullable, strong) NSURL *URL;
//... 更多媒体信息

//---------------信息相关---------------------------
//网页数据
@property(nullable, copy) NSData *HTMLContentData;
//文本数据
@property(nullable, copy) NSString *textContent;
//作者数据
@property(nullable, copy) NSArray<CSPerson*> *authors;
//邮箱地址
@property(nullable, copy) NSArray<NSString*> *authorEmailAddresses;
@property(nullable, copy) NSArray<NSString*> *emailAddresses;
//电话号码
@property(nullable, copy) NSArray<NSString*> *phoneNumbers;
//... 更多

//--------------地址相关-------------------
//邮编
@property(nullable, copy) NSString *postalCode;
//城市
@property(nullable, copy) NSString *city;
//国家
@property(nullable, copy) NSString *country;
//海拔
@property(nullable, strong) NSNumber *altitude;
//经度
@property(nullable, strong) NSNumber *latitude;
//纬度
@property(nullable, strong) NSNumber *longitude;
//速度
@property(nullable, strong) NSNumber *speed;
//时间戳
@property(nullable, strong) NSDate *timestamp;
//...  更多
```

---
title: iOS9系列专题二——全新的搜索功能api
date: 2015-09-27
categories: iOS9专题
tags: []
---
## 更加智能的搜索方案——iOS9搜索功能新api

### 一、引言

        iOS9中为我们提供了许多新的api，搜索功能的加强无疑是其中比较显眼的一个。首先，我们先设想一下：如果在你的app中定义一种标识符，在siri和搜索中，可以用过这个标识符搜索到你的app，是不是很棒？不，这还差得远，你可以定义任意的数据，使其在搜索和siri中可以快速检索到，这样的搜索功能是不是非常酷？不，还有更cool的，你甚至可以在你的网站中添加一些标志，使apple的爬虫可以检索到，那样，即使用户没有安装你的app，也可以在搜索中获取到相应的信息，这太强大了，对吧。

### 二、3种全新的搜索模式

#### ‍1、NSUserActivity‍

        我们可以在项目中使用相应的函数来添加一些用户的活跃元素，使我们可以在搜索中通过搜索这样的活跃元素展现我们的app。例如：

```
    //创建一个对象，这里的type用于区分搜索的类型
    NSUserActivity *userActivity = [[NSUserActivity alloc] initWithActivityType: @"myapp"];
    //显示的标题
    userActivity.title = @"我的app";
    // 搜索的关键字
    userActivity.keywords = [NSSet setWithArray: @[@"sea",@"rch"]];
    // 支持Search
    userActivity.eligibleForSearch = YES;
    //提交设置
    [userActivity becomeCurrent];
```

在下面的函数中，我们可以处理用户点击搜索后的回调：

```
- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:
{
 NSString *activityType = userActivity.activityType;
    if ([activityType isEqual: @"myapp"]){
        // Handle restoration for values provided in userInfo
        // do something

        return YES;
    }
    return NO;
    //处理回调
}
```

TIP：这种方式添加的关键字搜索，必须创建全局变量，否则无法进行搜索:

![](http://static.oschina.net/uploads/space/2015/0927/191703_JDYc_2340880.png)

#### 2、CoreSpotlight

        CoreSpotlight是一种更加自由的搜索方式，可以通过添加类似item的模型，将app中的数据展示在搜索栏中，CoreSpotlight框架类似提供了一些增、删、改、查的操作，可是使我们自由的进行搜索属性的设置。

##### （1）认识3个类

在iOS9中，新增加了3个类，通过对这三个类的操作与配合，我们可以轻易的在app中添加CoreSpotlight搜索的功能。

 CSSearchableItemAttributeSet：设置类，这个类用于设置搜索标签里的icon，内容，图片等。主要用法如下：

```
//这个类的核心方法只有一个init方法，通过一个类型字符串进行创建，字符串用于在回调中区分
@interface CSSearchableItemAttributeSet : NSObject <NSCopying,NSSecureCoding>
- (instancetype)initWithItemContentType:(nonnull NSString *)itemContentType;
@end
//更多的属性设置在其扩展类中，例如：
@interface CSSearchableItemAttributeSet (CSGeneral)

//展示的名称
@property(nullable, copy) NSString *displayName;

//名称数组
@property(nullable, copy) NSArray<NSString*> *alternateNames;

//完整的路径
@property(nullable, copy) NSString *path;

//链接url
@property(nullable, strong) NSURL *contentURL;

//图片链接的url
@property(nullable, strong) NSURL *thumbnailURL;

//设置图片数据
@property(nullable, copy) NSData *thumbnailData;

//设置一个标识符
@property(nullable, copy) NSString *relatedUniqueIdentifier;

@property(nullable, strong) NSDate *metadataModificationDate;

//内容类型
@property(nullable, copy) NSString *contentType;

@property(nullable, copy) NSArray<NSString*> *contentTypeTree;

//搜索的关键字数组
@property(nullable, copy) NSArray<NSString*> *keywords;

//标题信息
@property(nullable, copy) NSString *title;

@end
```

 CSSearchableItem：搜索标签类，通过这个类，来创建响应的搜索标签。主要内容如下：

```
//这个类主要用于创建搜索的标签
@interface CSSearchableItem : NSObject <NSSecureCoding, NSCopying>
//init方法
- (instancetype)initWithUniqueIdentifier:(nullable NSString *)uniqueIdentifier //Can be null, one will be generated
                        domainIdentifier:(nullable NSString *)domainIdentifier
                            attributeSet:(CSSearchableItemAttributeSet *)attributeSet;
//相应 的属性
@property (copy) NSString *uniqueIdentifier;

@property (copy, nullable) NSString *domainIdentifier;

@property (copy, null_resettable) NSDate * expirationDate;

@property (strong) CSSearchableItemAttributeSet *attributeSet;

@end
```

CSSearchableIndex：这个类，我个人理解，类似一个manager的作用，通过它对标签进行增、删、改、查等操作：

```
@interface CSSearchableIndex : NSObject

@property (weak,nullable) id<CSSearchableIndexDelegate> indexDelegate;

//判断设备是否支持
+ (BOOL)isIndexingAvailable;
//取系统的searchIndex管理者
+ (instancetype)defaultSearchableIndex;
//一般情况下，我们不需要重新创建对象
- (instancetype)initWithName:(NSString *)name;
- (instancetype)initWithName:(NSString *)name protectionClass:(nullable NSString *)protectionClass;

//设置索引标签
- (void)indexSearchableItems:(NSArray<CSSearchableItem *> *)items completionHandler:(void (^ __nullable)(NSError * __nullable error))completionHandler;

//删除指定id索引标签
- (void)deleteSearchableItemsWithIdentifiers:(NSArray<NSString *> *)identifiers completionHandler:(void (^ __nullable)(NSError * __nullable error))completionHandler;

- (void)deleteSearchableItemsWithDomainIdentifiers:(NSArray<NSString *> *)domainIdentifiers completionHandler:(void (^ __nullable)(NSError * __nullable error))completionHandler;

//删除所有索引标签
- (void)deleteAllSearchableItemsWithCompletionHandler:(void (^ __nullable)(NSError * __nullable error))completionHandler;

@end
```

##### （2）一个小例子

        下面，我们通过一个小例子来应用下CoreSpotlight的搜索功能。

首先，需要在项目中导入如下库：

![](http://static.oschina.net/uploads/space/2015/0927/190756_ZO8R_2340880.png)

实现如下代码：

```
 //进行标签设置
 CSSearchableItemAttributeSet * itemSet = [[CSSearchableItemAttributeSet alloc]initWithItemContentType:@"myApp"];
    itemSet.title = @"我的APP";
    itemSet.keywords = @[@"haha",@"123"];
    itemSet.contentDescription = @"这是搜索到得内容";
    itemSet.thumbnailData = UIImagePNGRepresentation([UIImage imageNamed:@"Icon-114.png"]);
    
    CSSearchableItem * item = [[CSSearchableItem alloc]initWithUniqueIdentifier:@"1" domainIdentifier:@"1" attributeSet:itemSet];
    
    [[CSSearchableIndex defaultSearchableIndex]indexSearchableItems:@[item] completionHandler:nil];
```

我们在搜索中输入haha或者123效果如下：

![](http://static.oschina.net/uploads/space/2015/0927/191240_R2bZ_2340880.png)

CoreSpotlight的搜索回调和NSUserActivaty一样，只是区分id的方式有所不同：

```
- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:
{
    NSString *activityType = userActivity.activityType;
    //先取CSSearchableItemActionType
    if ([activityType isEqual: CSSearchableItemActionType]) {
        NSString *uniqueIdentifier = [userActivity.userInfo                                                 objectForKey:CSSearchableItemActivityIdentifier];
        // do something
        return YES;
    }
    return NO;
}
```

#### 3、Web Markup

        这个功能与我们app开发关系不大，但是对我app的推广却至关重要，这项技术可以让我们的app关联一个网站，apple通过爬虫来获取我们规定的一些标签值，无论用户是否安装了app，在搜索时，都可以展示出相关信息，因为这项功能主要关联前端技术，需要了解的可以参看：[App Search Programming Guide](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/AppSearch/WebContent.html#//apple_ref/doc/uid/TP40016308-CH8)。

### 三、结语

        在我参考的许多相关文章中，都一致建议，iOS9的搜索功能固然强大，然而滥用会造成垃圾信息的泛滥，这样的结果一定会适得其反，作为开发者，我们需要将最合适，最简洁的信息推送到用户的面前。另外，文章有疏漏和错误之处，欢迎指正。

欢迎转载 请注明出处

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

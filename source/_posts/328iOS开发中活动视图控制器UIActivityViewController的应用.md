---
title: iOS开发中活动视图控制器UIActivityViewController的应用
date: 2017-05-15
categories: iOS之UI控件
tags: []
---
## iOS开发中活动视图控制器UIActivityViewController的应用

    在iOS开发中，UIActivityViewController常用来弹出分享面板，其实除了用来社会化分享，UIActivityViewController还有一大应用是用来进行自定义行为。先看如下示例代码：

```objectivec
    //活动内容
    NSString * content = @"活动的内容";
    //活动的url
    NSURL * url = [NSURL URLWithString:@"https://www.baidu.com"];
    //活动的图片
    UIImage * image = [UIImage imageNamed:@"ios"];
    UIActivityViewController * con = [[UIActivityViewController alloc]initWithActivityItems:@[content,url,image] applicationActivities:nil];
    //活动行为结束后回调的block
    con.completionWithItemsHandler = ^(UIActivityType activityType, BOOL completed, NSArray * returnedItems, NSError * __nullable activityError){
        NSLog(@"%@\n%@",activityType,returnedItems);
    };
    [self presentViewController:con animated:YES completion:nil];
```

活动面板如下图：

![](https://static.oschina.net/uploads/space/2017/0515/171456_hyHg_2340880.png)

需要注意，活动面板可以分为3个部分，最上面为AirDrop传输功能，中间为分享相关功能，最下面为数据处理功能。UIActivityViewController继承自UIViewController，类解析如下：

```objectivec
//初始化方法
- (instancetype)init;
- (instancetype)initWithNibName:(nullable NSString *)nibNameOrNil bundle:(nullable NSBundle *)nibBundleOrNil;
- (nullable instancetype)initWithCoder:(NSCoder *)aDecoder;
/*
activityItems参数用来设置活动数据数组，其中可以是任意类型的对象，但是只有可以处理这些数据的行为会被展示出来
applicationActivitie参数可以设置自定义的操作行为，后面会介绍
*/
- (instancetype)initWithActivityItems:(NSArray *)activityItems applicationActivities:(nullable NSArray<__kindof UIActivity *> *)applicationActivitie;
/*
活动行为结束后执行的回调block，其中参数如下：
typedef void (^UIActivityViewControllerCompletionWithItemsHandler)(UIActivityType __nullable activityType, BOOL completed, NSArray * __nullable returnedItems, NSError * __nullable activityError);
activityType:活动的类型
completed:活动是否完成
returnItems:扩展程序返回的数据
*/
@property(nullable, nonatomic, copy) UIActivityViewControllerCompletionWithItemsHandler completionWithItemsHandler;
//这个参数可以设置不被显示的活动类型
@property(nullable, nonatomic, copy) NSArray<UIActivityType> *excludedActivityTypes;

//下面这些方法在iOS8后被弃用 在iOS6-iOS8之前可用
//设置活动行为结束后回调的block
/*
typedef void (^UIActivityViewControllerCompletionHandler)(UIActivityType __nullable activityType, BOOL completed);
*/
@property(nullable, nonatomic, copy) UIActivityViewControllerCompletionHandler completionHandler;
```

上面初始化方法中有提到activityItems这个参数，系统提供的一些分享与活动行为可支持的数据类型列表如下：

![](https://static.oschina.net/uploads/space/2017/0515/172647_LqxU_2340880.png)

    系统提供了一些活动类型，例如分享到微博、脸书、进行添加提示、发送信息等，系统提供的活动类型列举如下(UIActivityType实际上就是NSString*)：

```objectivec
UIActivityType const UIActivityTypePostToFacebook;//脸书
UIActivityType const UIActivityTypePostToTwitter;//推特
UIActivityType const UIActivityTypePostToWeibo;  // 微博
UIActivityType const UIActivityTypeMessage;//发送信息
UIActivityType const UIActivityTypeMail;//发送邮件
UIActivityType const UIActivityTypePrint;//打印
UIActivityType const UIActivityTypeCopyToPasteboard;//复制
UIActivityType const UIActivityTypeAssignToContact;//关联到联系人
UIActivityType const UIActivityTypeSaveToCameraRoll;//存照片
UIActivityType const UIActivityTypeAddToReadingList;//添加到提醒列表
UIActivityType const UIActivityTypePostToFlickr;//Flickr
UIActivityType const UIActivityTypePostToVimeo;//Vimeo
UIActivityType const UIActivityTypePostToTencentWeibo;//腾讯微博
UIActivityType const UIActivityTypeAirDrop;//AirDrop
UIActivityType const UIActivityTypeOpenInIBooks;//在IBooks中打开
```

    自定义活动行为需要创建继承于UIActivity类的子类，示例如下：

```objectivec
#import "CustomActivity.h"

@implementation CustomActivity
//设置活动类别
+(UIActivityCategory)activityCategory{
    return UIActivityCategoryShare;
}
//设置活动类型
-(UIActivityType)activityType{
    return @"Custom";
}
//设置活动标题
-(NSString *)activityTitle{
    return @"title";
}
//设置活动图标
-(UIImage *)activityImage{
    return [UIImage imageNamed:@"ios"];
}
//设置是否可以对指定的数据进行响应
-(BOOL)canPerformWithActivityItems:(NSArray *)activityItems{
    NSLog(@"%@",activityItems);
    return YES;
}
//为要响应的活动进行准备工作
- (void)prepareWithActivityItems:(NSArray *)activityItems{
    
}
//响应互动
-(void)performActivity{
    NSLog(@"=========");
    //活动处理完成后 必须调用activityDidFinish函数
    [self activityDidFinish:YES];
}

@end
```

用自定义的活动对UIActivityViewController进行初始化：

```objectivec
    NSString * content = @"活动的内容";
    NSURL * url = [NSURL URLWithString:@"https://www.baidu.com"];
    UIImage * image = [UIImage imageNamed:@"ios"];
    CustomActivity * activity = [[CustomActivity alloc]init];
    UIActivityViewController * con = [[UIActivityViewController alloc]initWithActivityItems:@[content,url,image] applicationActivities:@[activity]];
    con.completionWithItemsHandler = ^(UIActivityType activityType, BOOL completed, NSArray * returnedItems, NSError * __nullable activityError){
        NSLog(@"%@\n%@",activityType,returnedItems);
    };
    [self presentViewController:con animated:YES completion:nil];
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2017/0515/173557_CA0h_2340880.png)

    UIActivity类解析如下：

```objectivec
//子类实现，设置自定义活动的类别
/*
typedef NS_ENUM(NSInteger, UIActivityCategory) {
    UIActivityCategoryAction,//行为类别 显示在活动面板下面
    UIActivityCategoryShare,//分享类别，显示在活动面板中间
};
*/
+ (UIActivityCategory)activityCategory;
//子类实现 设置自定义活动的类型 返回字符串
- (nullable UIActivityType)activityType;
//子类实现 设置自定义活动的标题 返回字符串 
- (nullable NSString *)activityTitle;
//子类实现 设置自定义活动的图标 UIImage  
- (nullable UIImage *)activityImage; 
//子类实现 activityItems为活动数据数组 返回布尔值决定此活动是否可以响应这些数据
- (BOOL)canPerformWithActivityItems:(NSArray *)activityItems;
//子类实现 如果上面的方法返回YES，会接着执行这个方法，开发者可以做些活动处理的准备
- (void)prepareWithActivityItems:(NSArray *)activityItems;
//子类实现 返回一个视图控制器作为处理活动的模态视图 活动处理完成后需要调用activityDidFinish方法 
- (nullable UIViewController *)activityViewController; 
//子类实现 如果子类没有实现上一个方法 或者返回nil，则会执行这个方法来处理活动 活动处理完成后需要调用activityDidFinish方法
- (void)performActivity; 
//活动处理完成后需要调用这个方法 之后会通知UIActivityViewController执行活动完成后的回调block
- (void)activityDidFinish:(BOOL)completed;
```

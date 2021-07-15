---
title: iOS使用VOIP与CallKit实现体验优质的网络通讯功能
date: 2018-04-24
categories: iOS逻辑初窥
tags: []
---
## iOS使用VOIP与CallKit实现体验优质的网络通讯功能

    VOIP是Apple提供给开发者的网络电话功能接口。简单来说，其可以让你的应用程序在完全杀死的情况下被服务端唤醒。CallKit是iOS10引入的新框架，使用它可以让你的应用程序调用系统的通话和通话记录界面。试想一下，用户可以在锁屏，应用被杀死，应用在后台等情况下收到通讯请求并且弹出系统的通话界面进行交互是多么酷的一件事。

### 一、创建VOIP推送证书

    VOIP说是一种网络电话服务，其实质是一种特殊的长连接，使用它每个网络电话类APP不需要自己单独进行保活维护，在进行通话请求时，只需要发送一条VOIP推送，VOIP推送会将应用程序拉起，之后由应用程序处理通讯逻辑。VOIP也是Push的一种，只是其是一种特殊的Push，普通的Push当应用被杀死后可以收到，但是用户点击Push消息前应用程序是不会被激活的，VOIP则不然，可以直接激活应用。

    VOIP推送证书的创建方式与普通推送证书的创建方式基本一致，首先需要生成certSigningRequest文件，打开钥匙串应用：

![](https://static.oschina.net/uploads/space/2018/0423/141531_jLCt_2340880.png)

在证书助理栏选择从证书颁发机构申请证书：

![](https://static.oschina.net/uploads/space/2018/0423/141604_ashA_2340880.png)

填写相关资料后，将生成的文件保存：

![](https://static.oschina.net/uploads/space/2018/0423/141636_splB_2340880.png)

在Apple开发者中心创建新的证书，证书类型选择生产环境的VOIP服务证书：

![](https://static.oschina.net/uploads/space/2018/0423/141942_vfAz_2340880.png)

需要注意，普通的推送分开发环境和生产环境，VOIP证书不进行区分，生产环境和开发环境是通用的。之后选择一个AppID并且上传前面生成的certSigningRequest文件来完成VOIP证书的创建。

    创建完成后，在证书列表可以看到多了一个VOIP服务证书，可以加载此证书进行VOIP推送。

![](https://static.oschina.net/uploads/space/2018/0423/142407_sOzY_2340880.png)

### 二、PushKit详析

    我们知道，客户端若想要接收普通的Push消息，是需要注册Token，通过Token来进行个推的。VOIP推送也是一样的，只是这类推送需要使用PushKit框架。

    首先需要用到PKPushRegistey类，这个类进行推送的相关配置和Token的申请：

```objectivec
@interface PKPushRegistry : NSObject
//代理对象
@property (readwrite,weak,nullable) id<PKPushRegistryDelegate> delegate;
//目标推送类型
/*
PK_EXPORT PKPushType const PKPushTypeVoIP NS_AVAILABLE_IOS(8_0);//VOIP推送
PK_EXPORT PKPushType const PKPushTypeComplication NS_AVAILABLE_IOS(9_0);//Watch更新
PK_EXPORT PKPushType const PKPushTypeFileProvider NS_AVAILABLE_IOS(11_0);//文件传输
*/
@property (readwrite,copy,nullable) NSSet<PKPushType> *desiredPushTypes;
//获取本地缓存的Token  申请Token执行回调后 这个方法可以直接获取缓存
- (nullable NSData *)pushTokenForType:(PKPushType)type;
//初始化，并设置工作线程
- (instancetype)initWithQueue:(nullable dispatch_queue_t)queue NS_DESIGNATED_INITIALIZER;
- (instancetype)init NS_UNAVAILABLE;
@end
```

PKPushRegistryDelegate相关函数意义如下：

```objectivec
//申请Token更新后回调
/*
PKPushCredentials是证书对象，其中属性如下：
@property (readonly,copy) PKPushType type;//推送类型
@property (readonly,copy) NSData *token; //Token
*/
- (void)pushRegistry:(PKPushRegistry *)registry didUpdatePushCredentials:(PKPushCredentials *)pushCredentials forType:(PKPushType)type;
//收到推送后执行的回调
/*
PKPushPayload为推送信息 其中属性如下：
@property (readonly,copy) PKPushType type; //推送类型
@property (readonly,copy) NSDictionary *dictionaryPayload; //服务端发来的信息
*/
- (void)pushRegistry:(PKPushRegistry *)registry didReceiveIncomingPushWithPayload:(PKPushPayload *)payload forType:(PKPushType)type NS_DEPRECATED_IOS(8_0, 11_0);
//作用同上，最后的block需要在逻辑处理完成后主动回调
- (void)pushRegistry:(PKPushRegistry *)registry didReceiveIncomingPushWithPayload:(PKPushPayload *)payload forType:(PKPushType)type withCompletionHandler:(void(^)(void))completion NS_AVAILABLE_IOS(11_0);
//Token失效的回调
- (void)pushRegistry:(PKPushRegistry *)registry didInvalidatePushTokenForType:(PKPushType)type;
```

如果配置成功，在收到VOIP推送时，无论应用程序是否活跃，都会执行代理函数，我们便可以在其中进行逻辑处理。

### 三、关于CallKit框架

    CallKit框架是iOS10后系统提供的一套网络电话UI和交互相关接口，应用程序可以调用系统的电话界面来进行逻辑传递。下图比较形象的表达了应用程序与CallKit的关系：

![](https://static.oschina.net/uploads/space/2018/0423/150352_MOMA_2340880.png)

以收到网络电话为例，如果应用程序在前台，客户端可以直接处理通讯逻辑，如果应用程序不在前台，服务端可以发送一条VOIP推送唤醒APP，之后APP通知CallKit框架来唤起系统的通讯界面。CXProvider类主要负责系统服务于APP之间的交互。例如可以通过它来更新通话界面，显示通话的来自方，当用户点击通话界面的某些按钮后，也通过它来通知APP做逻辑处理。

    需要注意，上图在CallKit和System之间有两个双向的白色箭头，这描述了CallKit和系统交互的四个方向。

    首先，App想要和系统交互，例如接收到VOIP通知后弹出通话界面，需要使用CXProvider通过CXCallUpdate来进行控制。如下图：

![](https://static.oschina.net/uploads/space/2018/0423/222253_IDKX_2340880.png)

     之后系统会将一些用户操作通过CSAction传递会APP，如下：

![](https://static.oschina.net/uploads/space/2018/0423/222346_nXGZ_2340880.png)

    APP中进行的操作如果需要通知系统，需要使用CXCallController通过CXTransaction传递。例如App内的通讯需要添加到系统的历史通话列表。如下：

![](https://static.oschina.net/uploads/space/2018/0423/223200_XVBF_2340880.png)

#### 1.先来看CXProvider类

    CXProvider类用来对系统通话界面进行一些配置操作，并处理回调逻辑，解析如下：

```objectivec
//初始化方法 使用CXProviderConfiguration来进行配置 后面会介绍
- (instancetype)initWithConfiguration:(CXProviderConfiguration *)configuration NS_DESIGNATED_INITIALIZER;
- (instancetype)init NS_UNAVAILABLE;
//设置代理与代理函数所工作的线程
- (void)setDelegate:(nullable id<CXProviderDelegate>)delegate queue:(nullable dispatch_queue_t)queue;
//向系统发起一个新的通话请求
/*
UUID为此通话请求的标识 可以使用它来关闭通话
update设置界面的更新参数
*/
- (void)reportNewIncomingCallWithUUID:(NSUUID *)UUID update:(CXCallUpdate *)update completion:(void (^)(NSError *_Nullable error))completion;
//结束某个通话 使用上面的UUID作为标识
/*
//通话结束的原因设置
typedef NS_ENUM(NSInteger, CXCallEndedReason) {
    CXCallEndedReasonFailed = 1, // 通话服务失败
    CXCallEndedReasonRemoteEnded = 2, // 对方挂断
    CXCallEndedReasonUnanswered = 3, // 超时 对方为接听
    CXCallEndedReasonAnsweredElsewhere = 4, // 通话在其他设备接听
    CXCallEndedReasonDeclinedElsewhere = 5, // 通话在其他设备拒绝
} API_AVAILABLE(ios(10.0));
*/
- (void)reportCallWithUUID:(NSUUID *)UUID endedAtDate:(nullable NSDate *)dateEnded reason:(CXCallEndedReason)endedReason;
//更新通话对方的信息
- (void)reportCallWithUUID:(NSUUID *)UUID updated:(CXCallUpdate *)update;
//调用这个函数来进行通话呼出开始
- (void)reportOutgoingCallWithUUID:(NSUUID *)UUID startedConnectingAtDate:(nullable NSDate *)dateStartedConnecting;
//调用这个函数来进行通话呼出连接完成
- (void)reportOutgoingCallWithUUID:(NSUUID *)UUID connectedAtDate:(nullable NSDate *)dateConnected;
//配置对象
@property (nonatomic, readwrite, copy) CXProviderConfiguration *configuration;
//调用此函数来将通话失效
- (void)invalidate;
//所有未完成的事物
@property (nonatomic, readonly, copy) NSArray<CXTransaction *> *pendingTransactions;
- (NSArray<__kindof CXCallAction *> *)pendingCallActionsOfClass:(Class)callActionClass withCallUUID:(NSUUID *)callUUID;
```

#### 2.在看CXProviderConfiguration类

    这个类用来进行Provider的配置，例如设置通讯服务名称，铃声，图标，是否支持组等。解析如下：

```objectivec
//设置服务名称
@property (nonatomic, readonly, copy) NSString *localizedName;
//设置铃声  资源必须在 app的 bundle里
@property (nonatomic, strong, nullable) NSString *ringtoneSound;
//设置应用图标
@property (nonatomic, copy, nullable) NSData *iconTemplateImageData;
//设置最大支持的组数 默认为2
@property (nonatomic) NSUInteger maximumCallGroups;
//设置最大的每组人数 默认为5
@property (nonatomic) NSUInteger maximumCallsPerCallGroup;
//设置是否将通话记录保存进最近通话列表
@property (nonatomic) BOOL includesCallsInRecents;
//设置是否支持视频通话
@property (nonatomic) BOOL supportsVideo;
//设置支持的操作类型
@property (nonatomic, copy) NSSet<NSNumber *> *supportedHandleTypes;
```

当App接收到来电VOIP通知时，可以使用CXCallUpdate来更新状态唤出通话界面。

#### 3.CXCallUpdate类

```objectivec
//远程操作对象 如果是接收方 则此为呼叫方 如果是呼叫方 则此为接收方
@property (nonatomic, copy, nullable) CXHandle *remoteHandle;
//名称
@property (nonatomic, copy, nullable) NSString *localizedCallerName;
//是否支持暂时挂起
@property (nonatomic) BOOL supportsHolding;
//是否支持组
@property (nonatomic) BOOL supportsGrouping;
//是否支持非组通话
@property (nonatomic) BOOL supportsUngrouping;
//是否支持DTMF
@property (nonatomic) BOOL supportsDTMF;
//是否包含视频
@property (nonatomic) BOOL hasVideo;
```

CXHandle中来定义操作的类型，解析如下：

```objectivec
//类型
/*
typedef NS_ENUM(NSInteger, CXHandleType) {
    CXHandleTypeGeneric = 1,//通用
    CXHandleTypePhoneNumber = 2,//电话
    CXHandleTypeEmailAddress = 3,//邮箱地址
} API_AVAILABLE(ios(10.0));
*/
@property (nonatomic, readonly) CXHandleType type;
//值
@property (nonatomic, readonly, copy) NSString *value;

- (instancetype)initWithType:(CXHandleType)type value:(NSString *)value NS_DESIGNATED_INITIALIZER;
```

下面给出了简单的当被叫收到VOIP后调起通话界面的代码：

```objectivec
CXCallUpdate * callUpdate = [[CXCallUpdate alloc]init];
callUpdate.supportsGrouping = YES;
callUpdate.supportsDTMF = YES;
callUpdate.hasVideo = YES;
callUpdate.supportsHolding = YES;
[callUpdate setLocalizedCallerName:nickName];
CXHandle * handle = [[CXHandle alloc]initWithType:CXHandleTypePhoneNumber value:from];
callUpdate.remoteHandle = handle;
 [[self shareInstance].callProvider reportNewIncomingCallWithUUID:[self shareInstance].uuid update:callUpdate completion:^(NSError * _Nullable error) {
     LOG(@"吊起界面");
}];
```

锁屏和应用程序在后台的效果分别如下所示：

![](https://static.oschina.net/uploads/space/2018/0423/215747_SILb_2340880.jpeg)               ![](https://static.oschina.net/uploads/space/2018/0423/215800_wlPB_2340880.jpeg)

![](https://static.oschina.net/uploads/space/2018/0423/215816_JJGo_2340880.jpeg)

#### 4.CXProviderDelegate相关函数解析

    CXProviderDelegate中的相关函数用来处理系统通话界面的某些操作回调给应用程序。

```objectivec
//当接收到呼叫重置时 调用的函数，这个函数必须被实现，其不需做任何逻辑，只用来重置状态
- (void)providerDidReset:(CXProvider *)provider;
//呼叫开始时回调 
- (void)providerDidBegin:(CXProvider *)provider;
//音频会话激活状态的回调
- (void)provider:(CXProvider *)provider didActivateAudioSession:(AVAudioSession *)audioSession;
//音频会话停用的回调
- (void)provider:(CXProvider *)provider didDeactivateAudioSession:(AVAudioSession *)audioSession;
//行为超时的回调 
- (void)provider:(CXProvider *)provider timedOutPerformingAction:(CXAction *)action;
//有事务被提交时调用 
//如果返回YES 则表示事务被捕获处理 后面的回调都不会调用 如果返回NO 则表示事务不被捕获，会回调后面的函数
- (BOOL)provider:(CXProvider *)provider executeTransaction:(CXTransaction *)transaction;
//点击开始按钮的回调
- (void)provider:(CXProvider *)provider performStartCallAction:(CXStartCallAction *)action;
//点击接听按钮的回调
- (void)provider:(CXProvider *)provider performAnswerCallAction:(CXAnswerCallAction *)action;
//点击结束按钮的回调
- (void)provider:(CXProvider *)provider performEndCallAction:(CXEndCallAction *)action;
//点击保持通话按钮的回调
- (void)provider:(CXProvider *)provider performSetHeldCallAction:(CXSetHeldCallAction *)action;
//点击静音按钮的回调
- (void)provider:(CXProvider *)provider performSetMutedCallAction:(CXSetMutedCallAction *)action;
//点击组按钮的回调
- (void)provider:(CXProvider *)provider performSetGroupCallAction:(CXSetGroupCallAction *)action;
//DTMF功能回调
- (void)provider:(CXProvider *)provider performPlayDTMFCallAction:(CXPlayDTMFCallAction *)action;
```

需要注意，上面的最后几个回调中CXStartCallAction都会提供一个fullfill的函数，当处理完成回调逻辑后，开发者需要手动调用此函数来通知系统。同样，其中还有一个fail和timeout函数，调用它要通知系统此行为执行失败和超时。

#### 5.CXCallController解析

    当用户在应用程序内部进行的通讯操作时，可以使用这个类来通知系统。

```objectivec
//初始化方法
- (instancetype)init;
- (instancetype)initWithQueue:(dispatch_queue_t)queue;
//通讯监听
@property (nonatomic, readonly, strong) CXCallObserver *callObserver;
//发起一个事务请求 CXProvider之后会接收到请求 进行逻辑
- (void)requestTransaction:(CXTransaction *)transaction completion:(void (^)(NSError *_Nullable error))completion;
//通过行为发起事务
- (void)requestTransactionWithActions:(NSArray<CXAction *> *)actions completion:(void (^)(NSError *_Nullable error))completion API_AVAILABLE(ios(11.0));
- (void)requestTransactionWithAction:(CXAction *)action completion:(void (^)(NSError *_Nullable error))completion API_AVAILABLE(ios(11.0));
```

#### 6.CXTransaction类

    CXTransaction是封装了行为的事务。

```objectivec
//唯一 ID
@property (nonatomic, readonly, copy) NSUUID *UUID;
//行为完成后的回调
@property (nonatomic, readonly, assign, getter=isComplete) BOOL complete;
//行为数组
@property (nonatomic, readonly, copy) NSArray<__kindof CXAction *> *actions;
//初始化函数
- (instancetype)initWithActions:(NSArray<CXAction *> *)actions;
- (instancetype)initWithAction:(CXAction *)action;
//添加行为
- (void)addAction:(CXAction *)action;
```

### 四、进行来电拦截与号码识别

    上面我们介绍了使用CallKit框架来实现的通讯功能，有通讯功能就难免需要进行联系人识别与黑名单。CallKit框架中还有一部分内容可以结合Call Directory Extension来实现号码拦截与识别。

    首先创建一个扩展Target，选择Call Directory Extension：

![](https://static.oschina.net/uploads/space/2018/0424/134623_xk7W_2340880.png)

创建好Target工程后，其实需要的核心代码Xcode已经帮我们都生成。

    第一步，需要在主APP中进行号码服务的验证和更新，

```objectivec
 _manager = [[CXCallDirectoryManager alloc]init];
    [_manager getEnabledStatusForExtensionWithIdentifier:@"jaki.CallKitTest.Ex" completionHandler:^(CXCallDirectoryEnabledStatus enabledStatus, NSError * _Nullable error) {
        if (enabledStatus==CXCallDirectoryEnabledStatusEnabled) {
            NSLog(@"允许");
        }else{
            NSLog(@"请开启");
        }
    }];
    [_manager reloadExtensionWithIdentifier:@"jaki.CallKitTest.Ex" completionHandler:^(NSError * _Nullable error) {
        NSLog(@"刷新配置");
    }];
    
```

通常情况下，当用户在主APP中进行添加联系人，登录，切换账户等操作后，需要通知扩展程序进行号码库的更新，当然，一般在号码库更新时需要从主APP传递数据给扩展，我们可以通过Group来实现，这里不再展开。

    工程运行后，会在用户的“设置->电话->来电组织与身份识别”项目中看到扩展程序：

![](https://static.oschina.net/uploads/space/2018/0424/141037_h53V_2340880.jpeg)

当用户打开此服务或者调用上面的reloadExtension时，会从执行扩展程序的相关方法来重新加载号码库。需要注意，reloadExtension函数中的id参数为扩展项目的bundleID，不是主项目的。

    在扩展工程的info.plist文件中，默认配置好了处理来电的操作类，如果要自定义，需要开发者手动修改：

![](https://static.oschina.net/uploads/space/2018/0424/142700_MYdl_2340880.png)

默认的CallDirectoryHandler类为来电拦截与身份识别的操作类，其集成自CXCallDirectoryProvider类，当收到加载号码库的请求时，会执行下面的函数：

```objectivec
- (void)beginRequestWithExtensionContext:(CXCallDirectoryExtensionContext *)context {
    context.delegate = self;
    //是否支持增量更新
    if (context.isIncremental) {
        [self addOrRemoveIncrementalBlockingPhoneNumbersToContext:context];

        [self addOrRemoveIncrementalIdentificationPhoneNumbersToContext:context];
    } else {
        [self addAllBlockingPhoneNumbersToContext:context];

        [self addAllIdentificationPhoneNumbersToContext:context];
    }
    //完成更新操作
    [context completeRequestWithCompletionHandler:nil];
}
```

上面是Xcode默认提供的实现，十分优雅，在iOS11后，号码库的更新支持增量，所以这里进行的区分。

    CXCallDirectoryExtensionContext是一个操作上下文，通过它可以像号码库中添加删除数据。解析如下：

```objectivec
//是否支持增量更新
@property (nonatomic, readonly, getter=isIncremental) BOOL incremental API_AVAILABLE(ios(11.0));
//添加一个黑名单号码
- (void)addBlockingEntryWithNextSequentialPhoneNumber:(CXCallDirectoryPhoneNumber)phoneNumber;
//移除一个黑名单号码
- (void)removeBlockingEntryWithPhoneNumber:(CXCallDirectoryPhoneNumber)phoneNumber API_AVAILABLE(ios(11.0));
//移除所有的黑名单号码
- (void)removeAllBlockingEntries API_AVAILABLE(ios(11.0));
//添加一个身份识别
- (void)addIdentificationEntryWithNextSequentialPhoneNumber:(CXCallDirectoryPhoneNumber)phoneNumber label:(NSString *)label;
//移除一个身份识别
- (void)removeIdentificationEntryWithPhoneNumber:(CXCallDirectoryPhoneNumber)phoneNumber API_AVAILABLE(ios(11.0));
//移除所有身份识别
- (void)removeAllIdentificationEntries API_AVAILABLE(ios(11.0));
//完成操作后 需要手动调用此函数
- (void)completeRequestWithCompletionHandler:(nullable void (^)(BOOL expired))completion;
```

添加了黑名单后，用户将收不到此号码的电话，同样，设置了身份识别后，当用户播出前，会显示设置的身份信息(需要注意，大陆号码需要前面带86)，如下：

![](https://static.oschina.net/uploads/space/2018/0424/150928_C7Dl_2340880.jpeg)

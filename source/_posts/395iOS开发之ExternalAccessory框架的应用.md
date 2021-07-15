---
title: iOS开发之ExternalAccessory框架的应用
date: 2019-06-27
categories: iOS逻辑初窥
tags: []
---
## iOS开发之ExternalAccessory框架的应用

      ExternalAccessory框架用来对外设进行管理，iOS外设通常是通过MFI认证的外部设备，可以通过蓝牙进行连接，也可以使用lighting端口进行连接。

      EAAccessoryManager类用来对外设进行管理，其中属性方法如下：

```objectivec
@interface EAAccessoryManager : NSObject
// 获取单例对象
+ (EAAccessoryManager *)sharedAccessoryManager;
// 打开蓝牙外设搜索列表
- (void)showBluetoothAccessoryPickerWithNameFilter:(nullable NSPredicate *)predicate completion:(nullable EABluetoothAccessoryPickerCompletion)completion;
// 注册本地通知 当外设连接状态变化后会触发通知
- (void)registerForLocalNotifications;
// 取消通知的注册
- (void)unregisterForLocalNotifications;
// 所有连接的外设列表
@property (nonatomic, readonly) NSArray<EAAccessory *> *connectedAccessories;
@end
```

如果注册了本地通知，则可以监听下面两个通知：

```objectivec
EAAccessoryDidConnectNotification     // 外设已经连接的通知
EAAccessoryDidDisconnectNotification  // 外设断开连接的通知
```

    EAAccessory是外设对象，其中定义了外设的相关信息，如下：

```objectivec
@interface EAAccessory : NSObject
// 是否已经连接
@property(nonatomic, readonly, getter=isConnected) BOOL connected;
// 连接ID
@property(nonatomic, readonly) NSUInteger connectionID;
// 制造商
@property(nonatomic, readonly) NSString *manufacturer; 
// 外设名称
@property(nonatomic, readonly) NSString *name;
// 模式编码
@property(nonatomic, readonly) NSString *modelNumber;
// 序列号
@property(nonatomic, readonly) NSString *serialNumber;
// 固件版本
@property(nonatomic, readonly) NSString *firmwareRevision;
// 硬件版本
@property(nonatomic, readonly) NSString *hardwareRevision;
// 接口类型
@property(nonatomic, readonly) NSString *dockType;
// 协议列表
@property(nonatomic, readonly) NSArray<NSString *> *protocolStrings;
// 代理
@property(nonatomic, assign, nullable) id<EAAccessoryDelegate> delegate;
@end

@protocol EAAccessoryDelegate <NSObject>
@optional
// 外设断开连接时调用
- (void)accessoryDidDisconnect:(EAAccessory *)accessory;
@end
```

      需要注意，与外设进行通讯需要指定对应的协议，首先，需要在iOS应用的info.plist文件中添加如下键来指定此应用要交互的外设协议：

![](https://oscimg.oschina.net/oscnet/15939fa7cecf10fb618f3a12dc3d4341a90.jpg)

具体的外设协议需要查看外设的说明文档。

EASession类用来进行外设交互，解析如下：

```objectivec
@interface EASession : NSObject
// 指定外设和协议来创建会话对象
- (nullable instancetype)initWithAccessory:(EAAccessory *)accessory forProtocol:(NSString *)protocolString;
// 外设对象
@property (nonatomic, readonly, nullable) EAAccessory *accessory;
// 协议
@property (nonatomic, readonly, nullable) NSString *protocolString;
// 输入流 用来向外设发送数据
@property (nonatomic, readonly, nullable) NSInputStream *inputStream;
// 输出流 用来接收外设发送的数据
@property (nonatomic, readonly, nullable) NSOutputStream *outputStream;
@end
```

      EAWiFiUnconfiguredAccessoryBrowser类用来浏览未配置的WIFI外设：

```objectivec
@interface EAWiFiUnconfiguredAccessoryBrowser : NSObject
// 初始化方法
- (instancetype)initWithDelegate:(nullable id<EAWiFiUnconfiguredAccessoryBrowserDelegate>)delegate queue:(nullable dispatch_queue_t)queue;
// 代理
@property (weak, nonatomic, nullable) id<EAWiFiUnconfiguredAccessoryBrowserDelegate> delegate;
// 未配置的外设
@property (readonly, copy, atomic) NSSet<EAWiFiUnconfiguredAccessory *> *unconfiguredAccessories;
// 开始进行搜索
- (void)startSearchingForUnconfiguredAccessoriesMatchingPredicate:(nullable NSPredicate *)predicate;
// 结束搜索
- (void)stopSearchingForUnconfiguredAccessories;
// 对外设进行配置
- (void)configureAccessory:(EAWiFiUnconfiguredAccessory *)accessory withConfigurationUIOnViewController:(UIViewController *)viewController;
@end

@protocol EAWiFiUnconfiguredAccessoryBrowserDelegate <NSObject>
// 搜索状态改变后调用的回调
/*
typedef NS_ENUM(NSInteger, EAWiFiUnconfiguredAccessoryBrowserState)
{
    EAWiFiUnconfiguredAccessoryBrowserStateWiFiUnavailable = 0,  // WIFI不可用
    EAWiFiUnconfiguredAccessoryBrowserStateStopped,              // 停止
    EAWiFiUnconfiguredAccessoryBrowserStateSearching,            // 搜索中
    EAWiFiUnconfiguredAccessoryBrowserStateConfiguring,          // 配置中
};
*/
- (void)accessoryBrowser:(EAWiFiUnconfiguredAccessoryBrowser *)browser didUpdateState:(EAWiFiUnconfiguredAccessoryBrowserState)state;

// 发现了未配置的外设
- (void)accessoryBrowser:(EAWiFiUnconfiguredAccessoryBrowser *)browser didFindUnconfiguredAccessories:(NSSet<EAWiFiUnconfiguredAccessory *> *)accessories;

// 移除了未配置的外设
- (void)accessoryBrowser:(EAWiFiUnconfiguredAccessoryBrowser *)browser didRemoveUnconfiguredAccessories:(NSSet<EAWiFiUnconfiguredAccessory *> *)accessories;

// 完成配置
- (void)accessoryBrowser:(EAWiFiUnconfiguredAccessoryBrowser *)browser didFinishConfiguringAccessory:(EAWiFiUnconfiguredAccessory *)accessory withStatus:(EAWiFiUnconfiguredAccessoryConfigurationStatus)status;

@end
```

EAWiFiUnconfiguredAccessory对象描述WIFI外设，如下：

```objectivec
@interface EAWiFiUnconfiguredAccessory : NSObject
// 名称
@property(copy, nonatomic, readonly) NSString *name;
// 设备商
@property(copy, nonatomic, readonly) NSString *manufacturer;
// 模式
@property(copy, nonatomic, readonly) NSString *model;
// WIFI的SSID
@property(copy, nonatomic, readonly) NSString *ssid;
// 硬件地址
@property(copy, nonatomic, readonly) NSString *macAddress;
//属性 
/*
typedef NS_OPTIONS(NSUInteger, EAWiFiUnconfiguredAccessoryProperties)
{
    EAWiFiUnconfiguredAccessoryPropertySupportsAirPlay  = (1 << 0),
    EAWiFiUnconfiguredAccessoryPropertySupportsAirPrint = (1 << 1),
    EAWiFiUnconfiguredAccessoryPropertySupportsHomeKit  = (1 << 2)
};
*/
@property(readonly, nonatomic, readonly) EAWiFiUnconfiguredAccessoryProperties properties;
@end
```

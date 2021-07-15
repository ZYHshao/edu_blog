---
title: iOS开发之CoreLocation框架使用
date: 2018-12-26
categories: iOS逻辑初窥
tags: []
---
## iOS开发之CoreLocation框架使用

     CoreLocation框架是iOS开发中比较基础的一个位置信息相关框架，关于定位和地图，之前也有博客多详细的介绍。但是对于CoreLocation框架，并没有完整和细致的记录。本篇博客将剖析这个框架的结构并进行应用示例。下图为CoreLocation框架的相关类布局图：

![](https://oscimg.oschina.net/oscnet/edb165fccdf17311fe969ceca2ae877460a.jpg)

从图中可以看到，在CoreLocation框架中除了一些数据模型，CLLocationManager作用最为重要，它是整个框架的管理中心，从图中也可以看出，CoreLocation框架功能也非常完善，常规定位，方向信息获取，室内定位，GEO编码等功能都支持。

### 一、CLLocationManager管理类详解

CLLocationManager作为整个CoreLocation框架的核心管理类，其第一部分功能是获取设备的功能可用性，如下：

```objectivec
//获取位置服务是否可用
+ (BOOL)locationServicesEnabled;
//获取方向信息服务是否可用
+ (BOOL)headingAvailable;
//获取设备是否支持显著位置更改的监听
+ (BOOL)significantLocationChangeMonitoringAvailable;
//获取是否支持针对某种位置区域改变的监听
+ (BOOL)isMonitoringAvailableForClass:(Class)regionClass;
//获取是否支持区域监听 同上面一个函数作用一致
+ (BOOL)regionMonitoringAvailable;
//获取区域监听是否可用 除了设备支持 还需要 用户授权
+ (BOOL)regionMonitoringEnabled;
//获取是否支持测距
+ (BOOL)isRangingAvailable;
```

在使用CLLocationManager的服务之前，首先需要获取用户的授权，在iOS8之后，还需要在info.plist中添加相关的键，如下：

![](https://oscimg.oschina.net/oscnet/2415e471fbf16c762802e2f00b02487190d.jpg)

上面标出的3个键，只需要设置一个，根据自己的定位要求来选择即可。

下面列出了与用户授权相关的方法：

```objectivec
//获取当前用户授权类型
/*
CLAuthorizationStatus枚举如下：
kCLAuthorizationStatusNotDetermined  用户目前还没有选择
kCLAuthorizationStatusRestricted     当前应用尚未授权
kCLAuthorizationStatusDenied         用户拒绝使用位置服务
kCLAuthorizationStatusAuthorizedAlways 用户授权始终可以使用位置服务
kCLAuthorizationStatusAuthorizedWhenInUse 用户授权可以在APP使用时使用位置
kCLAuthorizationStatusAuthorized      用户授权使用位置  iOS8之前
*/
+ (CLAuthorizationStatus)authorizationStatus;
//请求在APP使用时使用用户的位置信息
- (void)requestWhenInUseAuthorization;
//请求始终使用用户的位置信息
- (void)requestAlwaysAuthorization;
```

下面列举了CLLocationManager中核心的属性和方法：

```objectivec
//设置代理
@property(assign, nonatomic, nullable) id<CLLocationManagerDelegate> delegate;
//定位服务是否可用
@property(readonly, nonatomic) BOOL locationServicesEnabled;
//位置服务的用途，目前这个属性不再使用，在info.plist中配置
@property(copy, nonatomic, nullable) NSString *purpose;
//指定定位活动的行为
/*
CLActivityTypeAutomotiveNavigation 汽车导航
CLActivityTypeFitness    步行
CLActivityTypeOtherNavigation  其他导航 如火车 轮船
CLActivityTypeAirborne  飞机导航
CLActivityTypeOther 其他类型

*/
@property(assign, nonatomic) CLActivityType activityType;
//设置以米为单位的精度，当小于此精度的位置改变不会收到通知CLLocationDistance就是double类型
@property(assign, nonatomic) CLLocationDistance distanceFilter;
//设置预期的定位精度CLLocationAccuracy为double类型 不一定完全精准 只是设置预期
@property(assign, nonatomic) CLLocationAccuracy desiredAccuracy;
//设置是否自动暂停位置更新 根据电量等
@property(assign, nonatomic) BOOL pausesLocationUpdatesAutomatically;
//设置是否在后台一直更新定位
@property(assign, nonatomic) BOOL allowsBackgroundLocationUpdates;
//当后台定位时 是否显示活动指示器 在状态栏上
@property(assign, nonatomic) BOOL showsBackgroundLocationIndicator;
//最近的一次位置信息
@property(readonly, nonatomic, copy, nullable) CLLocation *location;
//是否支持方向
@property(readonly, nonatomic) BOOL headingAvailable;
//设置最小方向更新精度
@property(assign, nonatomic) CLLocationDegrees headingFilter;
//用来作为参考的物理设备方向
/*
    CLDeviceOrientationUnknown ,
    CLDeviceOrientationPortrait,
    CLDeviceOrientationPortraitUpsideDown,
    CLDeviceOrientationLandscapeLeft,
    CLDeviceOrientationLandscapeRight,
    CLDeviceOrientationFaceUp,
    CLDeviceOrientationFaceDown
*/
@property(assign, nonatomic) CLDeviceOrientation headingOrientation;
//最近一次更新的方向信息
@property(readonly, nonatomic, copy, nullable) CLHeading *heading;
//设置最大的区域监听范围
@property (readonly, nonatomic) CLLocationDistance maximumRegionMonitoringDistance;
//一组目前正在监听的范围集合
@property (readonly, nonatomic, copy) NSSet<__kindof CLRegion *> *monitoredRegions;
//返回一组支持测距的范围
@property (readonly, nonatomic, copy) NSSet<__kindof CLRegion *> *rangedRegions;
//开始更新位置信息
- (void)startUpdatingLocation;
//停止更新位置信息
- (void)stopUpdatingLocation;
//请求一次位置信息
- (void)requestLocation;
//开始更新方向信息
- (void)startUpdatingHeading;
//停止更新方向信息
- (void)stopUpdatingHeading;
//取消航向校准
- (void)dismissHeadingCalibrationDisplay;
//开始显著位置变化的监听
- (void)startMonitoringSignificantLocationChanges;
//停止线束位置变化的监听
- (void)stopMonitoringSignificantLocationChanges;
//开启范围监听
- (void)startMonitoringForRegion:(CLRegion *)region
                 desiredAccuracy:(CLLocationAccuracy)accuracy;
- (void)startMonitoringForRegion:(CLRegion *)region;
//停止范围监听
- (void)stopMonitoringForRegion:(CLRegion *)region;
//获取指定区域的状态 会在代理回调中返回
- (void)requestStateForRegion:(CLRegion *)region;
//开始计算信标范围  关于Beacon 是一种专门的室内定位服务
- (void)startRangingBeaconsInRegion:(CLBeaconRegion *)region;
//停止计算信标范围
- (void)stopRangingBeaconsInRegion:(CLBeaconRegion *)region;
//允许延迟更新  即位置改变了多少距离或者间隔了多少秒后更新
- (void)allowDeferredLocationUpdatesUntilTraveled:(CLLocationDistance)distance
                      timeout:(NSTimeInterval)timeout;
//不允许进行延迟更新
- (void)disallowDeferredLocationUpdates;
//设备是否支持延迟更新
+ (BOOL)deferredLocationUpdatesAvailable;
```

### 二、CLLocationManagerDelegate协议

        CLLocationManagerDelegate配合CLLocationManager进行使用，其中定义了管理器在提供服务时进行回调的相关函数。解析如下：

```objectivec
//位置更新后调用的回调 会将原位置和更新后的位置信息传入
- (void)locationManager:(CLLocationManager *)manager
    didUpdateToLocation:(CLLocation *)newLocation
           fromLocation:(CLLocation *)oldLocation;
//位置更新后调用的回调 会传入按时间排序的一组位置信息对象
- (void)locationManager:(CLLocationManager *)manager
     didUpdateLocations:(NSArray<CLLocation *> *)locations;
//方向信息更新后调用的回调
- (void)locationManager:(CLLocationManager *)manager
       didUpdateHeading:(CLHeading *)newHeading;
//返回布尔值设置是否需要进行方向校准
- (BOOL)locationManagerShouldDisplayHeadingCalibration:(CLLocationManager *)manager;
//当监听的区域状态变化时调用
/*
CLRegionState定义区域的状态
CLRegionStateUnknown 未知状态
CLRegionStateInside  进入
CLRegionStateOutside 出去
*/
- (void)locationManager:(CLLocationManager *)manager
    didDetermineState:(CLRegionState)state forRegion:(CLRegion *)region ;
//区域内有新的信标可用时调用
- (void)locationManager:(CLLocationManager *)manager
    didRangeBeacons:(NSArray<CLBeacon *> *)beacons inRegion:(CLBeaconRegion *)region;
//区域内的信标测距发生错误时调用
- (void)locationManager:(CLLocationManager *)manager
    rangingBeaconsDidFailForRegion:(CLBeaconRegion *)region
    withError:(NSError *)error;
//进入监听区域时调用
- (void)locationManager:(CLLocationManager *)manager
    didEnterRegion:(CLRegion *)region;
//退出监听区域时调用
- (void)locationManager:(CLLocationManager *)manager
    didExitRegion:(CLRegion *)region;
//服务发生错误时调用
- (void)locationManager:(CLLocationManager *)manager
    didFailWithError:(NSError *)error;
//监听区域发生错误时调用
- (void)locationManager:(CLLocationManager *)manager
    monitoringDidFailForRegion:(nullable CLRegion *)region
    withError:(NSError *)error;
//用户授权改变时调用
- (void)locationManager:(CLLocationManager *)manager didChangeAuthorizationStatus:(CLAuthorizationStatus)status;
//开启区域监听时调用
- (void)locationManager:(CLLocationManager *)manager
    didStartMonitoringForRegion:(CLRegion *)region;
//暂停位置更新时调用
- (void)locationManagerDidPauseLocationUpdates:(CLLocationManager *)manager;
//恢复位置更新时调用
- (void)locationManagerDidResumeLocationUpdates:(CLLocationManager *)manager;
//延迟更新位置信息错误时调用
- (void)locationManager:(CLLocationManager *)manager
    didFinishDeferredUpdatesWithError:(nullable NSError *)error;
//如果启用了访问监听 则收到访问会调用
- (void)locationManager:(CLLocationManager *)manager didVisit:(CLVisit *)visit;
```

### 三、进行GEO编码的工具类CLGeocoder

      如前所述，使用CLLocationManager获取到的位置信息是CLLocation对象，这个对象封装了经纬度等基础信息，但是在实际开发中，我们往往需要获取到的是位置的更多实际信息，比如国家，省份，城市等等，GEO编码的作用就是通过经纬度信息发起请求，获取现实意义的更多数据。CLGeocoder类解析如下：

```objectivec
//是否正在进行GEO编码
@property (nonatomic, readonly, getter=isGeocoding) BOOL geocoding;
//反地理位置信息编码 
/*
CLGeocodeCompletionHandler会传回一组标志建筑物
*/
- (void)reverseGeocodeLocation:(CLLocation *)location completionHandler:(CLGeocodeCompletionHandler)completionHandler;
//作用同上 local用来设置区域 可以通过这个参数控制返回的语言
- (void)reverseGeocodeLocation:(CLLocation *)location preferredLocale:(nullable NSLocale *)locale completionHandler:(CLGeocodeCompletionHandler)completionHandler;

//下面这些方法通常与 AddressBook配合使用
//将地址信息字典进行编码 这个方法与AddressBook框架配合使用 AddressBook框架中定义这个字典
- (void)geocodeAddressDictionary:(NSDictionary *)addressDictionary completionHandler:(CLGeocodeCompletionHandler)completionHandler;
- (void)geocodeAddressString:(NSString *)addressString completionHandler:(CLGeocodeCompletionHandler)completionHandler;
- (void)geocodeAddressString:(NSString *)addressString inRegion:(nullable CLRegion *)region completionHandler:(CLGeocodeCompletionHandler)completionHandler;
- (void)geocodeAddressString:(NSString *)addressString inRegion:(nullable CLRegion *)region preferredLocale:(nullable NSLocale *)locale completionHandler:(CLGeocodeCompletionHandler)completionHandler;
//对邮编地址进行GEO编码
- (void)geocodePostalAddress:(CNPostalAddress *)postalAddress completionHandler:(CLGeocodeCompletionHandler)completionHandler;
- (void)geocodePostalAddress:(CNPostalAddress *)postalAddress preferredLocale:(nullable NSLocale *)locale completionHandler:(CLGeocodeCompletionHandler)completionHandler;
//取消编码
- (void)cancelGeocode;
```

### 四、位置信息模型CLLocation相关类

    首先在CoreLocation框架中，位置的经纬度是由CLLocationCoordinate2D结构体描述的，这个结构体定义如下：

```objectivec
struct CLLocationCoordinate2D {
    CLLocationDegrees latitude; //精度
    CLLocationDegrees longitude;//维度
};
```

使用下面的函数可以快速的检查和创建CLLocationCoordinate2D对象：

```objectivec
//检查经纬度对象是否有效
BOOL CLLocationCoordinate2DIsValid(CLLocationCoordinate2D coord);
//创建对象
CLLocationCoordinate2D CLLocationCoordinate2DMake(CLLocationDegrees latitude, CLLocationDegrees longitude);
```

CLFloor类是一个用来描述楼层信息的模型，并不一定精准，是基于地面的粗略计算，其中属性如下：

```objectivec
@interface CLFloor : NSObject <NSCopying, NSSecureCoding>
//楼层 地下室可能为负值
@property(readonly, nonatomic) NSInteger level;

@end
```

下面列举了CLLocation中封装的属性的方法：

```objectivec
//初始化方法
//通过经纬度初始化 CLLocationDegrees就是double类型
- (instancetype)initWithLatitude:(CLLocationDegrees)latitude
    longitude:(CLLocationDegrees)longitude;
//初始化方法
/*
hAccuracy设置水平方向的精确度
verticalAccuracy 设置垂直方向的精确度
timestamp 设置时间戳
*/
- (instancetype)initWithCoordinate:(CLLocationCoordinate2D)coordinate
    altitude:(CLLocationDistance)altitude
    horizontalAccuracy:(CLLocationAccuracy)hAccuracy
    verticalAccuracy:(CLLocationAccuracy)vAccuracy
    timestamp:(NSDate *)timestamp;
/*
course设置方向
speed 设置速度
*/
- (instancetype)initWithCoordinate:(CLLocationCoordinate2D)coordinate
    altitude:(CLLocationDistance)altitude
    horizontalAccuracy:(CLLocationAccuracy)hAccuracy
    verticalAccuracy:(CLLocationAccuracy)vAccuracy
    course:(CLLocationDirection)course
    speed:(CLLocationSpeed)speed
    timestamp:(NSDate *)timestamp;
//获取位置对象的经纬度信息
@property(readonly, nonatomic) CLLocationCoordinate2D coordinate;
//获取海波高度
@property(readonly, nonatomic) CLLocationDistance altitude;
//水平方向精度
@property(readonly, nonatomic) CLLocationAccuracy horizontalAccuracy;
//垂直方向精度
@property(readonly, nonatomic) CLLocationAccuracy verticalAccuracy;
//方向值 0-360之间
@property(readonly, nonatomic) CLLocationDirection course;
//速度  单位m/s
@property(readonly, nonatomic) CLLocationSpeed speed;
//时间戳
@property(readonly, nonatomic, copy) NSDate *timestamp;
//楼层信息
@property(readonly, nonatomic, copy, nullable) CLFloor *floor;
//获取到某个位置之间的距离
- (CLLocationDistance)getDistanceFrom:(const CLLocation *)location;
- (CLLocationDistance)distanceFromLocation:(const CLLocation *)location;
```

### 五、航向信息模型CLHeading

      当使用CLLocationManager进行航向信息的请求时，代理回调中会获取到CLHeading对象，这个对象封装了航向相关数据，解析如下：

```objectivec
//地磁方向0-360
@property(readonly, nonatomic) CLLocationDirection magneticHeading;
//地理方向0-360
@property(readonly, nonatomic) CLLocationDirection trueHeading;
//航向精度
@property(readonly, nonatomic) CLLocationDirection headingAccuracy;
//x轴测量的原始值
@property(readonly, nonatomic) CLHeadingComponentValue x;
//y轴测量的原始值
@property(readonly, nonatomic) CLHeadingComponentValue y;
//z轴测量的原始值
@property(readonly, nonatomic) CLHeadingComponentValue z;
//时间戳
@property(readonly, nonatomic, copy) NSDate *timestamp;
```

### 六、地标数据模型CLPlacemark

    CLPlacemark是进行GEO编码后返回的地标对象，解析如下：

```objectivec
//初始化方法 使用另一个placemark拷贝
- (instancetype)initWithPlacemark:(CLPlacemark *) placemark;
//位置信息
@property (nonatomic, readonly, copy, nullable) CLLocation *location;
//区域范围信息
@property (nonatomic, readonly, copy, nullable) CLRegion *region;
//时区
@property (nonatomic, readonly, copy, nullable) NSTimeZone *timeZone;
//地理信息字典
@property (nonatomic, readonly, copy, nullable) NSDictionary *addressDictionary;
//地标名字
@property (nonatomic, readonly, copy, nullable) NSString *name;
//街道名字
@property (nonatomic, readonly, copy, nullable) NSString *thoroughfare;
//子街道名字
@property (nonatomic, readonly, copy, nullable) NSString *subThoroughfare;
//城镇
@property (nonatomic, readonly, copy, nullable) NSString *locality;
//子城镇
@property (nonatomic, readonly, copy, nullable) NSString *subLocality;
//州域信息
@property (nonatomic, readonly, copy, nullable) NSString *administrativeArea;
//子州域信息
@property (nonatomic, readonly, copy, nullable) NSString *subAdministrativeArea;
//邮编
@property (nonatomic, readonly, copy, nullable) NSString *postalCode;
//ISO国家编码
@property (nonatomic, readonly, copy, nullable) NSString *ISOcountryCode;
//国家
@property (nonatomic, readonly, copy, nullable) NSString *country;
//水域名称
@property (nonatomic, readonly, copy, nullable) NSString *inlandWater;
//海洋名称
@property (nonatomic, readonly, copy, nullable) NSString *ocean;
//相关的兴趣点数组
@property (nonatomic, readonly, copy, nullable) NSArray<NSString *> *areasOfInterest;
```

在CLPlacemark对象中有封装region对象，这个对象封装区域信息，如下：

```objectivec
//通过中心点和半径创建区域
- (instancetype)initCircularRegionWithCenter:(CLLocationCoordinate2D)center
                                      radius:(CLLocationDistance)radius
                                  identifier:(NSString *)identifier;
//区域中心
@property (readonly, nonatomic) CLLocationCoordinate2D center;
//半径
@property (readonly, nonatomic) CLLocationDistance radius;
//进入区域是否发出提醒  需要开启了位置区域监听
@property (nonatomic, assign) BOOL notifyOnEntry;
//离开区域是否发出提醒  需要开启了位置区域监听
@property (nonatomic, assign) BOOL notifyOnExit;
//检查某个点是否在区域内
- (BOOL)containsCoordinate:(CLLocationCoordinate2D)coordinate;
```

CLRegion类还有两个子类，CLCircularRegion类用来创建圆形区域，提供了一个便捷的初始化方法，其作用和CLRegion基本一致。CLBeaconRegion专门用来支持Beacon技术，Beacon技术是室内定位技术的一种，例如商场中有很多店铺，如果有店铺部署了Beacon发射设备，则当iOS设备作为接收设备靠近时会受到通知。CLBeaconRegion除了包含基础的区域信息外，还封装了与Beacon设备相关的设备码等信息。

### 七、区域访问功能

    在iOS8之前，如果我们想获取用户是否在某个区域停留了一段时间是比较困难的，可能要多次定位并进行整理和分析。在iOS8后，CoreLocation框架中提供了方便的方法来处理这个问题。

    首先，用户的区域停留可以理解为用户对某个区域进行了访问，开启和关闭访问监听十分方便，如下：

```objectivec
@interface CLLocationManager (CLVisitExtensions)
//开启服务
- (void)startMonitoringVisits;
//关闭服务
- (void)stopMonitoringVisits;
@end
```

开启后，当有相关的访问信息时，会回调代理中的相应函数，并且收到CLVisit对象，这个对象信息如下：

```objectivec
//访问开始时间
@property (nonatomic, readonly, copy) NSDate *arrivalDate;
//访问结束时间
@property (nonatomic, readonly, copy) NSDate *departureDate;
//位置
@property (nonatomic, readonly) CLLocationCoordinate2D coordinate;
//精度
@property (nonatomic, readonly) CLLocationAccuracy horizontalAccuracy;
```

> 热爱技术，热爱生活，写代码，交朋友 珲少  QQ：316045346

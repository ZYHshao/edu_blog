---
title: iOS定位服务的应用
date: 2015-05-14
categories: iOS逻辑初窥
tags: []
---
## iOS定位服务的应用

### 一、授权的申请与设置

在IOS8之后，IOS的定位服务做了优化，若要使用定位服务，必须先获取用户的授权。

首先需要在info.plist文件中添加一个键：NSLocationAlwaysUsageDescription或者NSLocationWhenInUseUsageDescription。其中NSLocationAlwaysUsageDescription是要始终使用定位服务，NSLocationWhenInUseUsageDescription是只在前台使用定位服务。

![](http://static.oschina.net/uploads/space/2015/0514/112130_maRK_2340880.png)

IOS8中CLLocationManager新增的两个新方法：

\- (void)requestAlwaysAuthorization;

\- (void)requestWhenInUseAuthorization;

这两个方法对应上面的两个键值，用于在代码中申请定位服务权限。

### 二、定位服务相关方法

IOS的定位服务在CoreLocation.framework框架内，首先引入这个框架：

![](http://static.oschina.net/uploads/space/2015/0514/112450_1NAE_2340880.png)

开启定位服务的代码非常简单，示例如下：

```
#import "ViewController.h"
#import <CoreLocation/CoreLocation.h>
@interface ViewController ()<CLLocationManagerDelegate>//定位服务的代理
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    CLLocationManager* manager = [[CLLocationManager alloc]init];//初始化一个定位管理对象
    [manager requestWhenInUseAuthorization];//申请定位服务权限
    manager.delegate=self;//设置代理
    [manager startUpdatingLocation];//开启定位服务
}
//定位位置改变后调用的函数
-(void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations{
    NSLog(@"%@",locations);
}
@end
```

CLLocationManager相关方法解读：

\+ (BOOL)locationServicesEnabled;

判断设备是否支持定位服务

\+ (BOOL)headingAvailable;

判断设备是否支持航向信息功能(海拔，速度，方向等传感器的支持)

\+ (BOOL)significantLocationChangeMonitoringAvailable;

判断设备是否支持更新位置信息

\+ (BOOL)isMonitoringAvailableForClass:(Class)regionClass;

判断设备是否支持区域检测，regionClass是地图框架中的类。

\+ (BOOL)isRangingAvailabl;

判断设备是否支持蓝牙测距

\+ (CLAuthorizationStatus)authorizationStatus;

获得定位服务的授权状态，CLAuthorizationStatus的枚举如下：

```
typedef NS_ENUM(int, CLAuthorizationStatus) {
    kCLAuthorizationStatusNotDetermined = 0,//用户还没有做选择
    kCLAuthorizationStatusRestricted,//应用拒接使用定位服务
    kCLAuthorizationStatusDenied,//用户拒绝授权
    kCLAuthorizationStatusAuthorizedAlways,//8.0后可用，始终授权位置服务
    kCLAuthorizationStatusAuthorizedWhenInUse,//8.0后可用，只在前台授权位置服务
};
```

@property(assign, nonatomic) CLActivityType activityType;

这个属性用来设置位置更新的模式，枚举如下：

```
typedef NS_ENUM(NSInteger, CLActivityType) {
    CLActivityTypeOther = 1,//未知模式，默认为此
    CLActivityTypeAutomotiveNavigation,    //车辆导航模式
    CLActivityTypeFitness,                //行人模式
    CLActivityTypeOtherNavigation         //其他交通工具模式
};
```

模式的应用可以起到节省电量的作用，例如车辆导航模式，当汽车停止时，位置更新服务会暂停。

@property(assign, nonatomic) CLLocationDistance distanceFilter;

设置位置更新的敏感范围，单位为米。

@property(assign, nonatomic) CLLocationAccuracy desiredAccuracy;

设置定位服务的精确度，系统定义好的几个参数如下：

kCLLocationAccuracyBestForNavigation;//导航最高精确  
kCLLocationAccuracyBest;//高精确  
kCLLocationAccuracyNearestTenMeters;//10米  
kCLLocationAccuracyHundredMeters;//百米  
kCLLocationAccuracyKilometer;//千米  
kCLLocationAccuracyThreeKilometers;//三公里

@property(assign, nonatomic) BOOL pausesLocationUpdatesAutomatically;

设置位置更新是否自动暂停

@property(readonly, nonatomic, copy) CLLocation *location;

最后一次更新的位置信息，只读属性

@property(assign, nonatomic) CLLocationDegrees headingFilter;

相关航向更新的敏感范围

@property(assign, nonatomic) CLDeviceOrientation headingOrientation;

定位航向时的参照方向默认为正北，枚举如下：

```
typedef NS_ENUM(int, CLDeviceOrientation) {
    CLDeviceOrientationUnknown = 0,//方向未知
    CLDeviceOrientationPortrait,//纵向模式
    CLDeviceOrientationPortraitUpsideDown,//纵向倒置模式
    CLDeviceOrientationLandscapeLeft,//左向横向模式
    CLDeviceOrientationLandscapeRight,//右向横向模式
    CLDeviceOrientationFaceUp,//水平屏幕向上模式
    CLDeviceOrientationFaceDown//水平屏幕下模式
};
```

@property(readonly, nonatomic, copy) CLHeading *heading;

最后一个定位得到的航向信息

\- (void)startUpdatingLocation;

开启定位服务

\- (void)stopUpdatingLocation;

停止定位服务

\- (void)startUpdatingHeading;

开启航向地理信息服务

\- (void)stopUpdatingHeading;

停止航向地理信息服务

### 三、定位服务代理的相关方法

\- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations；

位置更新后调用的方法，数组中是所有定位到的位置信息，最后一个是最新的。

\- (void)locationManager:(CLLocationManager *)manager didUpdateHeading:(CLHeading *)newHeading；

航向信息更新后调用的方法

\- (void)locationManager:(CLLocationManager *)manager didFailWithError:(NSError *)error;

定位异常时调用的方法

### 四、定位服务获取到的位置对象

上面也提到，定位后返回的数组中存放的都是CLLocation对象，这里面有很详细的位置信息，属性如下：

@property(readonly, nonatomic) CLLocationCoordinate2D coordinate;

经纬度属性，CLLocationCoordinate2D是一个结构体，如下：

```
typedef struct {
    CLLocationDegrees latitude;//纬度
    CLLocationDegrees longitude;//经度
} CLLocationCoordinate2D;
```

@property(readonly, nonatomic) CLLocationDistance altitude;

海拔高度，浮点型

@property(readonly, nonatomic) CLLocationAccuracy horizontalAccuracy;

水平方向的容错半径

@property(readonly, nonatomic) CLLocationAccuracy verticalAccuracy;

竖直方向的容错半径

@property(readonly, nonatomic) CLLocationDirection course;

设备前进的方向，取值范围为0-359.9，相对正北方向

@property(readonly, nonatomic) CLLocationSpeed speed;

速度，单位为m/s

@property(readonly, nonatomic, copy) NSDate *timestamp;

定位时的时间戳

### 五、航标定位得到的航标信息对象

CLHeading对象的属性信息：

@property(readonly, nonatomic) CLLocationDirection magneticHeading;

设备朝向航标方向，0为北磁极。

@property(readonly, nonatomic) CLLocationDirection trueHeading;

设备朝向真实方向，0被地理上的北极

@property(readonly, nonatomic) CLLocationDirection headingAccuracy;

方向偏差

@property(readonly, nonatomic) CLHeadingComponentValue x;

x轴的方向值

@property(readonly, nonatomic) CLHeadingComponentValue y;

y轴方向值

@property(readonly, nonatomic) CLHeadingComponentValue z;

z轴方向值

@property(readonly, nonatomic, copy) NSDate *timestamp;

方向定位时间戳

如有疏漏 欢迎指正

学习使用 欢迎转载

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

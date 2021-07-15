---
title: iOS开发之CoreMotion框架的应用
date: 2019-01-22
categories: iOS逻辑初窥
tags: []
---
## iOS开发之CoreMotion框架的应用

      我们知道，现在智能手机手机的功能已经越来越强大。小小的手机中集成了众多的传感器配件。通过这些传感器可以获取到手机甚至用户的状态信息。

      在iOS5之前，加速度传感器的相关信息封装在UIAccelerometer这个类中，其主要用来获取设备在三维空间中的状态信息，之后，加速度传感器以及螺旋仪传感器的相关信息都封装在了CoreMotion这个框架中，这个框架对加速度，磁力以及螺旋仪传感器信息进行统一管理，并封装了许多强大的计算方法帮助开发者获取设备的空间状态。

      之前有写过一篇关于UIAccelerometer与CoreMotion简单使用的博客，比较偏用法介绍，并不系统，本篇博客是针对CoreMotion的完善与补充。

[https://my.oschina.net/u/2340880/blog/543434](https://my.oschina.net/u/2340880/blog/543434)

### 一、CoreMotion框架整体结构

    在学习这个框架之前，首先需要对框架中类的关系与作用有个整体的了解。下图展示了CoreMotion框架的整体结构：

![](https://oscimg.oschina.net/oscnet/e778c6022c890fe98765be064810b084bfb.jpg)

从上图中可以看出，CoreMotion框架中主要分为3大块，一部分是用来获取设备的运动状态，如速度，加速度，海拔，三维方向等。一部分是用来配合iWatch进行用户的运动状态获取、另一部分为用户步数相关接口。

### 二、CMMotionManager

      CMMotionManager类是CoreMotion框架中非常核心的一个类，其用来进行设备运动信息的整体管理。主要包括开启更新信息，停止更新信息，获取更新信息等。解析如下：

```objectivec
//获取加速计是否可用
@property(readonly, nonatomic, getter=isAccelerometerAvailable) BOOL accelerometerAvailable;
//加速计更新间隔
@property(assign, nonatomic) NSTimeInterval accelerometerUpdateInterval;
//加速计是否在持续进行更新
@property(readonly, nonatomic, getter=isAccelerometerActive) BOOL accelerometerActive;
//最后一次更新的加速计信息  CMAccelerometerData后面会介绍
@property(readonly, nullable) CMAccelerometerData *accelerometerData;
//开始进行加速计数据更新
- (void)startAccelerometerUpdates;
//开始进行加速计数据更新 并且指定回调函数以及回调函数执行的线程
- (void)startAccelerometerUpdatesToQueue:(NSOperationQueue *)queue withHandler:(CMAccelerometerHandler)handler;
//停止加速计数据的更新
- (void)stopAccelerometerUpdates;
//陀螺仪是否可用
@property(readonly, nonatomic, getter=isGyroAvailable) BOOL gyroAvailable;
//陀螺仪数据的更新间隔
@property(assign, nonatomic) NSTimeInterval gyroUpdateInterval;
//陀螺仪是否在持续进行更新
@property(readonly, nonatomic, getter=isGyroActive) BOOL gyroActive;
//陀螺仪数据
@property(readonly, nullable) CMGyroData *gyroData;
//开启陀螺仪的更新
- (void)startGyroUpdates;
//开始进行陀螺仪的更新 并且指定回调函数以及回调函数执行的线程
- (void)startGyroUpdatesToQueue:(NSOperationQueue *)queue withHandler:(CMGyroHandler)handler;
//停止进行陀螺仪数据的更新
- (void)stopGyroUpdates;
//磁力计是否可用
@property(readonly, nonatomic, getter=isMagnetometerAvailable) BOOL magnetometerAvailable;
//磁力计数据更新间隔
@property(assign, nonatomic) NSTimeInterval magnetometerUpdateInterval;
//磁力计数据是否在持续更新
@property(readonly, nonatomic, getter=isMagnetometerActive) BOOL magnetometerActive;
//磁力计数据
@property(readonly, nullable) CMMagnetometerData *magnetometerData;
//开始更新磁力计数据
- (void)startMagnetometerUpdates;
//开始更新磁力计数据 并且指定回调函数以及回调函数执行的线程
- (void)startMagnetometerUpdatesToQueue:(NSOperationQueue *)queue withHandler:(CMMagnetometerHandler)handler;
//停止磁力计的更新
- (void)stopMagnetometerUpdates;

//设备运动数据并非某个传感器的数据 而是上面3种传感器数据的组合与运算
//设备运动数据是否可用
@property(readonly, nonatomic, getter=isDeviceMotionAvailable) BOOL deviceMotionAvailable;
//设备运动数据更新间隔
@property(assign, nonatomic) NSTimeInterval deviceMotionUpdateInterval;
//是否在持续更新设备运动数据
@property(readonly, nonatomic, getter=isDeviceMotionActive) BOOL deviceMotionActive;
//最后一次更新的设备运动信息数据
@property(readonly, nullable) CMDeviceMotion *deviceMotion;
//开始更新设备运动数据
- (void)startDeviceMotionUpdates;
//开始更新设备运动数据 并指定回调函数以及回调函数执行的线程
- (void)startDeviceMotionUpdatesToQueue:(NSOperationQueue *)queue withHandler:(CMDeviceMotionHandler)handler;
//停止更新设备运动信息
- (void)stopDeviceMotionUpdates;
```

上面的方法看上去非常繁多，其实很有规律，总的来说就是对是否开启传感器数据进行管理，并且进行传感器数据的获取。下面我们来看几种具体的传感器数据类的定义。

### 三、数据模型类

      首先，CoreMotion框架中的数据模型类都继承自CMLogItem类，这个类里面只有一个属性：

```objectivec
@interface CMLogItem : NSObject <NSSecureCoding, NSCopying>
@property(readonly, nonatomic) NSTimeInterval timestamp;
@end
```

CMLogItem类的timestamp属性用来标记数据记录的时间戳。

#### 1.加速计数据

      CMAccelerometerData是加速计数据的数据模型类：

```objectivec
@interface CMAccelerometerData : CMLogItem
//加速计数据
@property(readonly, nonatomic) CMAcceleration acceleration;
@end
//加速计数据结构体
typedef struct {
    double x;   //x方向加速度
    double y;   //y方向加速度
    double z;   //z方向加速度
} CMAcceleration;
```

#### 2、陀螺仪数据

      CMGyroData是陀螺仪数据的数据模型类：

```objectivec
@interface CMGyroData : CMLogItem
//角速度数据
@property(readonly, nonatomic) CMRotationRate rotationRate;
@end

//角速度结构体
typedef struct {
    double x;  //x方向的角速度
    double y;  //y方向的角速度 
    double z;  //z方向的角速度
} CMRotationRate;
```

#### 3.磁强计数据

      CMMagnetometerData是磁强计数据模型类：

```objectivec
@interface CMMagnetometerData : CMLogItem
{
//磁强数据
@property(readonly, nonatomic) CMMagneticField magneticField;

@end

typedef struct {
    double x;   //x轴磁场
    double y;   //y轴磁场
    double z;   //z轴磁场
} CMMagneticField;
```

#### 4.设备运动信息

      CMDeviceMotion类包含了设备的空间状态信息：

```objectivec
@interface CMDeviceMotion : CMLogItem
//设备的空间状态  CMAttitude后面会介绍
@property(readonly, nonatomic) CMAttitude *attitude;
//设备的 陀螺仪数据
@property(readonly, nonatomic) CMRotationRate rotationRate;
//设备的 加速计数据
@property(readonly, nonatomic) CMAcceleration gravity;
//获取用户给设备带来的加速度
@property(readonly, nonatomic) CMAcceleration userAcceleration;
//设备附近磁场相关信息
@property(readonly, nonatomic) CMCalibratedMagneticField magneticField;
//返回航向角度
@property(readonly, nonatomic) double heading;
@end
```

CMCalibratedMagneticField是一个结构体，如下：

```objectivec
typedef struct {
    CMMagneticField field;  //磁场
    CMMagneticFieldCalibrationAccuracy accuracy; //磁场强度
} CMCalibratedMagneticField;

typedef NS_ENUM(int, CMMagneticFieldCalibrationAccuracy) {
    CMMagneticFieldCalibrationAccuracyUncalibrated = -1,
    CMMagneticFieldCalibrationAccuracyLow, //低
    CMMagneticFieldCalibrationAccuracyMedium,//中
    CMMagneticFieldCalibrationAccuracyHigh//高
} ;
```

CMAttitude类中封装的信息如下：

```objectivec
@interface CMAttitude : NSObject <NSCopying, NSSecureCoding>
{
//设备翻滚弧度
@property(readonly, nonatomic) double roll;
//旋转弧度
@property(readonly, nonatomic) double pitch;
//航偏
@property(readonly, nonatomic) double yaw;
//描述设备状态的旋转矩阵
@property(readonly, nonatomic) CMRotationMatrix rotationMatrix;
//描述设备姿态的四元数
@property(readonly, nonatomic) CMQuaternion quaternion;
//进行转换
- (void)multiplyByInverseOfAttitude:(CMAttitude *)attitude;

@end
//四元数
typedef struct
{
    double x, y, z, w;
} CMQuaternion;
//矩阵
typedef struct 
{
    double m11, m12, m13;
    double m21, m22, m23;
    double m31, m32, m33;
} CMRotationMatrix;
```

### 四、高度信息

     CoreMotion框架中的CMAltimeter类提供对设备高度相关信息的数据支持，这个类是iOS 8后新加入的，CMAltimeter类解析如下：

```objectivec
@interface CMAltimeter : NSObject
//是否支持相对高度变化
+ (BOOL)isRelativeAltitudeAvailable;
//进行用户权限的申请
+ (CMAuthorizationStatus)authorizationStatus;
//开始更新高度变化信息数据
- (void)startRelativeAltitudeUpdatesToQueue:(NSOperationQueue *)queue withHandler:(CMAltitudeHandler)handler;
//停止更新高度变化信息数据
- (void)stopRelativeAltitudeUpdates;
@end
```

CMAltitudeData是高度信息数据：

```objectivec
@interface CMAltitudeData : CMLogItem
//相对高度 单位为米
@property(readonly, nonatomic) NSNumber *relativeAltitude;
//压力 单位为千帕
@property(readonly, nonatomic) NSNumber *pressure;
@end
```

### 五、用户活动信息

      CMMotionActivityManager类是iOS 7之后新引入到CoreMotion框架中的，这个类用来对用户的活动信息进行管理，解析如下：

```objectivec
@interface CMMotionActivityManager : NSObject
//活动数据是否可用
+ (BOOL)isActivityAvailable;
//进行用户权限的申请
+ (CMAuthorizationStatus)authorizationStatus;
//请求某一段时间内的用户活动信息
- (void)queryActivityStartingFromDate:(NSDate *)start
                               toDate:(NSDate *)end
                              toQueue:(NSOperationQueue *)queue
                          withHandler:(CMMotionActivityQueryHandler)handler;
//开始进行用户活动信息的更新
- (void)startActivityUpdatesToQueue:(NSOperationQueue *)queue
                        withHandler:(CMMotionActivityHandler)handler;
//停止用户活动信息的更新
- (void)stopActivityUpdates;
@end
```

CMMotionActivity是用户活动信息的具体记录:

```objectivec
@interface CMMotionActivity : CMLogItem
//数据的可信度
/*
typedef NS_ENUM(NSInteger, CMMotionActivityConfidence) {
    CMMotionActivityConfidenceLow = 0,  //可信度低
    CMMotionActivityConfidenceMedium,   //可信度中
    CMMotionActivityConfidenceHigh      //可信度高
};
*/
@property(readonly, nonatomic) CMMotionActivityConfidence confidence;
//记录开始时间
@property(readonly, nonatomic) NSDate *startDate;
//是否是位置状态 如果是 可能关机状态
@property(readonly, nonatomic) BOOL unknown;
//设备是否没有移动
@property(readonly, nonatomic) BOOL stationary;
//设备持有者是否在步行
@property(readonly, nonatomic) BOOL walking;
//设备持有者是否在跑步
@property(readonly, nonatomic) BOOL running;
//设备持有者是否在乘车
@property(readonly, nonatomic) BOOL automotive;
//设备持有者是否在骑自行车
@property(readonly, nonatomic) BOOL cycling;
@end
```

### 六、用户手臂动作分析

      在iOS 12系统后，CoreMotion框架中又引入了一些列配合iWatch进行用户手臂动作分析的类，可以分析出用户是否发生了运动障碍等。其主要由CMMovementDisorderManager类进行管理，如下：

```objectivec
@interface CMMovementDisorderManager : NSObject
//运动障碍管理类是否可用
+ (BOOL)isAvailable;
//进行用户权限的请求
+ (CMAuthorizationStatus)authorizationStatus;
//记录和计算一段时间内的震颤和运动异常结果
- (void)monitorKinesiasForDuration:(NSTimeInterval)duration;
//获取一段时间内的运动障碍记录数据
- (void)queryDyskineticSymptomFromDate:(NSDate *)fromDate toDate:(NSDate *)toDate withHandler:(CMDyskineticSymptomResultHandler)handler;
//获取一段时间内的震颤记录数据
- (void)queryTremorFromDate:(NSDate *)fromDate toDate:(NSDate *)toDate withHandler:(CMTremorResultHandler)handler;
//最后一次更新数据的时间
- (NSDate * _Nullable)lastProcessedDate;
//最后一次计算数据的过期时间
- (NSDate * _Nullable)monitorKinesiasExpirationDate;
@end
```

CMDyskineticSymptomResult运动障碍数据模型：

```objectivec
@interface CMDyskineticSymptomResult : NSObject <NSCopying, NSSecureCoding>
//记录数据的开始时间
@property (copy, nonatomic, readonly) NSDate *startDate;
//记录数据的结束时间
@property (copy, nonatomic, readonly) NSDate *endDate;
//运动异常可能出现的百分比 
@property (nonatomic, readonly) float percentUnlikely;
//正常的百分比
@property (nonatomic, readonly) float percentLikely;
@end
```

CMTremorResult记录用户震颤数据：

```objectivec
@interface CMTremorResult : NSObject <NSCopying, NSSecureCoding>
//数据记录开始时间
@property (copy, nonatomic, readonly) NSDate *startDate;
//数据记录结束时间
@property (copy, nonatomic, readonly) NSDate *endDate;
//无法确定的时间百分比
@property (nonatomic, readonly) float percentUnknown;
//未检测到震颤的时间百分比
@property (nonatomic, readonly) float percentNone;
//可能发生震颤的百分比低 微震颤
@property (nonatomic, readonly) float percentSlight;
//可能发生震颤的百分比高 微震颤
@property (nonatomic, readonly) float percentMild;
//可能发生震颤的百分比高 中等震颤
@property (nonatomic, readonly) float percentModerate;
//可能发生震颤的百分比高 高震颤
@property (nonatomic, readonly) float percentStrong;
```

### 七、计步器应用

      在iOS 8之后，CoreMotion中引入了CMPedometer相关计步器类，这些类封装的更加应用层，开发者可以直接获取用户步数相关数据，CMPedometer是管理类，解析如下：

```objectivec
@interface CMPedometer : NSObject
//计步器是否可用
+ (BOOL)isStepCountingAvailable;
//距离检测是否可用
+ (BOOL)isDistanceAvailable;
//楼层检测是否可用
+ (BOOL)isFloorCountingAvailable;
//速度估算是否支持
+ (BOOL)isPaceAvailable;
//频率估算是否支持
+ (BOOL)isCadenceAvailable;
//计步器功能是否支持
+ (BOOL)isPedometerEventTrackingAvailable;
//进行用户权限申请
+ (CMAuthorizationStatus)authorizationStatus;
//请求一段时间的计步器数据
- (void)queryPedometerDataFromDate:(NSDate *)start
                            toDate:(NSDate *)end
                       withHandler:(CMPedometerHandler)handler;
//请求从某个时间至今的计步器数据
- (void)startPedometerUpdatesFromDate:(NSDate *)start
                          withHandler:(CMPedometerHandler)handler;
//停止计步器数据更新
- (void)stopPedometerUpdates;
//开始更新计步器事件
- (void)startPedometerEventUpdatesWithHandler:(CMPedometerEventHandler)handler;
//停止更新计数器事件
- (void)stopPedometerEventUpdates;
@end
```

CMPedometerEvent类记录计步器的事件变化：

```objectivec
@interface CMPedometerEvent : NSObject <NSSecureCoding, NSCopying>
//记录数据的时间
@property(readonly, nonatomic) NSDate *date;
/*
typedef NS_ENUM(NSInteger, CMPedometerEventType) {
    CMPedometerEventTypePause,   //计步器暂停
    CMPedometerEventTypeResume   //计步器恢复
}
*/
@property(readonly, nonatomic) CMPedometerEventType type;
@end


```

CMPedometerData计步器数据类：

```objectivec
@interface CMPedometerData
//记录开始时间
@property(readonly, nonatomic) NSDate *startDate;
//记录结束时间
@property(readonly, nonatomic) NSDate *endDate;
//步数
@property(readonly, nonatomic) NSNumber *numberOfSteps;
//距离
@property(readonly, nonatomic, nullable) NSNumber *distance;
//通过楼梯上升的楼层数
@property(readonly, nonatomic, nullable) NSNumber *floorsAscended;
//通过楼梯下降的楼层数
@property(readonly, nonatomic, nullable) NSNumber *floorsDescended;
//估算速度
@property(readonly, nonatomic, nullable) NSNumber *currentPace;
//步数频率
@property(readonly, nonatomic, nullable) NSNumber *currentCadence;
//平均速度
@property(readonly, nonatomic, nullable) NSNumber *averageActivePace;
@end
```

在CoreMotion中，CMStepCounter也是一个记录器类，其比较简易，只在iOS8之前进行使用，解析如下：

```objectivec
@interface CMStepCounter : NSObject
//计步器是否可用
+ (BOOL)isStepCountingAvailable;
//请求一段时间内的步数信息
- (void)queryStepCountStartingFrom:(NSDate *)start
                                to:(NSDate *)end
                           toQueue:(NSOperationQueue *)queue
                       withHandler:(CMStepQueryHandler)handler;
//进行不是更新
- (void)startStepCountingUpdatesToQueue:(NSOperationQueue *)queue
                               updateOn:(NSInteger)stepCounts
                            withHandler:(CMStepUpdateHandler)handler;
//停止计步器更新
- (void)stopStepCountingUpdates;
@end
```

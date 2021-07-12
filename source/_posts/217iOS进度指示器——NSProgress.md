---
title: iOS进度指示器——NSProgress
date: 2016-05-22
categories: iOS逻辑初窥
tags: []
---
## iOS进度指示器——NSProgress

### 一、引言

        在iOS7之前，系统一直没有提供一个完整的框架来描述任务进度相关的功能。这使得在开发中进行耗时任务进度的监听将什么麻烦，在iOS7之后，系统提供了NSProgress类来专门报告任务进度。

### 二、创建单任务进度监听器

        单任务进度的监听是NSProgress最简单的一种运用场景，我们来用定时器模拟一个耗时任务，示例代码如下：

```javascript
@interface ViewController ()
{
    NSProgress * progress;
}
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    //这个方法将创建任务进度管理对象 UnitCount是一个基于UI上的完整任务的单元数
    progress = [NSProgress progressWithTotalUnitCount:10];
    NSTimer * timer = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(task) userInfo:nil repeats:YES];
    //对任务进度对象的完成比例进行监听
    [progress addObserver:self forKeyPath:@"fractionCompleted" options:NSKeyValueObservingOptionNew context:nil];
}
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context

{
    NSLog(@"进度= %f",progress.fractionCompleted);
}
-(void)task{
   //完成任务单元数+1
    
    if (progress.completedUnitCount<progress.totalUnitCount) {
        progress.completedUnitCount +=1;
    }
    
}

```

上面的示例代码中，fractionCompleted属性为0-1之间的浮点值，为任务的完成比例。NSProgress对象中还有两个字符串类型的属性，这两个属性将进度信息转化成固定的格式：

```javascript
//显示完后比例 如：10% completed
@property (null_resettable, copy) NSString *localizedDescription;
//完成数量 如：1 of 10
@property (null_resettable, copy) NSString *localizedAdditionalDescription;
```

### 三、创建多任务进度监听器

        上面演示了只有一个任务时的进度监听方法，实际上，在开发中，一个任务中往往又有许多子任务，NSProgress是以树状的结构进行设计的，其支持子任务的嵌套，示例如下：

```javascript
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    //这个方法将创建任务进度管理对象 UnitCount是一个基于UI上的完整任务的单元数
    progress = [NSProgress progressWithTotalUnitCount:10];
    //对任务进度对象的完成比例进行监听
    [progress addObserver:self forKeyPath:@"fractionCompleted" options:NSKeyValueObservingOptionNew context:nil];
    //向下分支出一个子任务 子任务进度总数为5个单元 即当子任务完成时 父progerss对象进度走5个单元
    [progress becomeCurrentWithPendingUnitCount:5];
    [self subTaskOne];
    [progress resignCurrent];
    //向下分出第2个子任务
    [progress becomeCurrentWithPendingUnitCount:5];
    [self subTaskOne];
    [progress resignCurrent];
}

-(void)subTaskOne{
    //子任务总共有10个单元
    NSProgress * sub =[NSProgress progressWithTotalUnitCount:10];
    int i=0;
    while (i<10) {
        i++;
        sub.completedUnitCount++;
    }
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context

{
    NSLog(@"= %@",progress.localizedAdditionalDescription);
}

```

NSProgress的这种树状设计模式乍看起来确实有些令人费解，有一点需要注意，becomeCurrentWithPendingUnitCount:方法的意义是将此NSProgress对象注册为当前线程任务的根进度管理对象，resignCurrent方法为取消注册，这两个方法必须成对出现，当一个NSProgress对象被注册为当前线程的根节点时，后面使用类方法 progressWithTotalUnitCount:创建的NSProgress对象都默认作为子节点添加。

### 四、iOS9之后进行多任务进度监听的新设计方法

        正如上面的例子所演示，注册根节点的方式可读性很差，代码结构也不太清晰，可能Apple的工程师们也觉得如此，在iOS9之后，NSProgress类中又添加了一些方法，通过这些方法可以更加清晰的表达进度指示器之间的层级结构，示例代码如下：

```javascript
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    //这个方法将创建任务进度管理对象 UnitCount是一个基于UI上的完整任务的单元数
    progress = [NSProgress progressWithTotalUnitCount:10];
    //对任务进度对象的完成比例进行监听
    [progress addObserver:self forKeyPath:@"fractionCompleted" options:NSKeyValueObservingOptionNew context:nil];
    //创建子节点
    NSProgress * sub = [NSProgress progressWithTotalUnitCount:10 parent:progress pendingUnitCount:5];
    NSProgress * sub2 = [NSProgress progressWithTotalUnitCount:10 parent:progress pendingUnitCount:5];
    for (int i=0; i<10; i++) {
        sub.completedUnitCount ++;
        sub2.completedUnitCount ++;
    }
}

```

如上面代码所示，代码结构变得更加清晰，可操作性也更强了。

### 五、一点小总结

```
//获取当前线程的进度管理对象根节点
//注意：当有NSProgress对象调用了becomeCurrentWithPendingUnitCount:方法后，这个方法才能获取到
+ (nullable NSProgress *)currentProgress;
//创建一个NSProgress对象，需要传入进度的单元数量
+ (NSProgress *)progressWithTotalUnitCount:(int64_t)unitCount;
//和上一个方法功能相似 iOS9之后的新方法
+ (NSProgress *)discreteProgressWithTotalUnitCount:(int64_t)unitCount;
//iOS9之后的新方法 创建某个进度指示器节点的子节点
+ (NSProgress *)progressWithTotalUnitCount:(int64_t)unitCount parent:(NSProgress *)parent pendingUnitCount:(int64_t)portionOfParentTotalUnitCount;
//NSProgress实例的初始化方法 自父节点参数可以为nil
- (instancetype)initWithParent:(nullable NSProgress *)parentProgressOrNil userInfo:(nullable NSDictionary *)userInfoOrNil;
//注册为当前线程根节点
- (void)becomeCurrentWithPendingUnitCount:(int64_t)unitCount;
//取消注册 与注册方法必须同步出现
- (void)resignCurrent;
//iOS9新方法 向一个节点中添加一个子节点
- (void)addChild:(NSProgress *)child withPendingUnitCount:(int64_t)inUnitCount;
//进度单元总数
@property int64_t totalUnitCount;
//已完成的进度单元数
@property int64_t completedUnitCount;
//是否可取消
@property (getter=isCancellable) BOOL cancellable;
//是否可暂停
@property (getter=isPausable) BOOL pausable;
//进度比例 0-1之间
@property (readonly) double fractionCompleted;
//取消
- (void)cancel;
//暂停
- (void)pause;
//恢复
- (void)resume

```

### 六、关于NSProgress对象的用户配置字典

        在NSProgress对象的用户字典中可以设置一些特定的键值来进行显示模式的设置，示例如下：

```
//设置剩余时间 会影响localizedAdditionalDescription的值
/*
例如：0 of 10 — About 10 seconds remaining
*/
[progress setUserInfoObject:@10 forKey:NSProgressEstimatedTimeRemainingKey];
//设置完成速度信息 会影响localizedAdditionalDescription的值
/*
例如：Zero KB of 10 bytes (15 bytes/sec)
*/
[progress setUserInfoObject:@15 forKey:NSProgressThroughputKey];
/*
下面这些键值的生效 必须将NSProgress对象的kind属性设置为 NSProgressKindFile
NSProgressFileOperationKindKey键对应的是提示文字类型 会影响localizedDescription的值
NSProgressFileOperationKindKey可选的对应值如下：
NSProgressFileOperationKindDownloading： 显示Downloading files…
NSProgressFileOperationKindDecompressingAfterDownloading： 显示Decompressing files…
NSProgressFileOperationKindReceiving： 显示Receiving files…
NSProgressFileOperationKindCopying： 显示Copying files…
*/
 [progress setUserInfoObject:NSProgressFileOperationKindDownloading forKey:NSProgressFileOperationKindKey];
/*
NSProgressFileTotalCountKey键设置显示的文件总数 
例如：Copying 100 files…
*/
 [progress setUserInfoObject:@100 forKey:NSProgressFileTotalCountKey];
//设置已完成的数量
[progress setUserInfoObject:@1 forKey:NSProgressFileCompletedCountKey];
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

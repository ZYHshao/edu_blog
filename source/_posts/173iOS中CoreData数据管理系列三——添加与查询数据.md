---
title: iOS中CoreData数据管理系列三——添加与查询数据
date: 2016-01-29
categories: iOS逻辑初窥
tags: []
---
## iOS中CoreData数据管理系列三——添加与查询数据

### 一、引言

    在前两篇博客中，分别介绍了iOS中CoreData框架创建数据模型和CoreData框架中的三个核心类。博客地址如下：

iOS中CoreData框架简介：[http://my.oschina.net/u/2340880/blog/610488](http://my.oschina.net/u/2340880/blog/610488)。

CoreData框架中三个核心的类：[http://my.oschina.net/u/2340880/blog/610948](http://my.oschina.net/u/2340880/blog/610948)。

本篇博客将综合使用三个核心的类，进行数据创建和查询的操作介绍。

### 二、建立数据对象类

    前面博客介绍的NSManagedObjectModel是数据管理模型，可以将其类比如数据库，NSManagedObjectModel中存放着数据库的结构信息。NSEntityDescription是实体描述对象，它可以类比如数据库中的表，NSEntityDescription存放的是表的结构信息。这些类都是一些抽象的结构类，并不存储实际每条数据的信息，具体的数据由NSManagedObject类来描述，我们一般会将实体类化继承于NSManagedObject。

    Xocde工具提供了快捷的实体类化功能，还拿我们一开始创建的班级与学生实体来演示，点击.xcdatamodeld文件，点击Xcode工具上方导航栏的Editor标签，选择Creat NSManagedObject Subclass选项，在弹出的窗口中勾选要类化的实体，如下图：

![](http://static.oschina.net/uploads/space/2016/0129/151849_9YTv_2340880.png)![](http://static.oschina.net/uploads/space/2016/0129/152012_iWai_2340880.png)

这时，Xcode会自动为我们创建一个文件，这些文件中有各个类中属性的声明。

### 三、创建一条数据

    使用如下代码进行数据的创建：

```
    //读取数据模型文件
    NSURL *modelUrl = [[NSBundle mainBundle]URLForResource:@"Model" withExtension:@"momd"];
    //创建数据模型
    NSManagedObjectModel * mom = [[NSManagedObjectModel alloc]initWithContentsOfURL:modelUrl];
    //创建持久化存储协调者
    NSPersistentStoreCoordinator * psc = [[NSPersistentStoreCoordinator alloc]initWithManagedObjectModel:mom];
    //数据库保存路径
    NSURL * path =[NSURL fileURLWithPath:[[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)lastObject] stringByAppendingPathComponent:@"CoreDataExample.sqlite"]];
    //为持久化协调者添加一个数据接收栈
    /*
    可以支持的类型如下：
     NSString * const NSSQLiteStoreType;//sqlite
     NSString * const NSXMLStoreType;//XML
     NSString * const NSBinaryStoreType;//二进制
     NSString * const NSInMemoryStoreType;//内存
    */
    [psc addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:path options:nil error:nil];
    //创建数据管理上下文
    NSManagedObjectContext * moc = [[NSManagedObjectContext alloc]initWithConcurrencyType:NSMainQueueConcurrencyType];
    //关联持久化协调者
    [moc setPersistentStoreCoordinator:psc];
    //创建数据对象
    /*
    数据对象的创建是通过实体名获取到的
    */
    SchoolClass * modelS = [NSEntityDescription insertNewObjectForEntityForName:@"SchoolClass" inManagedObjectContext:moc];
    //对数据进行设置
    modelS.name = @"第一班";
    modelS.stuNum = @60;
    //进行存储
    if ([moc save:nil]) {
        NSLog(@"新增成功");
    }
    NSLog(@"%@",[[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)lastObject] stringByAppendingPathComponent:@"CoreDataExample.sqlite"]);
```

找到在打印出的路径，会发现里面多了一个sqlite文件，其中有一张表中添加进了一条数据。

### 四、查询数据

    CoreData中通过查询请求来对数据进行查询操作，查询请求由NSFetchRequest来进行管理和维护。

    NSFetchRequest主要提供两个方面的查询服务：

    1.提供范围查询的相关功能

    2.提供查询结果返回类型与排序的相关功能

    NSFetchRequest中常用方法如下：

```
//创建一个实体的查询请求 可以理解为在某个表中进行查询
+ (instancetype)fetchRequestWithEntityName:(NSString*)entityName;
//查询条件
@property (nullable, nonatomic, strong) NSPredicate *predicate;
//数据排序
@property (nullable, nonatomic, strong) NSArray<NSSortDescriptor *> *sortDescriptors;
//每次查询返回的数据条数
@property (nonatomic) NSUInteger fetchLimit;
//设置查询到数据的返回类型
/*
typedef NS_OPTIONS(NSUInteger, NSFetchRequestResultType) {
    NSManagedObjectResultType        = 0x00,
    NSManagedObjectIDResultType        = 0x01,
    NSDictionaryResultType          NS_ENUM_AVAILABLE(10_6,3_0) = 0x02,
    NSCountResultType                NS_ENUM_AVAILABLE(10_6,3_0) = 0x04
};
*/
@property (nonatomic) NSFetchRequestResultType resultType;
//设置查询结果是否包含子实体
@property (nonatomic) BOOL includesSubentities;
//设置要查询的属性值
@property (nullable, nonatomic, copy) NSArray *propertiesToFetch;
```

在SchoolClass实体中查询数据，使用如下的代码：

```
    //创建一条查询请求
    NSFetchRequest * request = [NSFetchRequest fetchRequestWithEntityName:@"SchoolClass"];
    //设置条件为 stuNum=60的数据
    [request setPredicate:[NSPredicate predicateWithFormat:@"stuNum == 60"]];
    //进行查询操作
    NSArray * res = [moc executeFetchRequest:request error:nil];
    NSLog(@"%@",[res.firstObject stuNum]);
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

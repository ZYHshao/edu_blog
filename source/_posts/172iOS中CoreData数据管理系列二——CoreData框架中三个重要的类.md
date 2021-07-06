---
title: iOS中CoreData数据管理系列二——CoreData框架中三个重要的类
date: 2016-01-28
categories: iOS逻辑初窥
tags: []
---
## iOS中CoreData数据管理系列二——CoreData框架中三个重要的类

### 一、引言

    在上一篇博客中，介绍了iOS中使用CoreData框架设计数据模型的相关步骤。CoreData框架中通过相关的类将数据——数据模型——开发者无缝的衔接起来。NSManagedObjectModel对应数据模型，即上篇博客中我们创建的.xcdatamodeld文件；NSPersistentStoreCoordinator相当于数据库与数据模型之间的桥接器，通过NSPersistentStoreCoordinator将数据模型存入数据库；NSManagedObjectContext是核心的数据管理类，开发者通过操作它来执行对数据的相关操作。

### 二、数据模型管理类NSManagedObjectModel

    通过NSManagedObjectModel，可以将创建的数据模型文件读取为模型管理类对象，使用如下方法：

```
    //获取.xcdatamodeld文件url
    NSURL *modelUrl = [[NSBundle mainBundle]URLForResource:@"Model" withExtension:@"momd"];
    //读取文件
    NSManagedObjectModel * mom = [[NSManagedObjectModel alloc]initWithContentsOfURL:modelUrl];
```

其中还有一些属性和方法进行数据模型的管理：

```
//将多个数据模型管理文件进行合并
+ (nullable NSManagedObjectModel *)mergedModelFromBundles:(nullable NSArray<NSBundle *> *)bundles;  
//将多个数据模型管理类对象进行合并 
+ (nullable NSManagedObjectModel *)modelByMergingModels:(nullable NSArray<NSManagedObjectModel *> *)models;
//存放数据中所有实体模型的字典 字典中是实体名和实体描述对象
@property (readonly, copy) NSDictionary<NSString *, NSEntityDescription *> *entitiesByName;
//存放数据中所有实体描述对象
@property (strong) NSArray<NSEntityDescription *> *entities;
//返回所有可用的配置名称
@property (readonly, strong) NSArray<NSString *> *configurations;
//获取关联某个配置的所有实体
- (nullable NSArray<NSEntityDescription *> *)entitiesForConfiguration:(nullable NSString *)configuration;
//为某个实体关联配置
- (void)setEntities:(NSArray<NSEntityDescription *> *)entities forConfiguration:(NSString *)configuration;
//创建请求模板
- (void)setFetchRequestTemplate:(nullable NSFetchRequest *)fetchRequestTemplate forName:(NSString *)name;
//获取请求模板
- (nullable NSFetchRequest *)fetchRequestTemplateForName:(NSString *)name;
```

关于实体描述对象NSEntityDescription：

实体类似于数据库中的表结构，例如上次我们创建的班级实体模型，一个实体模型中可以添加许多属性与关系，NSEntityDescription对象中存放这些信息，常用如下：

```
//实体所在的模型管理对象
@property (readonly, assign) NSManagedObjectModel *managedObjectModel;
//实体所在的模型管理对象的名称
@property (null_resettable, copy) NSString *managedObjectClassName;
//实体名
@property (nullable, copy) NSString *name;
//设置是否是抽象实体
@property (getter=isAbstract) BOOL abstract;
//子类实体字典
@property (readonly, copy) NSDictionary<NSString *, NSEntityDescription *> *subentitiesByName;
//所有子类实体对象数组
@property (strong) NSArray<NSEntityDescription *> *subentities;
//父类实体
@property (nullable, readonly, assign) NSEntityDescription *superentity;
//所有属性字典
@property (readonly, copy) NSDictionary<NSString *, __kindof NSPropertyDescription *> *propertiesByName;
//所有属性数组 
@property (strong) NSArray<__kindof NSPropertyDescription *> *properties;
//所有常类型属性
@property (readonly, copy) NSDictionary<NSString *, NSAttributeDescription *> *attributesByName;
//所有关系
@property (readonly, copy) NSDictionary<NSString *, NSRelationshipDescription *> *relationshipsByName;
//某个实体类型的所有关系
- (NSArray<NSRelationshipDescription *> *)relationshipsWithDestinationEntity:(NSEntityDescription *)entity;
//判断是否是某种实体
- (BOOL)isKindOfEntity:(NSEntityDescription *)entity;
```

NSPropertyDescription类是数据模型属性的父类，NSAttributeDescription和NSRelationshipDescription都是继承于NSPropertyDescription类，NSAttributeDescription描述正常类型的属性，NSRelationshipDescription用于描述自定义类型的关系。

### 三、持久化存储协调者类NSPersistentStoreCoordinator

    NSPersistentStoreCoordinator建立数据模型与本地文件或数据库之间的联系，通过它将本地数据读入内存或者将修改过的临时数据进行持久化的保存。其初始化与链接数据持久化接收对象方法如下：

```
//通过数据模型管理对象进行初始化
- (instancetype)initWithManagedObjectModel:(NSManagedObjectModel *)model；
//添加一个持久化的数据接收对象
- (nullable __kindof NSPersistentStore *)addPersistentStoreWithType:(NSString *)storeType configuration:(nullable NSString *)configuration URL:(nullable NSURL *)storeURL options:(nullable NSDictionary *)options error:(NSError **)error;
//移除一个持久化的数据接收对象
- (BOOL)removePersistentStore:(NSPersistentStore *)store error:(NSError **)error;
```

### 四、数据对象管理上下文NSManagedObjectContext

    NSManagedObjectContext是进行数据管理的核心类，我们通过这个类来进行数据的增删改查等操作。其中常用方法如下：

```
//初始化方法 通过一个并发类型进行初始化 参数枚举如下：
/*
typedef NS_ENUM(NSUInteger, NSManagedObjectContextConcurrencyType) {
    NSPrivateQueueConcurrencyType        = 0x01,//上下文对象与私有队列关联
    NSMainQueueConcurrencyType            = 0x02//上下文对象与主队列关联
};
*/
- (instancetype)initWithConcurrencyType:(NSManagedObjectContextConcurrencyType)ct;
//异步执行block
- (void)performBlock:(void (^)())block;
//同步执行block
- (void)performBlockAndWait:(void (^)())block;
//关联数据持久化对象
@property (nullable, strong) NSPersistentStoreCoordinator *persistentStoreCoordinator;
//是否有未提交的更改
@property (nonatomic, readonly) BOOL hasChanges;
//进行查询数据请求
- (nullable NSArray *)executeFetchRequest:(NSFetchRequest *)request error:(NSError **)error;
//进行查询数据条数请求
- (NSUInteger) countForFetchRequest: (NSFetchRequest *)request error: (NSError **)error ; 
//插入元素
- (void)insertObject:(NSManagedObject *)object;
//删除元素
- (void)deleteObject:(NSManagedObject *)object;
//回滚一步操作
- (void)undo;
//清楚缓存
- (void)reset;
//还原数据
- (void)rollback;
//提交保存数据
- (BOOL)save:(NSError **)error;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

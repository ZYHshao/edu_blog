---
title: 使用iOS原生sqlite3框架对sqlite数据库进行操作
date: 2016-01-13
categories: iOS逻辑初窥
tags: []
---
## 使用iOS原生sqlite3框架对sqlite数据库进行操作

### 一、引言

      sqlite数据库是一种小型数据库，由于其小巧与简洁，在移动开发领域应用深广，sqlite数据库有一套完备的sqlite语句进行管理操作，一些常用的语句和可视化的开发工具在上篇博客中有介绍，地址如下：

sqlite数据库常用语句及可视化工具介绍：[http://my.oschina.net/u/2340880/blog/600820](http://my.oschina.net/u/2340880/blog/600820)。

      在iOS的原生开发框架中可以对sqlite数据库进行很好的支持，这个框架中采用C风格且通过指针移动进行数据的操作，使用起来有些不便，我们可以对一些数据库的常用操作进行一些面向对象的封装。

### 二、libsqlite3系统库中操作数据库的常用方法

    libsqlite3是对sqlite数据库进行操作的系统库，在使用前，我们需要先导入，点击Xcode的Build Phases标签，展开Link Binary With Libraries，点击+号，在弹出的窗口中搜索libsqlite3.0，将其导入进工程，过程如下图：

![](http://static.oschina.net/uploads/space/2016/0113/114916_Z4xS_2340880.png)

![](http://static.oschina.net/uploads/space/2016/0113/114917_VKDz_2340880.png)

在需要操作sqlite数据的文件中导入如下头文件：

```
#import <sqlite3.h>
```

数据库文件的操作是由一个sqlite3类型的指针操作管理的，如下方法进行数据库的打开：

```
sqlite3 *sqlite；
sqlite3_open(dataBaePath, &sqlite)
```

sqlite3_open方法返回一个int值，实际上，在使用libsqlite3框架中的大多方法时都会返回一个int值，这个int值代表着方法执行的相应结果状态，这些状态再sqlite3.h文件中通过宏来定义，列举如下：

```
#define SQLITE_OK           0   //操作成功
/* 以下是错误代码 */
#define SQLITE_ERROR        1   /* SQL数据库错误或者丢失*/
#define SQLITE_INTERNAL     2   /* SQL内部逻辑错误 */
#define SQLITE_PERM         3   /* 没有访问权限 */
#define SQLITE_ABORT        4   /* 回调请求终止 */
#define SQLITE_BUSY         5   /* 数据库文件被锁定 */
#define SQLITE_LOCKED       6   /* 数据库中有表被锁定 */
#define SQLITE_NOMEM        7   /* 分配空间失败 */
#define SQLITE_READONLY     8   /* 企图向只读属性的数据库中做写操作 */
#define SQLITE_INTERRUPT    9   /* 通过sqlite3_interrupt()方法终止操作*/
#define SQLITE_IOERR       10   /* 磁盘发生错误 */
#define SQLITE_CORRUPT     11   /* 数据库磁盘格式不正确 */
#define SQLITE_NOTFOUND    12   /* 调用位置操作码 */
#define SQLITE_FULL        13   /* 由于数据库已满造成的添加数据失败 */
#define SQLITE_CANTOPEN    14   /* 不法打开数据库文件 */
#define SQLITE_PROTOCOL    15   /* 数据库锁协议错误 */
#define SQLITE_EMPTY       16   /* 数据库为空 */
#define SQLITE_SCHEMA      17   /* 数据库模式更改 */
#define SQLITE_TOOBIG      18   /* 字符或者二进制数据超出长度 */
#define SQLITE_CONSTRAINT  19   /* 违反协议终止 */
#define SQLITE_MISMATCH    20   /* 数据类型不匹配 */
#define SQLITE_MISUSE      21   /* 库使用不当 */
#define SQLITE_NOLFS       22   /* 使用不支持的操作系统 */
#define SQLITE_AUTH        23   /* 授权拒绝 */
#define SQLITE_FORMAT      24   /* 辅助数据库格式错误 */
#define SQLITE_RANGE       25   /* sqlite3_bind 第二个参数超出范围 */
#define SQLITE_NOTADB      26   /* 打开不是数据库的文件 */
#define SQLITE_NOTICE      27   /* 来自sqlite3_log()的通知 */
#define SQLITE_WARNING     28   /* 来自sqlite3_log() 的警告*/
#define SQLITE_ROW         100  /* sqlite3_step() 方法准备好了一行数据 */
#define SQLITE_DONE        101  /* sqlite3_step() 已完成执行*/
```

执行非查询类的语句，例如创建，添加，删除等操作，使用如下方法：

```
char * err;
sqlite3 *sql;
sqlite3_exec(sql, sqlStr, NULL, NULL, &err);
```

sqlite3_exec方法中第一个参数为成功执行了打开数据库操作的sqlite3指针，第二个参数为要执行的sql语句，最后一个参数为错误信息字符串。

执行查询语句的方法比较复杂，通过如下方法:

```
    sqlite3 * sqlite;
    sqlite3_stmt *stmt =nil;
    int code = sqlite3_prepare_v2(sqlite, sqlStr, -1, &stmt, NULL);
     while (sqlite3_step(stmt)==SQLITE_ROW) {
         char * cString =(char*)sqlite3_column_text(stmt, 0);
         NSString * value = [NSString stringWithCString:cString?cString:"NULL" encoding:NSUTF8StringEncoding];
         NSNumber * value = [NSNumber numberWithLongLong:sqlite3_column_int64(stmt, 1)];
        }
         sqlite3_finalize(stmt);
```

stmt是一个数据位置指针，标记查询到数库的数据位置，sqlite3\_prepare\_v2()方法进行数据库查询的准备工作，第一个参数为成功打开的数据库指针，第二个参数为要执行的查询语句，第三个参数为sqlite3_stmt指针的地址，这个方法也会返回一个int值，作为标记状态是否成功。

sqlite3\_step方法对stmt指针进行移动，会逐行进行移动，这个方法会返回一个int值，如果和SQLITE\_ROW宏对应，则表明有此行数据，可以通过while循环来对数据进行读取。

sqlite3\_column\_XXX()是取行中每一列的数据，根据数据类型的不同，sqlite3\_column\_XXX()有一系列对应的方法，这个方法中第一个参数是stmt指针，第二个参数为列序号。

sqlite3_finalize()方法对stmt指针进行关闭。

### 三、面向对象的sqlite数据库操作框架封装

        网上不乏有许多优秀的第三方sqlite数据库使用框架，FFDM就是其中之一，并且apple自带的coreData也十分优秀。这篇博客中所述内容并不全面，代码也并不十分完善健壮，封装出来的代码除了能够完成基本的数据库操作外，更多主要是对设计思路的示例。

#### 1.面向对象的sqlite管理类的设计思路

        为了便于使用，在设计时，我们尽量将libsqlite3中的方法不暴漏在使用层，通过面向应用的接口来进行方法的设计，设计思路类图如下：

![](http://static.oschina.net/uploads/space/2016/0113/124507_DQiH_2340880.png)

图中，文件管理中心对文件进行存取删改管理，不暴漏在外，数据库管理中心负责对数据库的创建，删除打开等操作，具体的数据操作由数据库操作对象来完成。

#### 2.文件管理中心方法的编写

        文件管理中心主要负责对数据库文件的存取，可以实现如下方法：

YHBaseCecheCenter.h

```
/**
 *  @brief 获取数据库方法的地址
 *
 *  @return 地址字符串
 *
 */
-(NSString *)getDataBaseFilePath;
/**
 *  @brief 获取某个数据库的大小
 *
 *  @param name 数据库名称
 *
 *  @return 文件大小 单位M
 *
 */
-(float)getSizeFromDataBaseName:(NSString *)name;
```

YHBaseCecheCenter.m

```
-(NSString *)getDataBaseFilePath{
    return NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
}
-(float)getSizeFromDataBaseName:(NSString *)name{
    NSString * path = [NSString stringWithFormat:@"/%@/%@",NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject,name];
    return  [self fileSizeAtPath:path]/(1024.0*1024.0);
}
//获取文件大小
- (long long) fileSizeAtPath:(NSString*) filePath{
    NSFileManager* manager = [NSFileManager defaultManager];
    if ([manager fileExistsAtPath:filePath]){
        return [[manager attributesOfItemAtPath:filePath error:nil] fileSize];
    }
    return 0;
}
```

在iOS系统中因为其沙盒结构的限制，数据库必须方法documents目录下才能正常打开使用。

#### 3.数据库管理中心的设计

        数据库管理中心主要负责对数据库的宏观操作，采用类方法的设计模式，如下

YHBaseSQLiteManager.h

```
/**
 *  @brief 打开一个数据库 如果不存在则会创建
 *
 *  @param name 数据库名称
 *
 *  @return 数据库操作对象 如果创建失败会返回nil
 *
 */
+(YHBaseSQLiteContext *)openSQLiteWithName:(NSString *)name;
/**
 *  @brief 获取数据库文件的大小 单位M
 *
 *  @param dataBase 数据库上下文对象
 *
 *  @return 数据库文件大小
 */
+(float)getSizeOfDataBase:(YHBaseSQLiteContext *)dataBase;
/**
 *  @brief 获取数据库文件的大小 单位M
 *
 *  @param dataBaseName 数据库名称
 *
 *  @return 数据库文件大小
 */
+(float)getSizeOfDataBaseName:(NSString *)dataBaseName;
/**
 *  @brief 删除所有数据库
 *
 */
+(void)removeDataBase;
```

 YHBaseSQLiteManager.m

```
+(YHBaseSQLiteContext *)openSQLiteWithName:(NSString *)name{
    
    NSString * path =  [[YHBaseCecheCenter sharedTheSingletion]getDataBaseFilePath];
    YHBaseSQLiteContext * context = [[YHBaseSQLiteContext alloc]init];
    context.name = name;
    BOOL success = [context openDataBaeWithName:[NSString stringWithFormat:@"%@/%@",path,name]];
    if (success) {
        return context;
    }else{
        return nil;
    }
}
+(float)getSizeOfDataBase:(YHBaseSQLiteContext *)dataBase{
    return [[YHBaseCecheCenter sharedTheSingletion]getSizeFromDataBaseName:dataBase.name];
}
+(float)getSizeOfDataBaseName:(NSString *)dataBaseName{
    return [[YHBaseCecheCenter sharedTheSingletion]getSizeFromDataBaseName:dataBaseName];
}
+(void)removeDataBase{
    NSString * path =  [[YHBaseCecheCenter sharedTheSingletion]getDataBaseFilePath];
    return [[YHBaseCecheCenter sharedTheSingletion]removeCacheFromPath:path];
}
```

#### 4.数据库操作对象

        将操作数据库的核心方法封装在这个类中：

YHBaseSQLiteContext.h

```
/**
 *操作的数据库名称
 */
@property(nonatomic,strong)NSString * name;
/**
 *内含sqlite3 对象
 */
@property(nonatomic,assign)sqlite3 * sqlite3_db;
/**
 * @brief 打开一个数据库 不存在则创建
 *
 * @param path 数据库路径
 *
 * @return 是否操作成功
 */
-(BOOL)openDataBaeWithName:(NSString *)path;
/**
 *  @brief 再数据库中创建一张表 如果已经存在 会返回错误信息
 *
 *  @param name 表的名称
 *
 *  @prarm dic 表中的键 其中字典中需传入 键名：类型  类型的宏定义在YHBaseSQLTypeHeader.h中
 *
 *  @param callBack 结果回调
 */
-(void)createTableWithName:(NSString *)name
            keysDictionary:(NSDictionary<NSString*,NSString*> *) dic
                  callBack:(void (^)(YHBaseSQLError * error))complete;

/**
 *  @brief 向表中添加一条数据
 *
 *  @param dataDic 添加数据的键值对
 *
 *  @param name 插入表的名称
 *
 *  @complete 回调
 */
-(void)insertData:(NSDictionary<NSString *,id>*)dataDic
        intoTable:(NSString *)name
         callBack:(void (^)(YHBaseSQLError * error))complete;
/**
 *  @brief 向表中添加一个键
 *
 *  @param kName 添加的键
 *
 *  @prarm type 类型
 *
 *  @prarm tableName 表名称
 *
 *  @prarm complete 结果回调
 */
-(void)addKey:(NSString *)kName
      keyType:(NSString *)type
    intoTable:(NSString *)tableName
     callBack:(void(^)(YHBaseSQLError *error))complete;
/**
 *  @brief 修改数据
 *
 *  @param dataDic 新的键值
 *
 *  @param wlStr 条件字符串 一般通过主键找到对应数据修改 可以为nil
 *
 *  @param complete 结果回调
 */
-(void)update:(NSDictionary<NSString*,id> *)dataDic
      inTable:(NSString *)tableName
  whileString:(NSString *)wlStr
     callBack:(void(^)(YHBaseSQLError * error))complete;
/**
 *  @brief 删除数据
 *
 *  @param tableName 表名
 *
 *  @param wlStr 条件字符串 一般通过主键找到对应数据删除 可以为nil 不传这个参数将删除所有数据
 *
 */
-(void)deleteDataFromTable:(NSString *)tableName
               whereString:(NSString *)wlStr
                  callBack:(void(^)(YHBaseSQLError * error))complete;
/**
 *  @brief 删除一张表
 *
 *  @param tableName 表名
 *
 */
-(void)dropTable:(NSString *)tableName
        callBack:(void(^)(YHBaseSQLError * error))complete;
/**
 *  @brief 查询数据
 *
 *  @param keys 要查询的键值 及其对应的数据类型 可以为nil则查询全部
 *
 *  @param tableName 表名
 *
 *  @param orderKey 进行排序的键值 可以为nil 则不排序
 *
 *  @param type 排序方式 在YHBaseSQLTypeHeader中有宏定义
 *
 *  @param wlstr 查询条件 同于查询单个数据
 *
 *  @param complete dataArray为查询到的数据 其内为字典
 *
 */
-(void)selectKeys:(NSArray<NSDictionary *> *)keys
        fromTable:(NSString*)tableName
          orderBy:(NSString *)orderKey
        orderType:(NSString *)type
         whileStr:(NSString *)wlstr
         callBack:(void(^)(NSArray<NSDictionary *> * dataArray,YHBaseSQLError * error))complete;
/**
 *  @brief 关闭数据库上下文操作
 *  调用此方法后 这个context对象将不再有效 如果再需要使用 需要YHBaseSQLiteManager中的类方法再次返回
 */
-(void)closeContext;
```

YHBaseSQLiteContext.m

```
-(BOOL)openDataBaeWithName:(NSString *)path{
    if (sqlite3_open([path UTF8String], &_sqlite3_db)!=SQLITE_OK) {
        sqlite3_close(_sqlite3_db);
        _sqlite3_db=nil;
        return NO;
    }else{
        return YES;
    }
}
-(void)createTableWithName:(NSString *)name keysDictionary:(NSDictionary<NSString *,NSString *> *)dic callBack:(void (^)(YHBaseSQLError *))complete{
    NSMutableString * keys = [[NSMutableString alloc]init];
    for (int i=0; i<dic.allKeys.count; i++) {
        NSString * key = dic.allKeys[i];
        if (i<dic.allKeys.count-1) {
            [keys appendFormat:@"%@ %@,",key,[dic objectForKey:key]];
        }else{
            [keys appendFormat:@"%@ %@",key,[dic objectForKey:key]];
        }
    }
    NSString * sqlStr = [NSString stringWithFormat:@"create table %@(%@)",name,keys];
    [self runSQL:sqlStr callBack:^(YHBaseSQLError * error) {
        
        if (complete) {
            complete(error);
        }
        
    }];
}
-(void)insertData:(NSDictionary<NSString *,id> *)dataDic intoTable:(NSString *)name callBack:(void (^)(YHBaseSQLError *))complete{
    NSMutableString * keys = [[NSMutableString alloc]init];
    NSMutableString * values = [[NSMutableString alloc]init];
    for (int i=0; i<dataDic.allKeys.count; i++) {
        NSString * key = dataDic.allKeys[i];
        if (i<dataDic.count-1) {
            [keys appendFormat:@"%@,",key];
            [values appendFormat:@"\"%@\",",[dataDic objectForKey:key]];
        }else{
            [keys appendFormat:@"%@",key];
            [values appendFormat:@"\"%@\"",[dataDic objectForKey:key]];
        }
    }
    NSString * sqlStr = [NSString stringWithFormat:@"insert into %@(%@) values(%@)",name,keys,values];
    [self runSQL:sqlStr callBack:^(YHBaseSQLError *error) {
        
        if (complete) {
            complete(error);
        }
        
    }];
}
-(void)addKey:(NSString *)kName keyType:(NSString *)type intoTable:(NSString *)tableName callBack:(void (^)(YHBaseSQLError *))complete{
    NSString * sqlStr = [NSString stringWithFormat:@"alter table %@ add %@ %@",tableName,kName,type];
    [self runSQL:sqlStr callBack:^(YHBaseSQLError *error) {
        if (complete) {
            complete(error);
        }
    }];
}
-(void)update:(NSDictionary<NSString *,id> *)dataDic inTable:(NSString *)tableName whileString:(NSString *)wlStr callBack:(void (^)(YHBaseSQLError *))complete{
    NSMutableString * sqlStr = [[NSMutableString alloc]init];
    [sqlStr appendFormat:@"update %@ set ",tableName];
    for (int i=0; i<dataDic.allKeys.count; i++) {
        NSString * key = dataDic.allKeys[i];
        if (i<dataDic.allKeys.count-1) {
            [sqlStr appendFormat:@"%@=\"%@\",",key,[dataDic objectForKey:key]];
        }else{
            [sqlStr appendFormat:@"%@=\"%@\"",key,[dataDic objectForKey:key]];
            if (wlStr!=nil) {
                [sqlStr appendFormat:@" where %@",wlStr];
            }
        }
    }
    [self runSQL:sqlStr callBack:^(YHBaseSQLError *error) {
        if (complete) {
            complete(error);
        }
    }];
}


-(void)deleteDataFromTable:(NSString *)tableName whereString:(NSString *)wlStr callBack:(void (^)(YHBaseSQLError *))complete{
    NSMutableString * sqlStr = [[NSMutableString alloc]init];
    [sqlStr appendFormat:@"delete from %@",tableName];
    if (wlStr!=nil) {
        [sqlStr appendFormat:@" where %@",wlStr];
    }
    [self runSQL:sqlStr callBack:^(YHBaseSQLError *error) {
        if (complete) {
            complete(error);
        }
    }];
}
-(void)dropTable:(NSString *)tableName callBack:(void (^)(YHBaseSQLError *))complete{
    NSString * sqlStr = [NSString stringWithFormat:@"drop table %@",tableName];
    [self runSQL:sqlStr callBack:^(YHBaseSQLError *error) {
        if (complete) {
            complete(error);
        }
    }];
}
-(void)selectKeys:(NSArray<NSDictionary *> *)keys fromTable:(NSString *)tableName orderBy:(NSString *)orderKey orderType:(NSString *)type whileStr:(NSString *)wlstr callBack:(void (^)(NSArray<NSDictionary *> *, YHBaseSQLError *))complete{
    NSMutableString * sqlStr = [[NSMutableString alloc]init];
    [sqlStr appendFormat:@"select"];
    if (keys==nil||keys.count==0) {
        [sqlStr appendFormat:@" * from %@",tableName];
    }else{
        for (int i=0; i<keys.count; i++) {
            if (i<keys.count-1) {
                [sqlStr appendFormat:@" %@,",keys[i].allKeys.firstObject];
            }else{
                [sqlStr appendFormat:@" %@ from %@",keys[i].allKeys.firstObject,tableName];
            }
            
        }
    }
    if (wlstr) {
        [sqlStr appendFormat:@" where %@",wlstr];
    }
    if (orderKey) {
        [sqlStr appendFormat:@" order by %@",orderKey];
    }
    if (type) {
        [sqlStr appendFormat:@" %@",type];
    }
    NSMutableArray * keysArr = [[NSMutableArray alloc]init];
    NSMutableArray * keysTypeArr = [[NSMutableArray alloc]init];
    if (keys==nil||keys.count==0) {
        NSArray<NSDictionary *> * tmpArr = [self getTheTableAllKeys:tableName];
        for (int i=0; i<tmpArr.count; i++) {
            NSString * key = tmpArr[i].allKeys.firstObject;
            [keysArr addObject:key];
            [keysTypeArr addObject:[tmpArr[i] objectForKey:key]];
        }
    }else{
        for (int i=0; i<keys.count; i++) {
            NSString * key = keys[i].allKeys.firstObject;
            [keysArr addObject:key];
            [keysTypeArr addObject:[keys[i] objectForKey:key]];
        }
    }
    
    [self runSelectSQL:sqlStr withKeys:keysArr withDataType:keysTypeArr callBack:^(NSArray<NSDictionary *> *dataArray, YHBaseSQLError *error) {
        if (complete) {
            complete(dataArray,error);
        }
    }];
   
}
-(void)closeContext{
    sqlite3_close(_sqlite3_db);
    _sqlite3_db = nil;
}

//内部方法 运行创建独立的非查询SQL语句
-(void)runSQL:(NSString *)sql callBack:(void(^)(YHBaseSQLError * error))complete{
    char * err;
    int code = sqlite3_exec(_sqlite3_db, [sql UTF8String], NULL, NULL, &err);
    if (code!=SQLITE_OK) {
        YHBaseSQLError * error = [[YHBaseSQLError alloc]init];
        error.errorInfo = [NSString stringWithCString:err encoding:NSUTF8StringEncoding];
        error.errorCode = code;
        complete(error);
    }else{
        complete(nil);
    }
}
//运行查询语句
-(void)runSelectSQL:(NSString *)sql withKeys:(NSArray *)keys withDataType:(NSArray *)dataType callBack:(void(^)(NSArray<NSDictionary *> * dataArray, YHBaseSQLError * error))complete{
    sqlite3_stmt *stmt =nil;
    int code = sqlite3_prepare_v2(_sqlite3_db, [sql UTF8String], -1, &stmt, NULL);
    if (code!=SQLITE_OK) {
        YHBaseSQLError * error = [[YHBaseSQLError alloc]init];
        error.errorInfo = @"查询失败";
        error.errorCode=code;
        complete(nil,error);
    }else{
        NSMutableArray * resultArray = [[NSMutableArray alloc]init];
        
        while (sqlite3_step(stmt)==SQLITE_ROW) {
            //数据类型的分别解析
            NSMutableDictionary * dic = [[NSMutableDictionary alloc]init];
            for (int i=0; i<dataType.count; i++) {
                NSString * type = dataType[i];
                if ([type isEqualToString:YHBASE_SQL_DATATYPE_BINARY]) {
                    int length = sqlite3_column_bytes(stmt, i);
                    const void *data = sqlite3_column_blob(stmt, i);
                    NSData * value = [NSData dataWithBytes:data length:length];
                    [dic  setObject:value forKey:keys[i]];
                }else if([type isEqualToString:YHBASE_SQL_DATATYPE_BLOB]){
                    int length = sqlite3_column_bytes(stmt, i);
                    const void *data = sqlite3_column_blob(stmt, i);
                    NSData * value = [NSData dataWithBytes:data length:length];
                    [dic  setObject:value forKey:keys[i]];
                }else if([type isEqualToString:YHBASE_SQL_DATATYPE_BOOLEAN]){
                    NSNumber * value = [NSNumber numberWithInt:sqlite3_column_int(stmt, i)];
                    [dic setObject:value forKey:keys[i]];
                }else if([type isEqualToString:YHBASE_SQL_DATATYPE_CURRENCY]){
                    NSNumber * value = [NSNumber numberWithLong:sqlite3_column_int64(stmt, i)];
                    [dic setObject:value forKey:keys[i]];
                }else if([type isEqualToString:YHBASE_SQL_DATATYPE_DATE]){
                    char * cString =(char*)sqlite3_column_text(stmt, i);
                    NSString * value = [NSString stringWithCString:cString?cString:"NULL" encoding:NSUTF8StringEncoding];
                    [dic setObject:value forKey:keys[i]];
                }else if([type isEqualToString:YHBASE_SQL_DATATYPE_DOUBLE]){
                    NSNumber * value = [NSNumber numberWithFloat:sqlite3_column_double(stmt, i)];
                    [dic setObject:value forKey:keys[i]];
                }else if([type isEqualToString:YHBASE_SQL_DATATYPE_FLOAT]){
                    NSNumber * value = [NSNumber numberWithFloat:sqlite3_column_double(stmt, i)];
                    [dic setObject:value forKey:keys[i]];
                }else if([type isEqualToString:YHBASE_SQL_DATATYPE_INTRGER]){
                    
                    NSNumber * value = [NSNumber numberWithInt:sqlite3_column_int(stmt, i)];
                    [dic setObject:value forKey:keys[i]];
                }else if([type isEqualToString:YHBASE_SQL_DATATYPE_REAL]){
                    NSNumber * value = [NSNumber numberWithDouble:sqlite3_column_int(stmt, i)];
                    [dic setObject:value forKey:keys[i]];
                }else if([type isEqualToString:YHBASE_SQL_DATATYPE_SMALLINT]){
                    NSNumber * value = [NSNumber numberWithShort:sqlite3_column_int(stmt, i)];
                    [dic setObject:value forKey:keys[i]];
                }else if([type isEqualToString:YHBASE_SQL_DATATYPE_TEXT]){
                    char * cString =(char*)sqlite3_column_text(stmt, i);
                    NSString * value = [NSString stringWithCString:cString?cString:"NULL" encoding:NSUTF8StringEncoding];
                    [dic setObject:value forKey:keys[i]];
                }else if([type isEqualToString:YHBASE_SQL_DATATYPE_TIME]){
                    char * cString =(char*)sqlite3_column_text(stmt, i);
                    NSString * value = [NSString stringWithCString:cString?cString:"NULL" encoding:NSUTF8StringEncoding];
                    [dic setObject:value forKey:keys[i]];
                }else if([type isEqualToString:YHBASE_SQL_DATATYPE_TIMESTAMP]){
                    NSNumber * value = [NSNumber numberWithLongLong:sqlite3_column_int64(stmt, i)];
                    [dic setObject:value forKey:keys[i]];
                }else if([type isEqualToString:YHBASE_SQL_DATATYPE_VARCHAR]){
                    char * cString =(char*)sqlite3_column_text(stmt, i);
                    NSString * value = [NSString stringWithCString:cString?cString:"NULL" encoding:NSUTF8StringEncoding];
                    [dic setObject:value forKey:keys[i]];
                }
               
            }
             [resultArray addObject:dic];
        }
         sqlite3_finalize(stmt);
        stmt=nil;
        complete(resultArray,nil);
    }
}
//获取表中所有字段名和类型
-(NSArray<NSDictionary *> *)getTheTableAllKeys:(NSString *)tableName{
    NSMutableArray * array = [[NSMutableArray alloc]init];
    NSString * getColumn = [NSString stringWithFormat:@"PRAGMA table_info(%@)",tableName];
    sqlite3_stmt *statement;
    sqlite3_prepare_v2(_sqlite3_db, [getColumn UTF8String], -1, &statement, nil);
    while (sqlite3_step(statement) == SQLITE_ROW) {
        char *nameData = (char *)sqlite3_column_text(statement, 1);
        NSString *columnName = [[NSString alloc] initWithUTF8String:nameData];
        char *typeData = (char *)sqlite3_column_text(statement, 2);
        NSString *columntype = [NSString stringWithCString:typeData encoding:NSUTF8StringEncoding];
        NSDictionary * dic = @{columnName:columntype};
        [array addObject:dic];
    }
     sqlite3_finalize(statement);
    statement=nil;
    return array;
}
```

#### 5.错误信息类可以将数据库操作中的异常抛出提示开发者

YHBaseSQLError.h

```
/**
 *异常的提示信息
 */
__PROPERTY_NO_STRONG__(NSString *, errorInfo);
/**
 *异常的对应code码
 */
__PROPERTY_NO_ASSIGN__(NSInteger, errorCode);
```

还有一个头文件中定义了sqlite数据库支持的数据类型和排序宏定义：

YHBaseSQLTypeHeader.h

```
#define YHBASE_SQL_DATATYPE_SMALLINT @"smallint" //short
#define YHBASE_SQL_DATATYPE_INTRGER @"integer"    //int
#define YHBASE_SQL_DATATYPE_REAL @"real"          //实数
#define YHBASE_SQL_DATATYPE_FLOAT @"float"        //float
#define YHBASE_SQL_DATATYPE_DOUBLE @"double"      //double
#define YHBASE_SQL_DATATYPE_CURRENCY @"currency"  //long
#define YHBASE_SQL_DATATYPE_VARCHAR @"varchar"    //char
#define YHBASE_SQL_DATATYPE_TEXT @"text"          //string
#define YHBASE_SQL_DATATYPE_BINARY @"binary"      //二进制
#define YHBASE_SQL_DATATYPE_BLOB @"blob"          //长二进制
#define YHBASE_SQL_DATATYPE_BOOLEAN @"boolean"    //bool
#define YHBASE_SQL_DATATYPE_DATE @"date"          //日期
#define YHBASE_SQL_DATATYPE_TIME @"time"          //时间
#define YHBASE_SQL_DATATYPE_TIMESTAMP @"timestamp"//时间戳

#define YHBASE_SQL_ORDERTYPE_ASC @"asc" //升序
#define YHBASE_SQL_ORDERTYPE_DESC @"desc" //降序
```

### 四、使用

        在使用时，直接调用context的相应方法操作数据库即可，例如：

```
YHBaseSQLiteContext * context = [YHBaseSQLiteManager openSQLiteWithName:@"testDataBase"];
    if (context) {
        [context selectKeys:nil fromTable:@"MySQL" orderBy:@"age" orderType:YHBASE_SQL_ORDERTYPE_DESC whileStr:@"age>18" callBack:^(NSArray<NSDictionary *> *dataArray, YHBaseSQLError *error) {
            NSLog(@"%@",dataArray);
            NSLog(@"%@",error.errorInfo);
            [context closeContext];
        }];
    }
```

上面的代码将查询textDataBase数据库中MySQL表里所有age列大于18的数据，并按照age从小到大进行排序，数据结果在回调的dataArray中。

外：完整的代码在下面的git地址中，这个git项目是一个基础的开发框架，里面封装了许多开发和调试常用功能，代码不完善之处，希望多多交流，QQ316045346.

git：[https://github.com/ZYHshao/YHBaseFoundationTest](https://github.com/ZYHshao/YHBaseFoundationTest)。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

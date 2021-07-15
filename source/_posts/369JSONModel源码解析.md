---
title: JSONModel源码解析
date: 2018-03-30
categories: iOS第三方库
tags: []
---
## JSONModel源码解析

### 一、引言

    做移动端开发，解析网络数据是必不可少的工作之一。iOS原生框架很早前就已经提供了将JSON数据直接映射成数组或者字典对象的方法，并且结合KVC，也可以将字典数据直接赋值给对象。但是这种方式十分不灵活，例如如果网络数据中的字段与我们数据模型中的字段不一致，某些网络数据的字段可能为nil等等都需要开发者单独的处理。使用JSOMModel可以十分方便的处理映射过程中的各种情况。

### 二、JSOMModel类概览

    平时在使用JSOMModel框架时，往往只会用到JSOMModel这一个类，其实JSOMModel中还封装了一套网络请求逻辑，你可以直接对某个对象调用请求来映射成为数据模型。但是我建议尽量将数据的请求和解析分开来做，这样更利于请求的维护(在新的JSOMModel版本中，也将有关网络请求的部分标记为了弃用)。JSOMModel功能十分强大，代码量却并不特别多，结构逻辑也较为清晰。其中类的关系和结构如下图表示。

![](https://static.oschina.net/uploads/space/2018/0330/144220_O4Rh_2340880.png)

如上图所示，其中网络相关模块已经弃用，并且也不是JSONModel的核心模块，不在本次博客的探讨范围之内。JSONModelError定义了许多错误类型，主要用来当请求或数据解析异常时进行抛出，需要注意，JSONModel定义的自己的log函数，其只会在模拟器运行时进行打印。

### 三、JSONModelClassProperty类的意义

    将网络数据映射为Model模型的实质即是对Model对象中属性的赋值，在JSONModel中，类的属性被抽象为JSONModelClassProperty对象，这个对象中封装这此属性的相关信息(通过runtime来动态生成)。JSONModelClassProerty类中的属性意义如下：

```objectivec
@interface JSONModelClassProperty : NSObject
//已经弃用  这个用来标识当前属性是否是对象的主键 用来进行数据模型的比较
@property (assign, nonatomic) BOOL isIndex DEPRECATED_ATTRIBUTE;
//属性名
@property (copy, nonatomic) NSString *name;
//属性类型
@property (assign, nonatomic) Class type;
//属性结构体名称 基本数据类型的属性 会被抽象成结构体
@property (strong, nonatomic) NSString *structName;
//属性遵守的协议名
@property (copy, nonatomic) NSString *protocol;
//当前属性是否是可选属性 如果是 在解析时允许这个属性值为nil
@property (assign, nonatomic) BOOL isOptional;
//是否是标准的json数据，如果是则不用再调用数据转换的方法
@property (assign, nonatomic) BOOL isStandardJSONType;
//当前属性是否是可变的 如果是 则会创建可变对象
@property (assign, nonatomic) BOOL isMutable;
//自定义的 属性get函数
@property (assign, nonatomic) SEL customGetter;
//自定义的属性 set函数
@property (strong, nonatomic) NSMutableDictionary *customSetters;
@end
```

### 四、关于属性映射器JSONKeyMapper

    简单理解，JSONKeyMapper属性映射器的作用就是用来制定在数据解析时所遵循的规则。最理想的情况是JSON数据与我们要解析成的数据模型完全对应，例如：

JSON数据：

```json
{ "firstName":"Bill" , "lastName":"Gates" }
```

数据Model:

```objectivec
@interface MyOnject : NSObject

@property(nonatomic,strong)NSString * firstName;

@property(nonatomic,strong)NSString * lastName;

@end

```

然而在实际的开发中，这种完美的情况却很少出现，我们更多遇到的是，JSON数据中某些字段可能有也可能无，数据Model中需要增加些本地字段，JSON数据和Model的某些字段名称可能不一致。更加复杂一点，我们可以Model的某个属性是另一个Model。或者某个属性是数组，数组中存放的是另一种Model。

    JSONKeyMapper接口定义如下：

```objectivec
//通过字典来创建映射器  字典的键为数据Model的属性名  值为JSOM数据的属性名 
- (instancetype)initWithModelToJSONDictionary:(NSDictionary *)toJSON;

//通过block来建立映射关系 block的定义如下，其中会将JSOM数据的属性名传入 需要返回要对应Model的属性名
/*
typedef NSString *(^JSONModelKeyMapBlock)(NSString *keyName);
*/ 
- (instancetype)initWithModelToJSONBlock:(JSONModelKeyMapBlock)toJSON;
//创建一个 将以下划线分割的命名键 转换成驼峰命名 例如 first_name => firstName
+ (instancetype)mapperForSnakeCase;
//创建一个 将首字母大写的明明键 转换成驼峰  例如 FirstName => firstName
+ (instancetype)mapperForTitleCase;
```

### 五、核心数据模型类JSONModel

    JSONModel框架中最核心的类JSONModel类，其中代码大约有1400行，除了一些调试，复写和提供方便功能的代码外，核心代码在800行左右。首先，其头文件中声明了几个协议，如下：

```objectivec
@protocol Index
@end
@protocol Ignore
@end
@protocol Optional
@end
```

需要注意，这些协议里面都没有约定任何方法，它们也不会用来实现的，其作为属性的一种标记，例如将属性添加Ignore协议，则JSONModel不会对这个属性进行解析，使用这种方式来进行本地数据的管理，例如：

```objectivec
@interface MyOnject : JSONModel

@property(nonatomic,strong)NSString * firstName;

@property(nonatomic,strong)NSString * lastName;
//这个属性是本地拼接 使用
@property(nonatomic,strong)NSString<Ignore> * fullName;

@end

```

Optional协议表示这个属性是可选的，即JSON数据中如果有这个属性就解析，如果没有就跳过。Index协议标记这个属性是当前对象的主键，已经弃用。

    有了这3个协议，在声明属性时，我们可以十分容易的设定他们的解析规则，在JSONModel中，协议除了可以用来规定解析规则外，还可以用来指定自定义数据类型的解析，只是我们需要自己定义一个协议，名称与自定义类名一致，示例如下：

```objectivec
#import "JSONModel.h"

@protocol Address
@end

@interface Address:JSONModel

@property(nonatomic,strong)NSString * info;

@end

@interface MyObject : JSONModel

@property(nonatomic,strong)NSString<Optional> * firstName;

@property(nonatomic,strong)NSString * lastName;

@property(nonatomic,strong)NSArray<Address> * address;

@end

```

如上代码所以，在解析数据时，会直接将address数组中赋值为Address的对象，当前也可以直接解析对象，例如：

```objectivec
@protocol Address
@end

@interface Address:JSONModel

@property(nonatomic,strong)NSString * info;

@end



@interface MyObject : JSONModel

@property(nonatomic,strong)NSString<Optional> * firstName;

@property(nonatomic,strong)NSString * lastName;

@property(nonatomic,strong)Address<Address> * address;

@end
```

需要注意，在Objective-C中，只有NSObject的子类可以遵守协议，原始数据类型是不能遵守协议的，那么对于类似BOOL，int这样的属性有没有办法设置他们的忽略解析或者可选解析呢，当然也可以，我们可以通过重写JSONModel中的一些函数来实现，这种方法更加通用，JSONModel类接口意义如下：

```objectivec
//将JSON字符串解析成数据模型对象
- (instancetype)initWithString:(NSString *)string error:(JSONModelError **)err;
- (instancetype)initWithString:(NSString *)string usingEncoding:(NSStringEncoding)encoding error:(JSONModelError **)err;
//将数据模型对象转换成JSON字符串
- (NSString *)toJSONString;
//将数据模型对象转换成JSON数据
- (NSData *)toJSONData;
//将数据模型对象中的某些键组合成JSON字符串
- (NSString *)toJSONStringWithKeys:(NSArray *)propertyNames;
//将数据模型对象中的某些键组合成JSON数据
- (NSData *)toJSONDataWithKeys:(NSArray *)propertyNames;

//重写这个函数 来设置解析时使用的属性映射器
+ (JSONKeyMapper *)keyMapper;
//重写这个函数 来设置某个属性是否是可选的
+ (BOOL)propertyIsOptional:(NSString *)propertyName;
//重写这个函数 来设置某个属性是否是忽略的
+ (BOOL)propertyIsIgnored:(NSString *)propertyName;
//重写这个函数 来设置 如果某个属性集合中是一个自定义对象或本身是自定义对象 设置此对象的类
+ (Class)classForCollectionProperty:(NSString *)propertyName;
```

JSONModel的源码这里就不在列举，其首先在类load函数中进行静态数据的加载，所支持的原生类型和基础数据类型的定义等。在对象的初始化方法中，首先使用runtime获取所有的属性和属性的修饰内容，所谓修饰内容，即是指属性名称，类型，所遵守的协议，以及是否忽略，是否可选，是否是主键等内容(过程中会使用到属性映射器keyMapper进行转化)，将其抽象成JSONModelClassProperty对象。后面在解析时，会根据JSONModelClassProperty中的自定义setter和其他信息进行赋值。

---
title: 30分钟摸透iOS中谓词NSPredicate的来龙去脉
date: 2018-03-12
categories: iOS逻辑初窥
tags: []
---
## 30分钟摸透iOS中谓词NSPredicate的来龙去脉

### 一、引言

    在现代汉语的解释中，谓词是用来描述或判断客体性质、特征或者客体之间关系的词项。通俗的说，它是描述事物属性的。在iOS开发Cocoa框架中，有提供NSPredicate类，这个类通常也被成为谓词类，其主要的作用是在Cocoa中帮助查询和检索，但是需要注意，实质上谓词并不是提供查询和检索的支持，它是一种描述查询检索条件的方式，就像更加标准通用的正则表达式一样。

    NSPredicate提供的谓词可以分为两类：比较谓词和复合谓词。

-   比较谓词：比较谓词通过使用比较运算符来描述所符合条件的属性状态。
-   复合谓词：复合谓词用来组合多个比较谓词的结果，取交集，并集或补集。

对于比较谓词，可以描述精准的比较也可以通过范围或者包含等进行模糊比较。需要注意，任何Cocoa类对象都可以支持谓词，但是此类需要实现键值编码(key-value-coding)协议。

### 二、NSPredicate类的应用解析

    NSPredicate提供创建谓词对象和解析谓词对象的方法，它也是Cocoa中有关谓词的类中的基类。我们在日常开发中，NSPredicate类的应用频率也最高。

    创建谓词对象有3种方式，分别是通过格式化字符串创建谓词，直接通过代码创建谓词，通过模板创建谓词。NSPredicate提供了如下函数来进行初始化：

```objectivec
//通过格式化字符串来进行谓词对象的初始化
+ (NSPredicate *)predicateWithFormat:(NSString *)predicateFormat argumentArray:(nullable NSArray *)arguments;
+ (NSPredicate *)predicateWithFormat:(NSString *)predicateFormat, ...;
+ (NSPredicate *)predicateWithFormat:(NSString *)predicateFormat arguments:(va_list)argList;
```

使用格式化字符串进行谓词的初始化十分灵活，但是需要注意，其谓词字符串的语法和正则表达式并不一样，后面会有具体的介绍，下面是一个谓词检索示例：

```objectivec
    //检索属性length为5的对象
    NSPredicate *  predicate = [NSPredicate predicateWithFormat:@"length = 5"];
    //对于这个数组中的字符串，即是检索字符串长度为5的元素
    NSArray * test = @[@"sda",@"321",@"sf12",@"dsdwq1",@"swfas"];
    NSArray * result = [test filteredArrayUsingPredicate:predicate];
    //将打印@[@"swfas"]
    NSLog(@"%@",result);
```

其实，你也可以像使用NSLog函数一样来进行格式化字符串的构造，可以使用%@,%d等等格式化字符来在运行时替换为变量的实际值。同时也需要注意，这种格式化字符串创建的谓词语句并不会进行语法检查，错误的语法会产生运行时错误，要格外小心。有一个小细节需要注意，在进行格式化时，如果使用的是变量则不需要添加引号，解析器会帮助你添加，如果使用到常量，则要用转义字符进行转义，例如：

```objectivec
NSPredicate *  predicate = [NSPredicate predicateWithFormat:@"name = %@ && age = \"25\"",name];
```

对于属性名，如果也需要进行格式化，需要注意不能使用%[@符号](https://my.oschina.net/uancn)，这个符号在解析时会被解析器自动添加上引号，可以使用%K，示例如下：

```objectivec
    NSString * key = @"length";
    NSPredicate *  predicate = [NSPredicate predicateWithFormat:@"%K = 5",key];
    NSArray * test = @[@"sda",@"321",@"sf12",@"dsdwq1",@"swfas"];
    NSArray * result = [test filteredArrayUsingPredicate:predicate];
    //将打印@[@"swfas"]
    NSLog(@"%@",result);
```

    通过模板来创建谓词对象也是一种十分常用的方式，和格式化字符串不同的是，谓词模板中只有键名，没有键值，键值需要在字典中进行提供，例如：

```objectivec
    NSPredicate *  predicate = [NSPredicate predicateWithFormat:@"length = $LENGTH"];
    predicate = [predicate predicateWithSubstitutionVariables:@{@"LENGTH":@5}];
    NSArray * test = @[@"sda",@"321",@"sf12",@"dsdwq1",@"swfas"];
    NSArray * result = [test filteredArrayUsingPredicate:predicate];
    //将打印@[@"swfas"]
    NSLog(@"%@",result);
```

NSPredicate中其他属性与方法解析如下：

```objectivec
//创建一个总是验证通过(YES)或不通过(NO)的谓词对象
/*
如果创建的是验证通过的，则任何检索都会成功进行返回，否则任何检索都会失败不返回任何对象
*/
+ (NSPredicate *)predicateWithValue:(BOOL)value;
//自定义实现检索函数
/*
例如前面的示例也可以这样写
NSPredicate * predicate = [NSPredicate predicateWithBlock:^BOOL(id  _Nullable evaluatedObject, NSDictionary<NSString *,id> * _Nullable bindings) {
        if ([evaluatedObject length]==5) {
            return YES;
        }
        return NO;
    }];
*/
+ (NSPredicate*)predicateWithBlock:(BOOL (^)(id _Nullable evaluatedObject, NSDictionary<NSString *, id> * _Nullable bindings))block;
//格式化字符串属性
@property (readonly, copy) NSString *predicateFormat;  
//当使用谓词模板来进行对象创建时，这个函数用来设置谓词模板中变量替换
- (instancetype)predicateWithSubstitutionVariables:(NSDictionary<NSString *, id> *)variables;
//检查一个Object对象是否可以通过验证
- (BOOL)evaluateWithObject:(nullable id)object; 
//用谓词模板进行对象的验证
- (BOOL)evaluateWithObject:(nullable id)object substitutionVariables:(nullable NSDictionary<NSString *, id> *)bindings;
```

### 三、通过代码来创建谓词对象

    前面我们说有3种创建谓词对象的方式，有两种我们已经有介绍，通过代码直接创建谓词对象是最复杂的一种。通过代码来创建谓词对象十分类似通过代码来创建Autolayout约束。通过前面我们的介绍，谓词实际是用表达式来验证对象，用代码来创建谓词实际就是用代码来创建表达式。

#### 1.先来看NSComparisonPredicate类

    这个类是NSPredicate的子类，其用来创建比较类型的谓词。例如使用下面的代码来改写上面的例子：

```objectivec
    //创建左侧表达式对象 对应为键
    NSExpression * left = [NSExpression expressionForKeyPath:@"length"];
    //创建右侧表达式对象 对应为值
    NSExpression * right = [NSExpression expressionForConstantValue:[NSNumber numberWithInt:5]];
    //创建比较谓词对象 这里设置为严格等于
    NSComparisonPredicate * pre = [NSComparisonPredicate predicateWithLeftExpression:left rightExpression:right modifier:NSDirectPredicateModifier type:NSEqualToPredicateOperatorType options:NSCaseInsensitivePredicateOption];
    NSArray * test = @[@"sda",@"321",@"sf12",@"dsdwq1",@"swfas"];
    NSArray * result = [test filteredArrayUsingPredicate:pre];
    //将打印@[@"swfas"]
    NSLog(@"%@",result);
```

NSComparisonPredicateModifier用来进行条件的修饰设置，枚举如下：

```objectivec
typedef NS_ENUM(NSUInteger, NSComparisonPredicateModifier) {
    NSDirectPredicateModifier = 0, //直接进行比较操作
    NSAllPredicateModifier,  //用于数组或集合 只有当内部所有元素都通过验证时 集合才算通过
    NSAnyPredicateModifier  //同于数组或集合 当内部有一个元素满足时 集合算通过验证
};
```

关于NSAllPredicateModifier和NSAnyPredicateModifier，这两个枚举专门用于数组或集合类型对象的验证，ALL会验证其中所有元素，全部通过后数组或集合才算验证通过，ANY则只要有一个元素验证通过，数组或集合就算验证通过，例如：

```objectivec
 NSPredicate *  pre = [NSPredicate predicateWithFormat:@"ALL length = 5"];
 NSArray * test = @[@[@"aaa",@"aa"],@[@"bbbb",@"bbbbb"],@[@"ccccc",@"ccccc"]];
    NSArray * result = [test filteredArrayUsingPredicate:pre];
    //将打印@[@[@"ccccc",@"ccccc"]]
    NSLog(@"%@",result);
```

NSPredicateOperatorType枚举用来设置运算符类型，如下：

```objectivec
typedef NS_ENUM(NSUInteger, NSPredicateOperatorType) {
    NSLessThanPredicateOperatorType = 0, // 小于
    NSLessThanOrEqualToPredicateOperatorType, // 小于等于
    NSGreaterThanPredicateOperatorType, // 大于
    NSGreaterThanOrEqualToPredicateOperatorType, // 大于等于
    NSEqualToPredicateOperatorType, // 等于
    NSNotEqualToPredicateOperatorType, //不等于
    NSMatchesPredicateOperatorType, //正则比配
    NSLikePredicateOperatorType,  //Like匹配 与SQL类似
    NSBeginsWithPredicateOperatorType, //左边的表达式 以右边的表达式作为开头
    NSEndsWithPredicateOperatorType,//左边的表达式 以右边的表达式作为结尾
    NSInPredicateOperatorType, // 左边的表达式 出现在右边的集合中 
    NSCustomSelectorPredicateOperatorType,//使用自定义的函数来进行 验证
    NSContainsPredicateOperatorType, //左边的集合包括右边的元素
    NSBetweenPredicateOperatorType //左边表达式的值在右边的范围中 例如 1 BETWEEN { 0 , 33 }
};
```

NSComparisonPredicateOptions枚举用来设置比较的方式，如下：

```objectivec
//如果不需要特殊指定  这个枚举值也可以传0
typedef NS_OPTIONS(NSUInteger, NSComparisonPredicateOptions) {
    NSCaseInsensitivePredicateOption = 0x01, //不区分大小写
    NSDiacriticInsensitivePredicateOption = 0x02,//不区分读音符号
    NSNormalizedPredicateOption //比较前进行预处理 代替上面两个选项
};
```

#### 2.NSExpression类

    NSExpression类则是提供创建表达式，下面列出了其中一些方便理解的方法：

```objectivec
//通过格式化字符串创建表达式
+ (NSExpression *)expressionWithFormat:(NSString *)expressionFormat argumentArray:(NSArray *)arguments;
+ (NSExpression *)expressionWithFormat:(NSString *)expressionFormat, ...;
+ (NSExpression *)expressionWithFormat:(NSString *)expressionFormat arguments:(va_list)argList;
//直接通过对象创建常亮的表达式
+ (NSExpression *)expressionForConstantValue:(nullable id)obj; 
//创建变量表达式  验证时将从binding字典中进行替换
+ (NSExpression *)expressionForVariable:(NSString *)string; 
//将多个表达式组合成一个
+ (NSExpression *)expressionForAggregate:(NSArray<NSExpression *> *)subexpressions;
+ (NSExpression *)expressionForUnionSet:(NSExpression *)left with:(NSExpression *)right;
+ (NSExpression *)expressionForIntersectSet:(NSExpression *)left with:(NSExpression *)right;
+ (NSExpression *)expressionForMinusSet:(NSExpression *)left with:(NSExpression *)right;
+ (NSExpression *)expressionForSubquery:(NSExpression *)expression usingIteratorVariable:(NSString *)variable predicate:(NSPredicate *)predicate;
//通过预定义的函数和参数数组来构建表达式对象  预定义的函数 可见dev开发文档
+ (NSExpression *)expressionForFunction:(NSString *)name arguments:(NSArray *)parameters;
```

#### 3.NSCompoundPredicate类

    这个类也是NSPredicate类的子类，其使用逻辑关系来组合多个谓词对象，解析如下：

```objectivec
//进行对象初始化
/*
typedef NS_ENUM(NSUInteger, NSCompoundPredicateType) {
    NSNotPredicateType = 0,   //取非
    NSAndPredicateType,  //与运算 
    NSOrPredicateType,   //或运算
};
*/
- (instancetype)initWithType:(NSCompoundPredicateType)type subpredicates:(NSArray<NSPredicate *> *)subpredicates;
//快速创建与运算
+ (NSCompoundPredicate *)andPredicateWithSubpredicates:(NSArray<NSPredicate *> *)subpredicates;
//快速创建或运算
+ (NSCompoundPredicate *)orPredicateWithSubpredicates:(NSArray<NSPredicate *> *)subpredicates;
//快速创建非运算
+ (NSCompoundPredicate *)notPredicateWithSubpredicate:(NSPredicate *)predicate;
```

### 四、谓词的几种使用场景

    谓词主要用在验证对象，数组和集合的过滤。对象的验证前面有介绍，关于数据和集合的过滤函数，类别如下：

```objectivec
@interface NSArray<ObjectType> (NSPredicateSupport)
//不可变数组使用过滤器后返回新数组
- (NSArray<ObjectType> *)filteredArrayUsingPredicate:(NSPredicate *)predicate; 
@end

@interface NSMutableArray<ObjectType> (NSPredicateSupport)
//可变数组可以直接进行过滤操作
- (void)filterUsingPredicate:(NSPredicate *)predicate;
@end

@interface NSSet<ObjectType> (NSPredicateSupport)
//不可变集合过滤后返回新集合
- (NSSet<ObjectType> *)filteredSetUsingPredicate:(NSPredicate *)predicate;
@end

@interface NSMutableSet<ObjectType> (NSPredicateSupport)
//可变集合可以直接进行过滤操作
- (void)filterUsingPredicate:(NSPredicate *)predicate;
@end

@interface NSOrderedSet<ObjectType> (NSPredicateSupport)

- (NSOrderedSet<ObjectType> *)filteredOrderedSetUsingPredicate:(NSPredicate *)p;

@end

@interface NSMutableOrderedSet<ObjectType> (NSPredicateSupport)

- (void)filterUsingPredicate:(NSPredicate *)p;

@end
```

### 五、谓词的格式化语法总览

    下面列出了在谓词的格式化字符串规则语法。

| 语法规则 | 意义 |
| = | 左侧等于右侧 |
| ==     | 左侧等于右侧，与=一致  |
| >=     | 左侧大于等于右侧 |
| => | 左侧大于等于右侧 与 >=一致 |
| <= | 左侧小于等于右侧 |
| =<     | 左侧小于等于右侧 与<=一致 |
| > | 左侧大于右侧 |
| < | 左侧小于右侧 |
| != | 左侧不等于右侧 |
| <> | 左侧不等于右侧 与!=一致 |
| BETWEEN    | 左侧在右侧的集合中 key BETWEEN @\[[@1](https://my.oschina.net/u/1198),@2\] |
| TRUEPREDICATE | 总是返回YES的谓词 |
| FALSEPREDICATE | 总是返回NO的谓词 |
| AND | 逻辑与 |
| && | 逻辑与 与AND一致 |
| OR | 逻辑或 |
| || | 逻辑或 与OR一致 |
| NOT | 逻辑非 |
| !     | 逻辑非 与NOT一致 |
| BEGINWITH | 左侧以右侧字符串开头 |
| ENDWITH | 左侧以右侧字符串结尾 |
| CONTAINS | 左侧集合包含右侧元素 |
| LIKE     | 左侧等于右侧 并且 *和?等通配符可以使用 |
| MATCHES | 正则匹配 |
| ANY | 对于数组集合类，验证其中任一元素 |
| SOME | 同ANY一致 |
| ALL | 对于数组集合类，验证其中所有元素 |
| NONE | 作用等同于NOT (ANY) |
| IN     | 左侧在右侧集合中 |
| SELF | 被验证的对象本身 |

---
title: Objective-C中NSArray类的解读
date: 2016-07-19
categories: Objective-C浅探
tags: []
---
## Objective-C中NSArray类的解读

    NSArray数组类是Objective-C语言中常用的也是重要的一个类，除了开发中常用到的一些基础功能，NSArray及其相关类中还封装了许多更加强大的功能。有机会总结了一下，与需要的朋友们分享。

NSArray中属性与方法：

```objectivec
//获取数组中元素个数
@property (readonly) NSUInteger count;
//通过下标获数组中的元素
- (ObjectType)objectAtIndex:(NSUInteger)index;
//初始化方法
- (instancetype)init;
//通过C语言风格的数组创建NSArray对象 需要注意，C数组中需要为Objective对象，cnt参数为C数组的长度
//如果cnt的值小于C数组的长度，则会对C数据进行截取赋值，如果大于则程序会崩溃
- (instancetype)initWithObjects:(const ObjectType [])objects count:(NSUInteger)cnt;
//数组的归档方法
- (nullable instancetype)initWithCoder:(NSCoder *)aDecoder;
//像数组中追加一个元素 这个方法会返回一个新的数组
- (NSArray<ObjectType> *)arrayByAddingObject:(ObjectType)anObject;
//像数组中追加一组元素 这个方法会返回一个新的数组
- (NSArray<ObjectType> *)arrayByAddingObjectsFromArray:(NSArray<ObjectType> *)otherArray;
//返回一个字符串，将数组中的元素以separator为分隔符进行组合
/*
NSArray  * array = @[@1,@2,@3,@4];
将打印1,2,3,4
NSString * res = [array componentsJoinedByString:@","];
*/
- (NSString *)componentsJoinedByString:(NSString *)separator;
//判断数组中是否包含某个元素
- (BOOL)containsObject:(ObjectType)anObject;
//数组的打印方法
@property (readonly, copy) NSString *description;
- (NSString *)descriptionWithLocale:(nullable id)locale;
- (NSString *)descriptionWithLocale:(nullable id)locale indent:(NSUInteger)level;
//获取第一个包含于另一个数组中的元素
- (nullable ObjectType)firstObjectCommonWithArray:(NSArray<ObjectType> *)otherArray;
//将数组中一定范围的元素读取到一个C数组中 objects参数需要为分配好空间的C指针
- (void)getObjects:(ObjectType __unsafe_unretained [])objects range:(NSRange)range;
//获取某个元素在数值中的下标值
- (NSUInteger)indexOfObject:(ObjectType)anObject;
//获取某个范围内的元素的下标值
- (NSUInteger)indexOfObject:(ObjectType)anObject inRange:(NSRange)range;
//获取与给定元素相同的元素在数组中的最小下标值
- (NSUInteger)indexOfObjectIdenticalTo:(ObjectType)anObject;
//在一定范围内 获取与给定元素相同的元素在数组中的最小下标值
- (NSUInteger)indexOfObjectIdenticalTo:(ObjectType)anObject inRange:(NSRange)range;
//判断两个数组是否相同
- (BOOL)isEqualToArray:(NSArray<ObjectType> *)otherArray;
//获取数组中第一个元素
@property (nullable, nonatomic, readonly) ObjectType firstObject NS_AVAILABLE(10_6, 4_0);
//获取数组中最后一个元素
@property (nullable, nonatomic, readonly) ObjectType lastObject;
//获取数组的枚举对象
- (NSEnumerator<ObjectType> *)objectEnumerator;
//获取数组的逆向枚举对象
- (NSEnumerator<ObjectType> *)reverseObjectEnumerator;
/*
这个属性可以获取一个已经排序数组的排序规则 在使用
- (NSArray<ObjectType> *)sortedArrayUsingFunction:(NSInteger (*)(ObjectType, ObjectType, void * __nullable))comparator context:(nullable void *)context hint:(nullable NSData *)hint;
方法时可以将此排序规则传入 
对于没有排序过的数组，使用
- (NSArray<ObjectType> *)sortedArrayUsingFunction:(NSInteger (*)(ObjectType, ObjectType, void * __nullable))comparator context:(nullable void *)context;
方法会自动产生一个这样的排序规则
*/
@property (readonly, copy) NSData *sortedArrayHint;
//通过C排序函数进行排序
/*
示例：
NSInteger sort(id obj1, id obj2, void *context){
    NSNumber *str1 =(NSNumber*) obj1;
    NSNumber *str2 =(NSNumber*) obj2;
    if ([str1 intValue] < [str2 intValue]) {
        return NSOrderedDescending;
    }
    else if([str1 intValue] == [str2 intValue])
    {
        return NSOrderedSame;
    }
    return NSOrderedAscending;
}
- (void)viewDidLoad {
    [super viewDidLoad];
    NSArray  * array = @[@1,@3,@2,@4];
    array = [array sortedArrayUsingFunction:sort context:nil];
    NSString * res = [array componentsJoinedByString:@","];
    NSLog(@"%@",res);
}
*/
- (NSArray<ObjectType> *)sortedArrayUsingFunction:(NSInteger (*)(ObjectType, ObjectType, void * __nullable))comparator context:(nullable void *)context;
//通过C排序函数进行数组排序
- (NSArray<ObjectType> *)sortedArrayUsingFunction:(NSInteger (*)(ObjectType, ObjectType, void * __nullable))comparator context:(nullable void *)context hint:(nullable NSData *)hint;
//使用函数选择器进行数组排序
- (NSArray<ObjectType> *)sortedArrayUsingSelector:(SEL)comparator;
//获取数组一定范围的子数组
- (NSArray<ObjectType> *)subarrayWithRange:(NSRange)range;
//将数组写入文件
- (BOOL)writeToFile:(NSString *)path atomically:(BOOL)useAuxiliaryFile;
//将数组写入指定url路径
- (BOOL)writeToURL:(NSURL *)url atomically:(BOOL)atomically;
//是数组中的所有元素调用某个方法选择器
- (void)makeObjectsPerformSelector:(SEL)aSelector;
//功能同上 支持传参
- (void)makeObjectsPerformSelector:(SEL)aSelector withObject:(nullable id)argument;
//获取一个下标集合所对应的元素
- (NSArray<ObjectType> *)objectsAtIndexes:(NSIndexSet *)indexes;
//数组的下标方法 子类重写
- (ObjectType)objectAtIndexedSubscript:(NSUInteger)idx NS_AVAILABLE(10_8, 6_0);
//对数组中的元素进行枚举遍历
- (void)enumerateObjectsUsingBlock:(void (^)(ObjectType obj, NSUInteger idx, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);
//对数组中的元素进行枚举遍历
/*
typedef NS_OPTIONS(NSUInteger, NSEnumerationOptions) {
    NSEnumerationConcurrent = (1UL << 0),//正向枚举
    NSEnumerationReverse = (1UL << 1),   //逆向枚举
}; 
*/
- (void)enumerateObjectsWithOptions:(NSEnumerationOptions)opts usingBlock:(void (^)(ObjectType obj, NSUInteger idx, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);
//在一个下标集合中枚举
- (void)enumerateObjectsAtIndexes:(NSIndexSet *)s options:(NSEnumerationOptions)opts usingBlock:(void (^)(ObjectType obj, NSUInteger idx, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);
//通过遍历的方式查找符合条件的元素下标
- (NSUInteger)indexOfObjectPassingTest:(BOOL (^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);
//通常 可以设置遍历方式
- (NSUInteger)indexOfObjectWithOptions:(NSEnumerationOptions)opts passingTest:(BOOL (^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);
//同上 在一定下标集合中遍历
- (NSUInteger)indexOfObjectAtIndexes:(NSIndexSet *)s options:(NSEnumerationOptions)opts passingTest:(BOOL (^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);
//通过遍历的方式查找所有符合条件的元素下标
- (NSIndexSet *)indexesOfObjectsPassingTest:(BOOL (^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);
//同上
- (NSIndexSet *)indexesOfObjectsWithOptions:(NSEnumerationOptions)opts passingTest:(BOOL (^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);
//同上
- (NSIndexSet *)indexesOfObjectsAtIndexes:(NSIndexSet *)s options:(NSEnumerationOptions)opts passingTest:(BOOL (^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);
//通过block进行数组排序
- (NSArray<ObjectType> *)sortedArrayUsingComparator:(NSComparator)cmptr NS_AVAILABLE(10_6, 4_0);
//同上
/*
typedef NS_OPTIONS(NSUInteger, NSSortOptions) {
    NSSortConcurrent = (1UL << 0),//同步排序
    NSSortStable = (1UL << 4),//稳定排序
};

*/
- (NSArray<ObjectType> *)sortedArrayWithOptions:(NSSortOptions)opts usingComparator:(NSComparator)cmptr NS_AVAILABLE(10_6, 4_0);
//二分查找的枚举参数
typedef NS_OPTIONS(NSUInteger, NSBinarySearchingOptions) {
    NSBinarySearchingFirstEqual = (1UL << 8),
    NSBinarySearchingLastEqual = (1UL << 9),
    NSBinarySearchingInsertionIndex = (1UL << 10),
};
//对区域排序的数组进行二分查找
- (NSUInteger)indexOfObject:(ObjectType)obj inSortedRange:(NSRange)r options:(NSBinarySearchingOptions)opts usingComparator:(NSComparator)cmp NS_AVAILABLE(10_6, 4_0); // binary search

//创建对象
+ (instancetype)array;
//通过一个元素创建数组对象
+ (instancetype)arrayWithObject:(ObjectType)anObject;
//通过C数组创建数组对象
+ (instancetype)arrayWithObjects:(const ObjectType [])objects count:(NSUInteger)cnt;
//通过一组元素创建数组对象
+ (instancetype)arrayWithObjects:(ObjectType)firstObj, ... NS_REQUIRES_NIL_TERMINATION;
//通过另一个数组创建数组对象
+ (instancetype)arrayWithArray:(NSArray<ObjectType> *)array;
//初始化方法
- (instancetype)initWithObjects:(ObjectType)firstObj, ... NS_REQUIRES_NIL_TERMINATION;
- (instancetype)initWithArray:(NSArray<ObjectType> *)array;
- (instancetype)initWithArray:(NSArray<ObjectType> *)array copyItems:(BOOL)flag;
//通过文件创建数组
+ (nullable NSArray<ObjectType> *)arrayWithContentsOfFile:(NSString *)path;
//通过url创建数组
+ (nullable NSArray<ObjectType> *)arrayWithContentsOfURL:(NSURL *)url;
同上
- (nullable NSArray<ObjectType> *)initWithContentsOfFile:(NSString *)path;
- (nullable NSArray<ObjectType> *)initWithContentsOfURL:(NSURL *)url;
//获取数组所有元素 需要传入分配了内存的C指针
- (void)getObjects:(ObjectType __unsafe_unretained [])objects;
```

NSMutableArray中属性与方法：

```objectivec
//向数组中追加一个元素
- (void)addObject:(ObjectType)anObject;
//向数组某个位置插入一个元素
- (void)insertObject:(ObjectType)anObject atIndex:(NSUInteger)index;
//删除数组中最后一个元素
- (void)removeLastObject;
//删除数组中指定位置的元素
- (void)removeObjectAtIndex:(NSUInteger)index;
//替换数组中一个位置的元素
- (void)replaceObjectAtIndex:(NSUInteger)index withObject:(ObjectType)anObject;
//初始化
- (instancetype)init NS_DESIGNATED_INITIALIZER;
- (instancetype)initWithCapacity:(NSUInteger)numItems NS_DESIGNATED_INITIALIZER;
- (nullable instancetype)initWithCoder:(NSCoder *)aDecoder NS_DESIGNATED_INITIALIZER;

//通过数组来追加元素
- (void)addObjectsFromArray:(NSArray<ObjectType> *)otherArray;
//交换两个元素
- (void)exchangeObjectAtIndex:(NSUInteger)idx1 withObjectAtIndex:(NSUInteger)idx2;
//删除所有元素
- (void)removeAllObjects;
//在一定范围内删除元素
- (void)removeObject:(ObjectType)anObject inRange:(NSRange)range;
//删除一个元素
- (void)removeObject:(ObjectType)anObject;
//删除指定范围内下标最小的某个元素
- (void)removeObjectIdenticalTo:(ObjectType)anObject inRange:(NSRange)range;
//删除某个元素 下标最小的
- (void)removeObjectIdenticalTo:(ObjectType)anObject;
//删除一定范围内的所有元素
- (void)removeObjectsFromIndices:(NSUInteger *)indices numIndices:(NSUInteger)cnt NS_DEPRECATED(10_0, 10_6, 2_0, 4_0);
//通过数组删除元素
- (void)removeObjectsInArray:(NSArray<ObjectType> *)otherArray;
//通过范围删除元素
- (void)removeObjectsInRange:(NSRange)range;
//替换一组元素
- (void)replaceObjectsInRange:(NSRange)range withObjectsFromArray:(NSArray<ObjectType> *)otherArray range:(NSRange)otherRange;
//替换一组元素
- (void)replaceObjectsInRange:(NSRange)range withObjectsFromArray:(NSArray<ObjectType> *)otherArray;
//设置数组元素
- (void)setArray:(NSArray<ObjectType> *)otherArray;
//进行数组排序
- (void)sortUsingFunction:(NSInteger (*)(ObjectType,  ObjectType, void * __nullable))compare context:(nullable void *)context;
//进行数组排序
- (void)sortUsingSelector:(SEL)comparator;
//插入一组元素
- (void)insertObjects:(NSArray<ObjectType> *)objects atIndexes:(NSIndexSet *)indexes;
//删除一组元素
- (void)removeObjectsAtIndexes:(NSIndexSet *)indexes;
//替换一组元素
- (void)replaceObjectsAtIndexes:(NSIndexSet *)indexes withObjects:(NSArray<ObjectType> *)objects;
//设置某个下标对应的元素 子类覆写
- (void)setObject:(ObjectType)obj atIndexedSubscript:(NSUInteger)idx NS_AVAILABLE(10_8, 6_0);
//进行数组排序
- (void)sortUsingComparator:(NSComparator)cmptr NS_AVAILABLE(10_6, 4_0);
//进行数组排序
- (void)sortWithOptions:(NSSortOptions)opts usingComparator:(NSComparator)cmptr NS_AVAILABLE(10_6, 4_0);

//创建数组 numItems为元素个数
+ (instancetype)arrayWithCapacity:(NSUInteger)numItems;
//通过文件创建数组
+ (nullable NSMutableArray<ObjectType> *)arrayWithContentsOfFile:(NSString *)path;
//通过url创建数组
+ (nullable NSMutableArray<ObjectType> *)arrayWithContentsOfURL:(NSURL *)url;
同上
- (nullable NSMutableArray<ObjectType> *)initWithContentsOfFile:(NSString *)path;
- (nullable NSMutableArray<ObjectType> *)initWithContentsOfURL:(NSURL *)url;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

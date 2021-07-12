---
title: iOS中第三方有序字典框架——M13OrderedDictionary
date: 2016-07-01
categories: iOS第三方库
tags: []
---
## iOS中第三方有序字典框架——M13OrderedDictionary

### 一、引言

        M13OrderedDictionary是拥有字典和数组功能的第三方集合序列，开发者可以通过索引和键值来实现对其中元素的访问。其实现了NSArray和NSDictionary中的所有方法，并且支持KVC与KVO。

        M13OederedDictionary中提供的方法包括：

1.创建与初始化。

2.访问键和值

3.查询与搜索。

4.发送消息。

5.比较与排序。

6.枚举与遍历。

7.描述与存储。

8.KVO键值监听。

9.KVC键值编码。

10.索引与下标。

        另外，M13OrderedDictionary针对Xcode7也做了许多优化，例如引入了泛型的代码支持的风格。M13OrderedDictionary库的git地址如下：[https://github.com/Marxon13/M13OrderedDictionary](https://github.com/Marxon13/M13OrderedDictionary)。

### 二、M13OrderedDictionary中方法与属性解析

```objectivec
//类方法创建实例对象
//默认的初始化方法
+ (instancetype)orderedDictionary;
//使用M13OrderedDictionary来创建实例对象
+ (instancetype)orderedDictionaryWithOrderedDictionary:(M13OrderedDictionary M13Generics(KeyType, ObjectType) *)orderedDictionary;
//通过文件来创建实例对象
+ (instancetype)orderedDictionaryWithContentsOfFile:(NSString *)path;
//通过URL来创建实例对象
+ (instancetype)orderedDictionaryWithContentsOfURL:(NSURL *)URL;
//创建单键值实例对象
+ (instancetype)orderedDictionaryWithObject:(M13GenericType(ObjectType, id))anObject
                              pairedWithKey:(M13GenericType(KeyType, id<NSCopying>))aKey;
//通过NSDictionary创建实例对象
+ (instancetype)orderedDictionaryWithDictionary:(NSDictionary M13Generics(KeyType, ObjectType) *)entries;

//初始化方法创建实例对象
//默认的初始化方法
- (instancetype)init;
//使用M13OrderedDictionary来进行初始化
- (instancetype)initWithOrderedDictionary:(M13OrderedDictionary M13Generics(KeyType, ObjectType) *)orderedDictionary;
//使用M13OrderedDictionary来进行初始化 可选是否对其中元素进行复制操作
- (instancetype)initWithOrderedDictionary:(M13OrderedDictionary M13Generics(KeyType, ObjectType) *)orderedDictionary copyEntries:(BOOL)flag;
//通过文件来进行初始化
- (instancetype)initWithContentsOfFile:(NSString *)path;
//通过url来进行初始化
- (instancetype)initWithContentsOfURL:(NSURL *)URL;
//通过NSDictionary对象来进行初始化
- (instancetype)initWithContentsOfDictionary:(NSDictionary M13Generics(KeyType, ObjectType) *)entries;
//通过键数组与值数组来进行初始化
- (instancetype)initWithObjects:(NSArray M13Generics(ObjectType) *)orderedObjects
                 pairedWithKeys:(NSArray M13Generics(KeyType) *)orderedKeys NS_DESIGNATED_INITIALIZER;

//方法
//判断字典中是否包含某个元素
- (BOOL)containsObject:(M13GenericType(ObjectType, id))object;
//判断字典中是否包含某个键值对
- (BOOL)containsObject:(M13GenericType(ObjectType, id))object
         pairedWithKey:(M13GenericType(KeyType, id<NSCopying>))key;
//判断字典中是否包含某个键值对 传入的字典参数需要为单键值字典
- (BOOL)containsEntry:(NSDictionary M13Generics(KeyType, ObjectType) *)entry;
//获取字典中元素个数
@property (nonatomic, readonly) NSUInteger count;
//获取字典中最后一个元素的值
@property (nonatomic, readonly, M13_NULLABLE) M13GenericType(ObjectType, id) lastObject;
//获取字典中最后一个元素的键
@property (nonatomic, readonly, M13_NULLABLE) M13GenericType(KeyType, id<NSCopying>) lastKey;
//获取字典中最后一个元素键值对
@property (nonatomic, readonly, M13_NULLABLE) NSDictionary M13Generics(KeyType, ObjectType) *lastEntry;
//通过某个下标获取字典中的元素的值
- (M13GenericType(ObjectType, id))objectAtIndex:(NSUInteger)index;
//通过某个下标获取字典中的元素的键
- (M13GenericType(KeyType, id<NSCopying>))keyAtIndex:(NSUInteger)index;
//通过某个下标获取字段中的元素 返回的为单键值对NSDictionary对象
- (NSDictionary M13Generics(KeyType, ObjectType) *)entryAtIndex:(NSUInteger)index;
//通过一组下标获取一组元素的值
- (NSArray M13Generics(ObjectType) *)objectsAtIndices:(NSIndexSet *)indeces;
//通过一组下标获取一组元素的键
- (NSArray M13Generics(KeyType) *)keysAtIndices:(NSIndexSet *)indices;
//通过一组下标获取一组元素 这个方法获取的是有序集合
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)entriesAtIndices:(NSIndexSet *)indices;
//通过一组下标获取一组元素 这个方法获取的是无序集合
- (NSDictionary M13Generics(KeyType, ObjectType) *)unorderedEntriesAtIndices:(NSIndexSet *)indices;
//示例中包含的无序字典集合
@property (nonatomic, readonly) NSDictionary M13Generics(KeyType, ObjectType) *unorderedDictionary;
//所有键组成的数组
@property (nonatomic, readonly) NSArray M13Generics(KeyType) *allKeys;
//所有值组成的数组
@property (nonatomic, readonly) NSArray M13Generics(ObjectType) *allObjects;
//获取某个值对应的所有键组成的数组
- (NSArray M13Generics(KeyType) *)allKeysForObject:(M13GenericType(ObjectType, id))anObject;
//获取某个键对应的值
- (M13_NULLABLE M13GenericType(ObjectType, id))objectForKey:(M13GenericType(KeyType, id<NSCopying>))key;
//获取某些键对应的值 如果没有找到 则可以设置默认返回的值 即参数anObject
- (NSArray M13Generics(ObjectType) *)objectForKeys:(NSArray M13Generics(KeyType) *)keys
                                    notFoundMarker:(M13GenericType(ObjectType, id))anObject;
//所有值的枚举
@property (nonatomic, readonly) NSEnumerator M13Generics(ObjectType) *objectEnumerator;
//所有键的枚举
@property (nonatomic, readonly) NSEnumerator M13Generics(KeyType) *keyEnumerator;
//所有元素的枚举
@property (nonatomic, readonly) NSEnumerator M13Generics(NSDictionary<KeyType, ObjectType> *) *entryEnumerator;
//所有值的反向枚举
@property (nonatomic, readonly) NSEnumerator M13Generics(ObjectType) *reverseObjectEnumerator;
//所有键的反向枚举
@property (nonatomic, readonly) NSEnumerator M13Generics(KeyType) *reverseKeyEnumerator;
//所有元素的反向枚举
@property (nonatomic, readonly) NSEnumerator M13Generics(NSDictionary<KeyType, ObjectType> *) *reverseEntryEnumerator;
//获取某个值的下标 找不到会返回NSNotFound
- (NSUInteger)indexOfObject:(M13GenericType(ObjectType, id))object;
//获取某个键的下标 找不到会返回NSNotFound
- (NSUInteger)indexOfKey:(M13GenericType(KeyType, id<NSCopying>))key;
//获取某个元素的下标 找不到会返回NSNotFound
- (NSUInteger)indexOfEntryWithObject:(M13GenericType(ObjectType, id))object
                       pairedWithKey:(M13GenericType(KeyType, id<NSCopying>))key;
//通过NSDictionary来获取某个元素的下标 找不到会返回NSNotFound
- (NSUInteger)indexOfEntry:(NSDictionary M13Generics(KeyType, ObjectType) *)entry;
//通过元素的值在某个范围内查询下标
- (NSUInteger)indexOfObject:(M13GenericType(ObjectType, id))object inRange:(NSRange)range;
//通过元素的键在某个范围内查询下标
- (NSUInteger)indexOfKey:(M13GenericType(KeyType, id<NSCopying>))key inRange:(NSRange)range;
//在某个范围内查询某个元素的下标
- (NSUInteger)indexOfEntryWithObject:(M13GenericType(ObjectType, id))object
                       pairedWithKey:(M13GenericType(KeyType, id<NSCopying>))key
                             inRange:(NSRange)range;
- (NSUInteger)indexOfEntry:(NSDictionary M13Generics(KeyType, ObjectType) *)entry inRange:(NSRange)range;
//查找与某个元素的值相同的元素下标
- (NSUInteger)indexOfObjectIdenticalTo:(M13GenericType(ObjectType, id))object;
//查找获取与某个元素的值相同的元素的键
- (M13_NULLABLE M13GenericType(KeyType, id<NSCopying>))keyOfObjectIdenticalTo:(M13GenericType(ObjectType, id))object;
//查找与某个元素的值相同的元素下标 在某个范围内进行查找
- (NSUInteger)indexOfObjectIdenticalTo:(M13GenericType(ObjectType, id))object inRange:(NSRange)range;
//查找获取与某个元素的值相同的元素的键 在某个范围内进行查找
- (M13_NULLABLE M13GenericType(KeyType, id<NSCopying>))keyOfObjectIdenticalTo:(M13GenericType(ObjectType, id))object inRange:(NSRange)range;
//符合查找block中检测条件的元素的下标
/*
开发者可以在block中获取到遍历出的 object与index，返回值决定是否停止查找
*/
- (NSUInteger)indexOfObjectPassingTest:(BOOL (^)(M13GenericType(ObjectType, id) obj,
                                                 NSUInteger idx,
                                                 BOOL *stop))predicate;
//同上 获取到的是键
- (M13_NULLABLE M13GenericType(KeyType, id<NSCopying>))keyOfObjectPassingTest:(BOOL (^)(M13GenericType(ObjectType, id) obj, NSUInteger idx,BOOL *stop))predicate;
//同上 只是这个方法可以设置枚举类型
/*
typedef NS_OPTIONS(NSUInteger, NSEnumerationOptions) {
    NSEnumerationConcurrent = (1UL << 0), //正向枚举
    NSEnumerationReverse = (1UL << 1),    //逆向枚举
};
*/
- (NSUInteger)indexOfObjectWithOptions:(NSEnumerationOptions)opts passingTest:(BOOL (^)(M13GenericType(ObjectType, id) obj, NSUInteger idx, BOOL *stop))predicate;
//同上 获取到的是元素
- (M13_NULLABLE M13GenericType(KeyType, id<NSCopying>))keyOfObjectWithOptions:(NSEnumerationOptions)opts passingTest:(BOOL (^)(M13GenericType(ObjectType, id) obj, NSUInteger idx,BOOL *stop))predicate;
//在一定下标集合中进行查找
- (NSUInteger)indexOfObjectAtIndices:(NSIndexSet *)indexSet
                             options:(NSEnumerationOptions)opts
                         passingTest:(BOOL (^)(M13GenericType(ObjectType, id) obj, NSUInteger idx, BOOL *stop))predicate;
//同上
- (M13_NULLABLE M13GenericType(KeyType, id<NSCopying>))keyOfObjectAtIndices:(NSIndexSet *)indexSet options:(NSEnumerationOptions)opts passingTest:(BOOL (^)(M13GenericType(ObjectType, id) obj,NSUInteger idx,BOOL *stop))predicate;
//在范围内进行比较查询
- (NSUInteger)indexOfObject:(M13GenericType(ObjectType, id))object
              inSortedRange:(NSRange)r options:(NSBinarySearchingOptions)opts
            usingComparator:(NSComparator)cmp;
//同上
- (M13_NULLABLE M13GenericType(KeyType, id<NSCopying>))keyOfObject:(M13GenericType(ObjectType, id))object
                                                     inSortedRange:(NSRange)r
                                                           options:(NSBinarySearchingOptions)opts
                                                   usingComparator:(NSComparator)cmp;
//进行一组元素下标的查询
- (NSIndexSet *)indicesOfObjectsPassingTest:(BOOL (^)(M13GenericType(ObjectType, id) obj,
                                                      NSUInteger idx,
                                                      BOOL *stop))predicate;
- (NSIndexSet *)indicesOfObjectsWithOptions:(NSEnumerationOptions)opts
                                passingTest:(BOOL (^)(M13GenericType(ObjectType, id) obj,
                                                      NSUInteger idx,
                                                      BOOL *stop))predicate;
//进行一组元素键的查询
- (NSArray M13Generics(KeyType) *)keysOfObjectsPassingTest:(BOOL (^)(M13GenericType(ObjectType, id) obj,
                                                                     NSUInteger idx,
                                                                     BOOL *stop))predicate;
- (NSArray M13Generics(KeyType) *)keysOfObjectsWithOptions:(NSEnumerationOptions)opts
                                               passingTest:(BOOL (^)(M13GenericType(ObjectType, id) obj,
                                                                     NSUInteger idx,
                                                                     BOOL *stop))predicate;
//向字典中的每一个元素发送消息
- (void)makeObjectsPerformSelector:(SEL)aSelector;
//向字典中的每一个元素发送消息 带参数
- (void)makeObjectsPerformSelector:(SEL)aSelector withObject:(id)anObject;
//对字典中的元素进行枚举遍历
- (void)enumerateObjectsUsingBlock:(void (^)(M13GenericType(ObjectType, id) obj, NSUInteger idx, BOOL *stop))block;
- (void)enumerateObjectsWithOptions:(NSEnumerationOptions)opts
                         usingBlock:(void (^)(M13GenericType(ObjectType, id) obj, NSUInteger idx, BOOL *stop))block;
//在一定范围内进行枚举
- (void)enumerateObjectsAtIndices:(NSIndexSet *)indexSet
                          options:(NSEnumerationOptions)opts
                       usingBlock:(void (^)(M13GenericType(ObjectType, id) obj, NSUInteger idx, BOOL *stop))block;
//获取与另一个数组中第一个相同的元素的值
- (M13GenericType(ObjectType, id))firstObjectInCommonWithOrderedDictionary:(M13OrderedDictionary *)otherOrderedDictionary;
//获取与另一个数组中第一个相同的元素的键
- (M13GenericType(ObjectType, id<NSCopying>))firstKeyInCommonWithOrderedDictionary:(M13OrderedDictionary *)otherOrderedDictionary;
//获取与另一个数组中第一个相同的元素
- (NSDictionary M13Generics(KeyType, ObjectType) *)firstEntryInCommonWithOrderedDictionary:(M13OrderedDictionary *)otherOrderedDictionary;
//判断两个字典是否相同
- (BOOL)isEqualToOrderedDictionary:(M13OrderedDictionary *)otherOrderedDictionary;
//向字典中追加键值对
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)orderedDictionaryByAddingObject:(M13GenericType(ObjectType, id))object pairedWithKey:(M13GenericType(KeyType, id<NSCopying>))aKey;
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)orderedDictionaryByAddingEntry:(NSDictionary M13Generics(KeyType, ObjectType) *)entry;
//向字典中追加一组键值对
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)orderedDictionaryByAddingObjects:(NSArray M13Generics(ObjectType) *)orderedObjects pairedWithKeys:(NSArray M13Generics(KeyType) *)orderedKeys;
//筛选元素
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)filteredOrderDictionarysUsingPredicateForObjects:(NSPredicate *)predicate;
//获取一定范围内的子字典
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)subOrderedDictionaryWithRange:(NSRange)range;
//进行元素排序相关的方法
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)sortedByObjectsUsingFunction:(NSInteger (*)(M13GenericType(ObjectType, id),M13GenericType(ObjectType, id),void * M13__NULLABLE))comparator context:(M13_NULLABLE void *)context;
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)sortedByKeysUsingFunction:(NSInteger (*)(M13GenericType(KeyType, id<NSCopying>),M13GenericType(KeyType, id<NSCopying>),void * M13__NULLABLE))comparator context:(M13_NULLABLE void *)context;
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)sortedByObjectsUsingFunction:(NSInteger (*)(M13GenericType(ObjectType, id),M13GenericType(ObjectType, id), void * M13__NULLABLE))comparator context:(M13_NULLABLE void *)context hint:(M13_NULLABLE NSData *)hint;
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)sortedByKeysUsingFunction:(NSInteger (*)(M13GenericType(KeyType, id<NSCopying>),M13GenericType(KeyType, id<NSCopying>),void * M13__NULLABLE))comparator context:(M13_NULLABLE void *)context hint:(M13_NULLABLE NSData *)hint;
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)sortedByObjectsUsingDescriptors:(NSArray *)descriptors;
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)sortedByKeysUsingDescriptors:(NSArray *)descriptors;
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)sortedByObjectsUsingSelector:(SEL)comparator;
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)sortedByKeysUsingSelector:(SEL)comparator;
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)sortedByObjectsUsingComparator:(NSComparator)cmptr;
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)sortedByKeysUsingComparator:(NSComparator)cmptr;
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)sortedByObjectsWithOptions:(NSSortOptions)opts usingComparator:(NSComparator)cmptr;
- (M13OrderedDictionary M13Generics(KeyType, ObjectType) *)sortedByKeysWithOptions:(NSSortOptions)opts usingComparator:(NSComparator)cmptr;
//写入文件
- (BOOL)writeToFile:(NSString *)path atomically:(BOOL)flag;
//写入到URL
- (BOOL)writeToURL:(NSURL *)aURL atomically:(BOOL)flag;
//添加监听
- (void)addObserver:(NSObject *)anObserver toObjectsAtIndices:(NSIndexSet *)indices forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(M13_NULLABLE void *)context;
- (void)addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(M13_NULLABLE void *)context;
//移除监听
- (void)removeObserver:(NSObject *)anObserver fromObjectsAtIndices:(NSIndexSet *)indices forKeyPath:(NSString *)keyPath;
- (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath context:(M13_NULLABLE void *)context;
//KVC相关方法
- (void)setValue:(M13_NULLABLE id)value forKey:(NSString *)key;
- (void)setValue:(M13_NULLABLE id)value forKeyPath:(NSString *)keyPath;
- (id)valueForKey:(NSString *)key;
- (id)valueForKeyPath:(NSString *)keyPath;
//归档相关方法
- (void)encodeWithCoder:(NSCoder *)aCoder;
- (id)initWithCoder:(NSCoder *)decoder;
//copy相关方法
- (id)copy;
- (id)copyWithZone:(M13_NULLABLE NSZone *)zone;
- (id)mutableCopy;
- (id)mutableCopyWithZone:(NSZone *)zone;
```

### 三、M13MutableOrderedDictionary

        基于M13OrderedDictionary，M13MutableOrderedDictionary为可变的有序字典类，其中方法解析如下：

```objectivec
//类创建方法
+ (instancetype)orderedDictionaryWithCapacity:(NSUInteger)numEntries;
//初始化方法
- (id)initWithCapacity:(NSUInteger)numEntries;
//添加键值对的方法
- (void)addObject:(M13GenericType(ObjectType, id))object pairedWithKey:(M13GenericType(KeyType, id<NSCopying>))key;
- (void)addEntry:(NSDictionary M13Generics(KeyType, ObjectType) *)entry;
- (void)addEntriesFromOrderedDictionary:(M13OrderedDictionary M13Generics(KeyType, ObjectType) *)orderedDictionary;
- (void)addEntriesFromDictionary:(NSDictionary M13Generics(KeyType, ObjectType) *)dictionary;
//插入键值对的方法
- (void)insertObject:(M13GenericType(ObjectType, id))object pairedWithKey:(M13GenericType(KeyType, id<NSCopying>))key atIndex:(NSUInteger)index;
- (void)insertEntry:(NSDictionary M13Generics(KeyType, ObjectType) *)entry atIndex:(NSUInteger)index;
- (void)insertEntriesFromOrderedDictionary:(M13OrderedDictionary M13Generics(KeyType, ObjectType) *)orderedDictionary atIndex:(NSUInteger)index;
- (void)insertEntriesFromDictionary:(NSDictionary M13Generics(KeyType, ObjectType) *)dictionary atIndex:(NSUInteger)index;
//设置键值对的方法
- (void)setObject:(M13GenericType(ObjectType, id))object forKey:(M13GenericType(KeyType, id<NSCopying>))aKey;
- (void)setEntry:(NSDictionary M13Generics(KeyType, ObjectType) *)entry;
- (void)setEntriesFromOrderedDictionary:(M13OrderedDictionary M13Generics(KeyType, ObjectType) *)orderedDictionary;
- (void)setEntriesFromDictionary:(NSDictionary M13Generics(KeyType, ObjectType) *)dictionary;
- (void)setObject:(M13GenericType(ObjectType, id))object forKey:(M13GenericType(KeyType, id<NSCopying>))aKey atIndex:(NSUInteger)index;
- (void)setEntry:(NSDictionary M13Generics(KeyType, ObjectType) *)entry atIndex:(NSUInteger)index;
- (void)setEntriesFromOrderedDictionary:(M13OrderedDictionary M13Generics(KeyType, ObjectType) *)orderedDictionary atIndex:(NSUInteger)index;
- (void)setEntriesFromDictionary:(NSDictionary M13Generics(KeyType, ObjectType) *)dictionary atIndex:(NSUInteger)index;
//移除键值对的方法
- (void)removeObjectForKey:(M13GenericType(KeyType, id<NSCopying>))key;
- (void)removeObjectsForKeys:(NSArray M13Generics(KeyType) *)keys;
- (void)removeAllObjects;
- (void)removeAllEntries;
- (void)removeLastEntry;
- (void)removeEntryWithObject:(M13GenericType(ObjectType, id))object;
- (void)removeEntryWithKey:(M13GenericType(KeyType, id<NSCopying>))key;
- (void)removeEntryWithObject:(M13GenericType(ObjectType, id))object pairedWithKey:(M13GenericType(KeyType, id<NSCopying>))key;
- (void)removeEntry:(NSDictionary M13Generics(KeyType, ObjectType) *)entry;
- (void)removeEntryWithObject:(M13GenericType(ObjectType, id))object inRange:(NSRange)range;
- (void)removeEntryWithKey:(M13GenericType(KeyType, id<NSCopying>))key inRange:(NSRange)range;
- (void)removeEntryWithObject:(M13GenericType(ObjectType, id))object pairedWithKey:(M13GenericType(KeyType, id<NSCopying>))key inRange:(NSRange)ramge;
- (void)removeEntry:(NSDictionary M13Generics(KeyType, ObjectType) *)entry inRange:(NSRange)range;
- (void)removeEntryAtIndex:(NSUInteger)index;
- (void)removeEntriesAtIndices:(NSIndexSet *)indices;
- (void)removeEntryWithObjectIdenticalTo:(M13GenericType(ObjectType, id))anObject;
- (void)removeEntryWithObjectIdenticalTo:(M13GenericType(ObjectType, id))anObject inRange:(NSRange)range;
- (void)removeEntriesWithObjectsInArray:(NSArray M13Generics(ObjectType) *)array;
- (void)removeEntriesWithKeysInArray:(NSArray M13Generics(KeyType) *)array;
- (void)removeEntriesInRange:(NSRange)range;
//替换键值对的方法
- (void)replaceEntryAtIndex:(NSInteger)index withObject:(M13GenericType(ObjectType, id))object pairedWithKey:(M13GenericType(KeyType, id<NSCopying>))key;
- (void)replaceEntryAtIndex:(NSUInteger)index withEntry:(NSDictionary M13Generics(KeyType, ObjectType) *)entry;
- (void)replaceEntriesAtIndices:(NSIndexSet *)indices
                    withObjects:(NSArray M13Generics(ObjectType) *)objects
                 pairedWithKeys:(NSArray M13Generics(KeyType) *)keys;
- (void)replaceEntriesAtIndices:(NSIndexSet *)indices
                    withEntries:(NSArray M13Generics(NSDictionary<KeyType, ObjectType> *) *)orderedEntries;
- (void)replaceEntriesAtIndices:(NSIndexSet *)indices
withEntriesFromOrderedDictionary:(M13OrderedDictionary M13Generics(KeyType, ObjectType) *)orderedDictionary;
- (void)replaceEntriesInRange:(NSRange)range
         withObjectsFromArray:(NSArray M13Generics(ObjectType) *)objects
      pairedWithKeysFromArray:(NSArray M13Generics(KeyType) *)keys
                      inRange:(NSRange)range2;
- (void)replaceEntriesInRange:(NSRange)range
              withEntriesFrom:(NSArray M13Generics(NSDictionary<KeyType, ObjectType> *) *)orderedEntries
                      inRange:(NSRange)range2;
- (void)replaceEntriesInRange:(NSRange)range
withEntriesFromOrderedDictionary:(M13OrderedDictionary M13Generics(KeyType, ObjectType) *)dictionary
                      inRange:(NSRange)range2;
- (void)replaceEntriesInRange:(NSRange)range
         withObjectsFromArray:(NSArray M13Generics(ObjectType) *)objects
      pairedWithKeysFromArray:(NSArray M13Generics(KeyType) *)keys;
- (void)replaceEntriesInRange:(NSRange)range
              withEntriesFrom:(NSArray M13Generics(NSDictionary<KeyType, ObjectType> *) *)orderedEntries;
- (void)replaceEntriesInRange:(NSRange)range
withEntriesFromOrderedDictionary:(M13OrderedDictionary M13Generics(KeyType, ObjectType) *)dictionary;
- (void)setEntriesToObjects:(NSArray M13Generics(ObjectType) *)objects pairedWithKeys:(NSArray M13Generics(KeyType) *)keys;
- (void)setEntriesToOrderedDictionary:(M13OrderedDictionary M13Generics(KeyType, ObjectType) *)orderedDictionary;
//进行元素筛选
- (void)filterEntriesUsingPredicateForObjects:(NSPredicate *)predicate;
//进行元素交换
- (void)exchangeEntryAtIndex:(NSUInteger)idx1 withEntryAtIndex:(NSUInteger)idx2;
//进行元素排序
- (void)sortEntriesByObjectUsingDescriptors:(NSArray *)descriptors;
- (void)sortEntriesByKeysUsingDescriptors:(NSArray *)descriptors;
- (void)sortEntriesByObjectUsingComparator:(NSComparator)cmptr;
- (void)sortEntriesByKeysUsingComparator:(NSComparator)cmptr;
- (void)sortEntriesByObjectWithOptions:(NSSortOptions)opts usingComparator:(NSComparator)cmptr;
- (void)sortEntriesByKeysWithOptions:(NSSortOptions)opts usingComparator:(NSComparator)cmptr;
- (void)sortEntriesByObjectUsingFunction:(NSInteger (*)(M13GenericType(ObjectType, id),
                                                        M13GenericType(ObjectType, id),
                                                        void * M13__NULLABLE))compare
                                 context:(M13_NULLABLE void *)context;
- (void)sortEntriesByKeysUsingFunction:(NSInteger (*)(M13GenericType(KeyType, id<NSCopying>),
                                                      M13GenericType(KeyType, id<NSCopying>),
                                                      void * M13__NULLABLE))compare
                               context:(M13_NULLABLE void *)context;
- (void)sortEntriesByObjectUsingSelector:(SEL)comparator;
- (void)sortEntriesByKeysUsingSelector:(SEL)comparator;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

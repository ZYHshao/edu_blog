---
title: 追根问底：Objective-C关联属性原理分析
date: 2022-07-17
categories: 编程珠玑
tags: []
---
# 追根问底：Objective-C关联属性原理分析

## 一.引子

Objective-C是一种动态性很强的语言，所谓动态能力，也可以理解为运行时能力。对于Objective-C开发者来说，动态性所带来的编程便利无处不在。例如通过Category类别来扩展已有类的功能。可以使已有类拥有新的方法和属性。但是，如果你有使用Category来扩展类的属性，你一定了解并非简单的使用@property进行声明即可。例如下面的代码：

```objectivec
#import <Foundation/Foundation.h>
@interface MyObject : NSObject

@end

@implementation MyObject

@end


@interface MyObject (Property)

@property (nonatomic, copy) NSString *addProperty;

@end

@implementation MyObject (Property)

@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        MyObject *object = [[MyObject alloc] init];
        object.addProperty = @"HelloWorld";
        NSLog(@"%@", object.addProperty);
    }
    return 0;
}

```

代码在编译时不会有任何问题，但是如果运行，就会出现未定义的方法异常。因此如果要扩展类的属性，我们通常会这样实现：

```objectivec
#import <objc/runtime.h>

@implementation MyObject (Property)

static NSString *kAddPropertyKey = @"kAddPropertyKey";

- (void)setAddProperty:(NSString *)addProperty {
    objc_setAssociatedObject(self, kAddPropertyKey, addProperty, OBJC_ASSOCIATION_COPY);
}

- (NSString *)addProperty {
    return objc_getAssociatedObject(self, kAddPropertyKey);
}

@end
```

再次运行，就可以正常的对addProperty属性进行存取值了。这里其实就使用到了Objective-C运行时的特性，在Objective-C中，类对象在创建时其所占用的内存空间就已经确定，那么你有没有想过，通过objc_setAssociatedObject这个运行时方法所存储的属性值是如何与当前对象关联起来的，这些数据又是存在哪里的？幸运的时，从objc源码可以清楚的了解关联属性的实现逻辑，这也是我们本篇文章要讨论的重点，了解这里的原理可能不能对你使用关联属性提供多大的帮助，但是这种设计思路定会使你在日常开发中受益匪浅。

## 二. objc_setAssociatedObject方法的核心原理

通过objc的runtime源码，我们可以看到objc_setAssociatedObject的方法实现如下：

```objectivec
void
objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
{
    _object_set_associative_reference(object, key, value, policy);
}

```

这一步无需过多解释，只是调用了一个内部函数，\_object\_set\_associative\_reference内部函数是关联属性实现的核心，此函数解析如下：

```objectivec
void
_object_set_associative_reference(id object, const void *key, id value, uintptr_t policy)
{
    // 1.安全性检查，要关联的对象和要关联的值不能为空
    if (!object && !value) return;

    // 2.检查当前对象是否不允许关联属性，某些类不允许其实例有关联属性
    if (object->getIsa()->forbidsAssociatedObjects())
        _objc_fatal("objc_setAssociatedObject called on instance (%p) of class %s which does not allow associated objects", object, object_getClassName(object));
    
    // 3.创建一个包装对象指针的结构对象，存储要关联的对象指针
    DisguisedPtr<objc_object> disguised{(objc_object *)object};
    // 4.创建一个包装关联策略和被关联值的对象
    ObjcAssociation association{policy, value};

    // 5.根据关联策略来对值进行引用（retain或copy）
    association.acquireValue();

    bool isFirstAssociation = false;
    {
        // 6.获取关联管理器及其中的关联表对象
        AssociationsManager manager;
        AssociationsHashMap &associations(manager.get());

        // 7.值如果存在，则进行关联
        if (value) {
            // 8.尝试向表中插入当前要关联的对象和值，如果已经存在，则什么都不做
            auto refs_result = associations.try_emplace(disguised, ObjectAssociationMap{});
            // 9. 检查try_emplace方法的插入结果，如果做了插入操作，则标记首次关联为true
            if (refs_result.second) {
                isFirstAssociation = true;
            }

            // 10.将表中存储的值与key进行关联
            auto &refs = refs_result.first->second;
            auto result = refs.try_emplace(key, std::move(association));
            // 11.如果key已经存在，则进行关联策略和关联值的交换
            if (!result.second) {
                association.swap(result.first->second);
            }
        // 12. 要关联的值为nil，则为清除操作
        } else {
            // 13. 查找到关联到此对象的属性对
            auto refs_it = associations.find(disguised);
            if (refs_it != associations.end()) {
                // 14.有关联属性，获取存储key的表
                auto &refs = refs_it->second;
                // 15.查找对应key是否存在
                auto it = refs.find(key);
                if (it != refs.end()) {
                    // 16.存在则进行关联数据的替换，包括关联策略和值，此时实际上是将值清空了
                    association.swap(it->second);
                    // 17.相关擦除操作
                    refs.erase(it);
                    if (refs.size() == 0) {
                        associations.erase(refs_it);
                    }
                }
            }
        }
    }

    // 18.判断是否为此类实例对象的第一次关联，如果是，则修改标记位，标明已经有关联属性
    if (isFirstAssociation)
        object->setHasAssociatedObjects();

    // 19.将旧的值进行release，如果需要的话
    association.releaseHeldValue();
}

```

可以看到，整个关联属性的过程非常清晰，对于新值是否需要retain以及旧值是否需要release，是由关联策略决定的：

```objectivec
enum {
    OBJC_ASSOCIATION_SETTER_ASSIGN      = 0,    // assgin属性
    OBJC_ASSOCIATION_SETTER_RETAIN      = 1,    // 设置值的时候需要retain
    OBJC_ASSOCIATION_SETTER_COPY        = 3,    // 设置值的时候需要copy
    OBJC_ASSOCIATION_GETTER_READ        = (0 << 8), // readonly的属性
    OBJC_ASSOCIATION_GETTER_RETAIN      = (1 << 8), // 获取值的时候需要retain
    OBJC_ASSOCIATION_GETTER_AUTORELEASE = (2 << 8), // 获取值的时候需要autorelease
    OBJC_ASSOCIATION_SYSTEM_OBJECT      = _OBJC_ASSOCIATION_SYSTEM_OBJECT, // 1 << 16
};
```

acquireValue方法实现如下，其只是判断是否需要retain和copy，之后调用对应的函数：

```objectivec
inline void acquireValue() {
    if (_value) {
        switch (_policy & 0xFF) {
        case OBJC_ASSOCIATION_SETTER_RETAIN:
            _value = objc_retain(_value);
            break;
        case OBJC_ASSOCIATION_SETTER_COPY:
            _value = ((id(*)(id, SEL))objc_msgSend)(_value, @selector(copy));
            break;
        }
    }
}
```

在上面第8步中，有调用try_emplace方法来将数据插入到表结构中，此函数插入时会判断要插入的数据是否存在，其返回值会告知调用者是否产生了插入操作，如果已经存在，则此函数会什么都不做。

## 三. 获取和移除关联属性的原理

现在，我们已经基本清楚了关联属性是如何设置和存储的，再来理解如果获取和移除就非常容易了。

获取关联属性的值是使用objc_getAssociatedObject运行时方法实现的，此方法实现如下：

```objectivec
id
objc_getAssociatedObject(id object, const void *key)
{
    return _object_get_associative_reference(object, key);
}
```

我们还是主要来解析下其调用的\_object\_get\_associative\_reference内部方法：

```objectivec
id
_object_get_associative_reference(id object, const void *key)
{
    // 1.创建关联对象结构
    ObjcAssociation association{};
    {   
        // 2.获取关联管理器及全局的Hash表
        AssociationsManager manager;
        AssociationsHashMap &associations(manager.get());
        // 3.尝试查找当前传入对象的关联属性
        AssociationsHashMap::iterator i = associations.find((objc_object *)object);
        if (i != associations.end()) {
            // 4.如果当前对象有关联属性，尝试查找存储key的列表中是否存在传入的key
            ObjectAssociationMap &refs = i->second;
            ObjectAssociationMap::iterator j = refs.find(key);
            if (j != refs.end()) {
                // 5.如果可以查到，对association进行赋值
                association = j->second;
                // 6.根据关联策略来决定是否对返回的值进行retain
                association.retainReturnedValue();
            }
        }
    }
    // 7.根据返回策略来决定是否需要autorelease，如果没有查到，会返回nil值
    return association.autoreleaseReturnedValue();
}
```

对于已经关联了属性的对象，我们也可以调用objc_removeAssociatedObjects方法来将关联的所有属性进行移除，此方法实现如下：

```objectivec
void objc_removeAssociatedObjects(id object) 
{
    // 对象存在，并且已经标记过关联属性
    if (object && object->hasAssociatedObjects()) {
        _object_remove_assocations(object, /*deallocating*/false);
    }
}
```

\_object\_remove_assocation内部函数的实现也不复杂，解析如下：

```objectivec
void
_object_remove_assocations(id object, bool deallocating)
{
    // 1.创建关联对象
    ObjectAssociationMap refs{};
    {
        // 2.获取关联管理器及Hash表
        AssociationsManager manager;
        AssociationsHashMap &associations(manager.get());
        // 3.查找传入对象的关联属性
        AssociationsHashMap::iterator i = associations.find((objc_object *)object);
        if (i != associations.end()) {
            // 4.查找到后，用空值进行交换
            refs.swap(i->second);
            // 5.如果是系统对象，则需要保留关联
            bool didReInsert = false;
            if (!deallocating) {
                for (auto &ref: refs) {
                    if (ref.second.policy() & OBJC_ASSOCIATION_SYSTEM_OBJECT) {
                        i->second.insert(ref);
                        didReInsert = true;
                    }
                }
            }
            // 6.无需保留关联，则直接清除数据
            if (!didReInsert)
                associations.erase(i);
        }
    }
    // 7.对旧值进行release操作
    SmallVector<ObjcAssociation *, 4> laterRefs;
    for (auto &i: refs) {
        if (i.second.policy() & OBJC_ASSOCIATION_SYSTEM_OBJECT) {
            // If we are not deallocating, then RELEASE_LATER associations don't get released.
            if (deallocating)
                laterRefs.append(&i.second);
        } else {
            i.second.releaseHeldValue();
        }
    }
    for (auto *later: laterRefs) {
        later->releaseHeldValue();
    }
}
```

## 四. 关联属性如何进行内存管理？

通过前面的介绍，我们知道在关联属性时，可以通过关联策略来设置一些和内存管理相关的选项，在设置关联属性时，如果需要的话，其内部会根据内存管理策略对旧值进行release操作，但是你是否有想过，当对象正常的生命周期结束后，这些关联属性占用的的内存是如何回收的？这就需要我们从系统的dealloc方法中寻找答案了。

系统对象在销毁时，dealloc方法最终会执行到一个名为objc_destructInstance的内部函数，此函数实现如下：

```objectivec
void *objc_destructInstance(id obj) 
{
    if (obj) {
        bool cxx = obj->hasCxxDtor();
        // 通过标记获取此对象是否有关联属性
        bool assoc = obj->hasAssociatedObjects();
        if (cxx) object_cxxDestruct(obj);
        // 移除掉此对象的关联属性
        if (assoc) _object_remove_assocations(obj, /*deallocating*/true);
        obj->clearDeallocating();
    }

    return obj;
}

```

其中会判断要销毁的对象是否有关联属性，如果有，又会调用到\_object\_remove_assocation函数来进行关联属性的移除，这个函数前面介绍过，内部会处理内存管理问题。

## 五.关联管理器与表的创建时机

在整个关联属性实现方案中，还有一点我们没有闭环介绍，即全局的关联管理器和Hash表是怎么创建的，何时创建的。我们目前只看到，当要设置或获取关联属性时，直接拿到管理器和Hash表进行使用，并无初始化。其实，这些全局数据结构的创建在runtime初始化时就已经完成，流程路径如下：

1\. 调用runtime入口函数\_objc\_init

2\. 通知调用map_images函数

3\. 调用map\_images\_nolock函数

4. map\_images\_nolock其中会调用arr_init函数，此函数实现如下：

```objectivec
void arr_init(void) 
{
    AutoreleasePoolPage::init();
    SideTablesMap.init();
    _objc_associations_init();
    if (DebugScanWeakTables)
        startWeakTableScan();
}
```

可以看到，此函数会进行自动释放池，关联属性等逻辑的初始化。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> —— 珲少 QQ：316045346
> 
> 同时，如果本篇文章让你觉得有用，欢迎分享给更多朋友，请标明出处。
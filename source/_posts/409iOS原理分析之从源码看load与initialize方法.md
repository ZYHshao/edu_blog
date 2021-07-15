---
title: iOS原理分析之从源码看load与initialize方法
date: 2021-02-18
categories: iOS逻辑初窥
tags: []
---
# iOS原理分析之从源码看load与initialize方法

## 一、引言

    在iOS开发中，NSObject类是万事万物的基类，其在Objective-C的整理类架构中非常重要，其中有两个很有名的方法：load方法与initialize方法。

```objectivec
+ (void)load;
+ (void)initialize;
```

说起这两个方法，你的第一反应一定是觉得太老套了，这两个方法的调用时机及作用几乎成为了iOS面试的必考题。其本身调用时机也非常简单：

1\. load方法在pre-main阶段被调用，每个类都会调用且只会调用一次。

2\. initialize方法在类或子类第一次进行方法调用前会调用。

上面的两点说明本身是正确的，但是除此之外，还有许多问题值得我们深究，例如：

1\. 子类与父类的load方法的调用顺序是怎样的？

2\. 类与分类的load方法调用顺序是怎样的？

3\. 子类未实现load方法，会调用父类的么？

4\. 当有多个分类都实现了load方法时，会怎么样？

5\. 每个类的load方法的调用顺序是怎样的？

6\. 父类与子类的initialize的方法调用顺序是怎样的？

7\. 子类实现initialize方法后，还会调用父类的initialize方法么？

8\. 多个分类都实现了initialize方法后，会怎么样？

9\. ...

如上所提到的问题，你现在都能给出明确的答案么？其实，load与initialize方法本身还有许多非常有意思的特点，本篇博客，我们将结合Objective-C源码，对这两个方法的实现原理做深入的分析，相信，如果你对load与initialize还不够了解，不能完全明白上面所提出的问题，那么本篇博客将会使其收获满满。无论在以后的面试中，还是工作中使用到load和initialize方法时，都可能帮助你从源码上理解其执行原理。

## 二、实践出真知 \- 先看load方法

    在开始分析之前，我们首先可以先创建一个测试工程，对load方法的执行时机先做一个简单的测试。首先，我们创建一个Xcode的命令行程序工程，在其中创建一些类、子类和分类，方便我们测试，目录结构如下图所示：

![](https://oscimg.oschina.net/oscnet/up-80c97d2af95aa24b10cf29505a3b6b5012e.png)

其中，MyObjectOne和MyObjectTwo都是继承自NSObject的类，MySubObjectOne是MyObjectOne的子类，MySubObjectTwo是MyObjectTwo的子类，同时我们还创建了3个分类，在类中实现load方法，并做打印处理，如下：

```objectivec
+ (void)load {
    NSLog(@"load:%@", [self className]);
}
```

同样，类似的也在分类中做实现：

```objectivec
+ (void)load {
    NSLog(@"load-category:%@", [self className]);
}
```

最后我们在main函数中添加一个Log：

```objectivec
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSLog(@"Main");
    }
    return 0;
}

```

运行工程，打印结果如下：

```
2021-02-18 14:33:46.773294+0800 KCObjc[21400:23090040] load:MyObjectOne
2021-02-18 14:33:46.773867+0800 KCObjc[21400:23090040] load:MySubObjectOne
2021-02-18 14:33:46.773959+0800 KCObjc[21400:23090040] load:MyObjectTwo
2021-02-18 14:33:46.774008+0800 KCObjc[21400:23090040] load:MySubObjectTwo
2021-02-18 14:33:46.774052+0800 KCObjc[21400:23090040] load-category:MyObjectTwo
2021-02-18 14:33:46.774090+0800 KCObjc[21400:23090040] load-category:MyObjectOne
2021-02-18 14:33:46.774127+0800 KCObjc[21400:23090040] load-category:MyObjectOne
2021-02-18 14:33:46.774231+0800 KCObjc[21400:23090040] Main
```

从打印结果可以看出，load方法在main方法开始之前被调用，执行顺序上来说，先调用类的load方法，再调用分类的load方法，从父子类的关系上看来，先调用父类的load方法，再调用子类的load方法。

    下面，我们就从源码上来分析下，系统如此调用load方法，是源自于什么样的奥妙。

## 三、从源码分析load方法的调用

    要深入的研究load方法，我们首先需要从Objective-C的初始化函数说起：

```objectivec
void _objc_init(void)
{
    static bool initialized = false;
    if (initialized) return;
    initialized = true;
    
    // fixme defer initialization until an objc-using image is found?
    environ_init();
    tls_init();
    static_init();
    runtime_init();
    exception_init();
    cache_init();
    _imp_implementationWithBlock_init();

    // 其他的我们都不需要关注，只需要关注这行代码
    _dyld_objc_notify_register(&map_images, load_images, unmap_image);

#if __OBJC2__
    didCallDyldNotifyRegister = true;
#endif
}

```

\_objc\_init函数定义在objc-os.mm文件中，这个函数用来做Objective-C程序的初始化，由引导程序进行调用，其调用实际会非常的早，并且是操作系统引导程序复杂调用驱动，对开发者无感。在\_objc\_init函数中，会进行环境的初始化，runtime的初始化以及缓存的初始化等等操作，其中很重要的一步操作是执行\_dyld\_objc\_notify\_register函数，这个函数会调用load_images函数来进行镜像的加载。

    load方法的调用，其实就是类加载过程中的一步，首先，我们先来看一个load_images函数的实现：

```objectivec
void
load_images(const char *path __unused, const struct mach_header *mh)
{
    if (!didInitialAttachCategories && didCallDyldNotifyRegister) {
        didInitialAttachCategories = true;
        loadAllCategories();
    }

    // Return without taking locks if there are no +load methods here.
    if (!hasLoadMethods((const headerType *)mh)) return;

    recursive_mutex_locker_t lock(loadMethodLock);

    // Discover load methods
    {
        mutex_locker_t lock2(runtimeLock);
        prepare_load_methods((const headerType *)mh);
    }

    // Call +load methods (without runtimeLock - re-entrant)
    call_load_methods();
}
```

滤掉其中我们不关心的部分，与load方法调用相关的核心如下：

```objectivec
void
load_images(const char *path __unused, const struct mach_header *mh)
{
    // 镜像中没有load方法，直接返回
    if (!hasLoadMethods((const headerType *)mh)) return;
    {
        // 准备load方法
        prepare_load_methods((const headerType *)mh);
    }
    // 进行load方法的调用
    call_load_methods();
}
```

最核心的部分在于load方法的准备与laod方法的调用，我们一步一步看，先来看load方法的准备(我们去掉了无关紧要的部分)：

```objectivec
void prepare_load_methods(const headerType *mhdr)
{
    size_t count, i;
    // 获取所有类 组成列表
    classref_t const *classlist = 
        _getObjc2NonlazyClassList(mhdr, &count);
    for (i = 0; i < count; i++) {
        // 将所有类的load方法进行整理
        schedule_class_load(remapClass(classlist[i]));
    }
    // 获取所有的分类 组成列表
    category_t * const *categorylist = _getObjc2NonlazyCategoryList(mhdr, &count);
    for (i = 0; i < count; i++) {
        category_t *cat = categorylist[i];
        // 将分类的load方法进行整理
        add_category_to_loadable_list(cat);
    }
}
```

看到这里，我们基本就有头绪了，load方法的调用顺序，基本可以确定是由整理过程所决定的，并且我们可以发现，类的load方法整理与分类的load方法整理是互相独立的，因此也可以推断其调用的时机也是独立的。首先我们先来看类的load方法整理函数schedule\_class\_load(去掉无关代码后)：

```objectivec
static void schedule_class_load(Class cls)
{
    // 类不存在或者已经加载过load，则return
    if (!cls) return;
    if (cls->data()->flags & RW_LOADED) return;

    // 保证加载顺序，递归进行父类加载
    schedule_class_load(cls->superclass);
    // 将当前类的load方法加载进load方法列表中
    add_class_to_loadable_list(cls);
    // 将当前类设置为已经加载过laod
    cls->setInfo(RW_LOADED); 
}
```

可以看到，schedule\_class\_load函数中使用了递归的方式演着继承链逐层向上，保证在加载load方法时，先加载父类，再加载子类。add\_class\_to\_loadable\_list是核心的load方法整理函数，如下(去掉了无关代码)：

```objectivec
void add_class_to_loadable_list(Class cls)
{
    IMP method;
    // 读取类中的load方法
    method = cls->getLoadMethod();
    if (!method) return; // 类中没有实现load方法，直接返回
    // 构建存储列表及扩容逻辑
    if (loadable_classes_used == loadable_classes_allocated) {
        loadable_classes_allocated = loadable_classes_allocated*2 + 16;
        loadable_classes = (struct loadable_class *)
            realloc(loadable_classes,
                              loadable_classes_allocated *
                              sizeof(struct loadable_class));
    }
    // 向列表中添加 loadable_class 结构体，这个结构体中存储了类与对应的laod方法
    loadable_classes[loadable_classes_used].cls = cls;
    loadable_classes[loadable_classes_used].method = method;
    // 标记列表index的指针移动
    loadable_classes_used++;
}
```

loadable_clas结构体的定义如下：

```objectivec
struct loadable_class {
    Class cls;  // may be nil
    IMP method;
};
```

getLoadMetho函数的实现主要是从类中获取到load方法的实现，如下：

```objectivec
IMP 
objc_class::getLoadMethod()
{
    // 获取方法列表
    const method_list_t *mlist;
    mlist = ISA()->data()->ro()->baseMethods();
    if (mlist) {
        // 遍历，找到load方法返回
        for (const auto& meth : *mlist) {
            const char *name = sel_cname(meth.name);
            if (0 == strcmp(name, "load")) {
                return meth.imp;
            }
        }
    }
    return nil;
}
```

现在，关于类的load方法的准备逻辑已经非常清晰了，最终会按照先父类后子类的顺序将所有类的load方法添加进名为loadable\_classes的列表中，loadable\_classes这个名字你要注意一下，后面我们还会遇到它。

    我们再来看分类的laod方法准备过程，其与我们上面介绍的类非常相似，add\_category\_to\_loadable\_list函数简化后如下：

```objectivec
void add_category_to_loadable_list(Category cat)
{
    IMP method;
    // 获取当前分类的load方法
    method = _category_getLoadMethod(cat);
    if (!method) return;
    // 列表创建与扩容逻辑
    if (loadable_categories_used == loadable_categories_allocated) {
        loadable_categories_allocated = loadable_categories_allocated*2 + 16;
        loadable_categories = (struct loadable_category *)
            realloc(loadable_categories,
                              loadable_categories_allocated *
                              sizeof(struct loadable_category));
    }
    // 将分类与load方法进行存储
    loadable_categories[loadable_categories_used].cat = cat;
    loadable_categories[loadable_categories_used].method = method;
    loadable_categories_used++;
}
```

可以看到，最终分类的load方法是存储在了loadable_categories列表中。

    准备好了load方法，我们再来分析下load方法的执行过程，call\_load\_methods函数的核心实现如下：

```objectivec
void call_load_methods(void)
{
    bool more_categories;
    do {
        // 先对 loadable_classes 进行遍历，loadable_classes_used这个字段可以理解为列表的元素个数
        while (loadable_classes_used > 0) {
            call_class_loads();
        }

        // 再对类别进行遍历调用
        more_categories = call_category_loads();
       
    } while (loadable_classes_used > 0  ||  more_categories);
 
}
```

call\_class\_loads函数实现简化后如下：

```objectivec
static void call_class_loads(void)
{
    int i;
    // loadable_classes列表
    struct loadable_class *classes = loadable_classes;
    // 需要执行load方法个数
    int used = loadable_classes_used;
    // 清理数据
    loadable_classes = nil;
    loadable_classes_allocated = 0;
    loadable_classes_used = 0;
    // 循环进行执行 循环的循序是从前到后
    for (i = 0; i < used; i++) {
        // 获取类
        Class cls = classes[i].cls;
        // 获取对应load方法
        load_method_t load_method = (load_method_t)classes[i].method;
        if (!cls) continue; 
        // 执行load方法
        (*load_method)(cls, @selector(load));
    }
}
```

call\_category\_loads函数的实现要复杂一些，简化后如下：

```objectivec
static bool call_category_loads(void)
{
    int i, shift;
    bool new_categories_added = NO;
    
    // 获取loadable_categories分类load方法列表
    struct loadable_category *cats = loadable_categories;
    int used = loadable_categories_used;
    int allocated = loadable_categories_allocated;
    loadable_categories = nil;
    loadable_categories_allocated = 0;
    loadable_categories_used = 0;

    // 从前往后遍历进行load方法的调用
    for (i = 0; i < used; i++) {
        Category cat = cats[i].cat;
        load_method_t load_method = (load_method_t)cats[i].method;
        Class cls;
        if (!cat) continue;
        cls = _category_getClass(cat);
        if (cls  &&  cls->isLoadable()) {
            (*load_method)(cls, @selector(load));
            cats[i].cat = nil;
        }
    }
    return new_categories_added;
}
```

现在，我相信你已经对load方法为何类先调用，分类后调用，并且为何父类先调用，子类后调用。但是还有一点，我们不甚明了，即类之间或分类之间的调用顺序是怎么确定的，从源码中可以看到，类列表是通过\_getObjc2NonlazyClassList函数获取的，同样分类的列表是通过\_getObjc2NonlazyCategoryList函数获取的。这两个函数获取到的类或分类的顺序实际上是与类源文件的编译顺序有关的，如下图所示：

![](https://oscimg.oschina.net/oscnet/up-8ef31810786ef312f5fedc6001fc20a1383.png)

可以看到，打印的load方法的执行顺序与源代码的编译顺序是一直的。

## 四、initialize方法分析

    我们可以采用和分析load方法时一样的策略来对initialize方法的执行情况，进行测试，首先将测试工程中所有类中添加initialize方法的实现。此时如果直接运行工程，你会发现控制台没有任何输出，这是由于只有第一次调用类的方法时，才会执行initialize方法，在main函数中编写如下测试代码：

```objectivec
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        [MySubObjectOne new];
        [MyObjectOne new];
        [MyObjectTwo new];
        NSLog(@"------------");
        [MySubObjectOne new];
        [MyObjectOne new];
        [MyObjectTwo new];
    }
    return 0;
}
```

运行代码控制台打印效果如下：

```
2021-02-18 21:29:55.761897+0800 KCObjc[43834:23521232] initialize-cateOne:MyObjectOne
2021-02-18 21:29:55.762526+0800 KCObjc[43834:23521232] initialize:MySubObjectOne
2021-02-18 21:29:55.762622+0800 KCObjc[43834:23521232] initialize-cate:MyObjectTwo
2021-02-18 21:29:55.762665+0800 KCObjc[43834:23521232] ------------
```

可以看到，打印数据都出现在分割线前，说明一旦一个类的initialize方法被调用后，后续再向这个类发送消息，也不会在调用initialize方法，还有一点需要注意，需要注意，如果对子类发送消息，父类的initialize会先调用，再调用子类的initialize，同时，分类中如果实现了initialize方法则会覆盖类本身的，并且分类的加载顺序靠后的会覆盖之前的。下面我们就通过源码来分析下initialize方法的这种调用特点。

    首先，在调用类的类方法时，会执行runtime中的class_getClassMethod方法来寻找实现函数，这个方法在源码中的实现如下：

```objectivec
Method class_getClassMethod(Class cls, SEL sel)
{
    if (!cls  ||  !sel) return nil;
    return class_getInstanceMethod(cls->getMeta(), sel);
}
```

通过源码可以看到，调用一个类的类方法，实际上是调用其元类的示例方法，getMeta函数用来获取类的元类，关于类和元类的相关组织原理，我们这里先不扩展。我们需要关注的是class_getInstanceMethod这个函数，这个函数的实现也非常简单，如下：

```objectivec
Method class_getInstanceMethod(Class cls, SEL sel)
{
    if (!cls  ||  !sel) return nil;

    // 做查询方法列表，尝试方法解析相关工作
    lookUpImpOrForward(nil, sel, cls, LOOKUP_RESOLVER);

    // 从类对象中获取方法
    return _class_getMethod(cls, sel);
}
```

在class\_getInstanceMethod方法的实现中，\_class_getMethod是最终获取要调用的方法的函数，在这之前，lookUpImpOrForward函数会做一些前置操作，其中就有initialize函数的调用逻辑，我们去掉无关的逻辑，lookUpImpOrForward中核心的实现如下：

```objectivec
IMP lookUpImpOrForward(id inst, SEL sel, Class cls, int behavior)
{
    IMP imp = nil;
    // 核心在于!cls->isInitialized() 如果当前类未初始化过，会执行initializeAndLeaveLocked函数
    if (slowpath((behavior & LOOKUP_INITIALIZE) && !cls->isInitialized())) {
        cls = initializeAndLeaveLocked(cls, inst, runtimeLock);
    }

    return imp;
}
```

initializeAndLeaveLocked会直接调用initializeAndMaybeRelock函数，如下：

```objectivec
static Class initializeAndLeaveLocked(Class cls, id obj, mutex_t& lock)
{
    return initializeAndMaybeRelock(cls, obj, lock, true);
}
```

initializeAndMaybeRelock函数中会做类的初始化逻辑，这个过程是线程安全的，其核心相关代码如下：

```objectivec
static Class initializeAndMaybeRelock(Class cls, id inst,
                                      mutex_t& lock, bool leaveLocked)
{
    // 如果已经初始化过，直接返回
    if (cls->isInitialized()) {
        return cls;
    }
    // 找到当前类的非元类
    Class nonmeta = getMaybeUnrealizedNonMetaClass(cls, inst);
    // 进行初始化操作
    initializeNonMetaClass(nonmeta);

    return cls;
}
```

initializeNonMetaClass函数会采用递归的方式沿着继承链向上查询，找到所有未初始化过的父类进行初始化，核心实现简化如下：

```objectivec
void initializeNonMetaClass(Class cls)
{
    Class supercls;
    // 标记是否需要初始化
    bool reallyInitialize = NO;
    // 父类如果存在，并且没有初始化过，则递归进行父类的初始化
    supercls = cls->superclass;
    if (supercls  &&  !supercls->isInitialized()) {
        initializeNonMetaClass(supercls);
    }
    
    SmallVector<_objc_willInitializeClassCallback, 1> localWillInitializeFuncs;
    {
        // 如果当前不是正在初始化，并且当前类没有初始化过
        if (!cls->isInitialized() && !cls->isInitializing()) {
            // 设置初始化标志，此类标记为初始化过
            cls->setInitializing();
            // 标记需要进行初始化
            reallyInitialize = YES;
        }
    }
    // 是否需要进行初始化
    if (reallyInitialize) {
        @try
        {
            // 调用初始化函数
            callInitialize(cls);
        }
        @catch (...) {
            @throw;
        }
        return;
    }
}

```

callInitialize函数最终会调用objc_msgSend函数来向类发送initialize初始化消息，如下：

```objectivec
void callInitialize(Class cls)
{
    ((void(*)(Class, SEL))objc_msgSend)(cls, @selector(initialize));
    asm("");
}
```

需要注意，initialize方法与load方法最大的区别在于其最终是通过objc\_msgSend来实现的，每个类如果未初始化过，都会通过objc\_msgSend来向类发送一次initialize消息，因此，如果子类没有对initialize实现，按照objc_msgSend的消息机制，其是会沿着继承链一路向上找到父类的实现进行调用的，所有initialize方法并不是只会被调用一次，假如父类中实现了这个方法，并且它有多个未实现此方法的子类，则当每个子类第一次接受消息时，都会调用一遍父类的initialize方法，这点非常重要，在实际开发中一定要牢记。

## 五、结语

    load和initialize方法是iOS开发中非常简单也也非常常用的两个方法，然而其与普通的方法比起来，还有有一些特殊，通过对源码的解读，我们可以更加深刻的理解这些特殊之处的原因及原理，编程的过程就像修行，知其然也知其所以然，与大家共勉。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：805263726

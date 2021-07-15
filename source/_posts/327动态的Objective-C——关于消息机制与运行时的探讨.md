---
title: 动态的Objective-C——关于消息机制与运行时的探讨
date: 2017-05-05
categories: iOS逻辑初窥
tags: []
---
## 动态的Objective-C——关于消息机制与运行时的探讨

### 一、引言

    Objective-C是一种很优美的语言，至少在我使用其进行编程的过程中，是很享受他那近乎自然语言的函数命名、灵活多样的方法调用方式以及配合IDE流顺畅快编写体验。Objective-C是扩展与C面向对象的编程语言，然而其方法的调用方式又和大多面向对象语言大有不同，其采用的是消息传递、转发的方式进行方法的调用。因此在Objective-C中对象的真正行为往往是在运行时确定而非在编译时确定，所以Objective-C又被称为是一种运行时的动态语言。

    本篇博客既不介绍iOS开发，也不提及MacOS开发，只对Objective-C语言的这种消息机制与运行时动态进行探讨，所提及的内容也都是我开发中的个人积累与经验，如果偏颇之处，欢迎讨论指正。

### 二、消息发送与转发机制

#### 1.初窥消息发送机制

    许多面向对象语言中方法的调用都是采用obj.function这样的方式，在Objective-C语言中却是采用中括号包裹的方式进行方法调用，例如\[obj function\]。实际上，Objective-C中的每一句方法调用最后都会转换成一条消息进行发送。一条消息包含3部分内容：方法选择器、接收消息的对象以及参数。objc_msgSend函数就是用来发送这种消息。例如，创建一个Xcode命令行工程，我们创建一个类，命名为MyObject，如下：

MyObject.h文件：

```objectivec
#import <Foundation/Foundation.h>

@interface MyObject : NSObject

@end

```

MyObject.m文件：

```objectivec
#import "MyObject.h"

@implementation MyObject

-(void)showSelf{
    NSLog(@"MyObject");
}

@end
```

首先在MyObject.h文件中并没有暴漏任何方法，MyObject.m文件中添加了一个showSelf方法，这个方法只是做了简单的打印操作。

    将main.m文件修改如下：

```objectivec
#import <Foundation/Foundation.h>
#import "MyObject.h"
#import <objc/message.h>
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        MyObject * obj = [[MyObject alloc]init];
        [obj class];
//为了消除未定义选择器的警告
#pragma clang diagnostic push
#pragma clang diagnostic ignored"-Wundeclared-selector"
        //进行消息发送
        ((void(*)(id,SEL))objc_msgSend)(obj,@selector(showSelf));
#pragma clang diagnostic pop
        
    }
    return 0;
}

```

运行工程，可以看到控制台执行了MyObject类的示例方法showSelf。如果要进行传参，在objc_msgSend方法中继续添加参数，并且指定对应的函数类型即可，例如：

MyObject.m文件：

```objectivec
#import "MyObject.h"

@implementation MyObject

-(void)showSelf:(NSString*)name age:(int)age{
    NSLog(@"MyObject:%@,%d",name,age);
}

@end
```

main.m文件：

```objectivec
#import <Foundation/Foundation.h>
#import "MyObject.h"
#import <objc/message.h>
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        MyObject * obj = [[MyObject alloc]init];
        [obj class];
//为了消除未定义选择器的警告
#pragma clang diagnostic push
#pragma clang diagnostic ignored"-Wundeclared-selector"
        //进行消息发送
        ((void(*)(id,SEL,NSString*,int))objc_msgSend)(obj,@selector(showSelf:age:),@"珲少",25);
#pragma clang diagnostic pop
        
    }
    return 0;
}
```

运行工程可以看到方法被调用，参数被正确传入。

#### 2.消息传递是基于继承链的

    上面代码只是简单演示了消息发送的效果，下面我们来剖析下消息发送的过程与原理，明白了这个原理，对Objective-C中许多神奇的现象你将会豁然开朗，后面我会再具体向你介绍这些现象。

    在介绍消息机制之前，我还是要再啰嗦一点，关于@selector()我们还需要深入理解一下，通过@selector(方法名)可以获取到一个SEL类型的对象，SEL实际上是objc\_selector结构体指针，在Objective-C库头文件中没有找到objc\_selector结构体的定义，但我们可以合理猜测，其中很有可能包含的是一个函数指针。因此SEL也可以理解为函数签名，在程序的编译阶段，我们定义类中所有所发会生成一个方法签名列表，这个列表时类直接关联的(原则上来说，类的本质也是对象，它是一个单例对象)，在运行时通过方法签名表来找到具体要执行的函数。

    我们再来看objc_msgSend()函数，前面说过，它的第一个参数为接收消息的对象，第2个参数为方法签名，之后为传递的参数。那么Objective-C运行时是如何根据一个对象实例来找到方法签名表，再找到要执行的方法呢，看似麻烦的事情其实原理也非常简单，细心观察，你会发现所有的NSObject子类对象中都包含一个isa成员变量，请看NSObject类的定义：

```objectivec
@interface NSObject <NSObject> {
    Class isa  OBJC_ISA_AVAILABILITY;
}
```

这个isa变量是Class类型，我们的主角终于来了，Class顾名思义就是“类”类型，其实质是objc_class结构体指针：

```objectivec
typedef struct objc_class *Class;
```

有些蒙圈了吧，不用着急，拨开层层迷雾，你就会发现Objective-C中类本质上只是结构体而已，下面是objc_class结构体的定义：

```objectivec
struct objc_class {
    //元类指针
    Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    //父类
    Class super_class                                        OBJC2_UNAVAILABLE;
    //类名
    const char *name                                         OBJC2_UNAVAILABLE;
    //类的版本
    long version                                             OBJC2_UNAVAILABLE;
    //信息
    long info                                                OBJC2_UNAVAILABLE;
    //内存布局
    long instance_size                                       OBJC2_UNAVAILABLE;
    //变量列表
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    //函数列表
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    //缓存方式
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    //协议列表
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
```

每一个“类”对象是也有一个isa指针，这个指针指向的类实际上是元类，即构造“类”的类。现在你无须纠结这些概念，举一个例子你就能明白，在Objective-C开发中有加方法与减方法，减方法是实例对象调用的方法，每一个“类”中都包含一个函数列表，就是上面的objc\_method\_list结构体数组指针，同样如果调用加方法，实际上是从类的元类中找到对应的方法列表，这个列表就是我们前面提到的方法签名列表，进行方法的执行。关于实例对象，“类”对象和元类，下图很好的表现了他们之间的关系：

![](https://static.oschina.net/uploads/space/2017/0503/150418_6vvT_2340880.png)

> 需要注意，使用LLDB调试器我们是可以拿到对象的isa指针的，并且可以看出它的确为Class类型，但是我们缺无法通过isa指针继续向下取抓取更多类的信息，其所在的内存是禁止我们访问的。但是Objective-C运行时提供了一些方法可以获取到这些信息，后面我们会一一介绍。

    上面我们介绍的消息发送机制其实十分不完整，首先Objective-C是支持继承的，因此如果在当前对象的类的方法列表中没有找到此消息对应的方法签名，系统会通过super_class一层层继续向上，直到找到相应的方法或者到达继承链的顶端。

    有了上面的理论知识作为基础，我们就可以更深入的分析消息传递的过程了，首先，如果消息的接收对象刚好可以处理这个消息，即其isa指针对应的类中可以查找到这个方法，那么万事大吉，找到对应方法直接执行就大功告成，可以如果接收对象无法处理，其父类，父父类...等都无法处理，那么该怎么办呢，Objective-C为了增强语言的动态性，如果真的出现了这种情况，程序并不会马上crash，在crash前，有3次机会可以挽救本条消息的命运。

#### 3.拯救未知消息的3根救命稻草

第一根救命稻草：

    如上所说，如果对象整个继承链都无法处理当前消息，那么首先会调用接收对象所属类的resolveInstanceMethod方法(这个对应实例方法，如果是无法处理的类方法消息，则会调用resolveClassMethod方法)，在这个方法中，开发者有机会为类动态添加方法，如果动态添加了方法，可以在这个方法中返回YES，那么此条消息依然会被成功处理。例如我们将main.m文件修改如下：

```objectivec
#import <Foundation/Foundation.h>
#import "MyObject.h"
#import <objc/message.h>
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        MyObject * obj = [[MyObject alloc]init];
        [obj class];
//为了消除未定义选择器的警告
#pragma clang diagnostic push
#pragma clang diagnostic ignored"-Wundeclared-selector"
        //进行消息发送
        ((void(*)(id,SEL))objc_msgSend)(obj,@selector(showSelf));
#pragma clang diagnostic pop
        
    }
    return 0;
}
```

MyObject类不做任何修改，当我们运行程序，程序会直接crash掉，现在我们在MyObject类中添加如下方法：

```objectivec
+(BOOL)resolveInstanceMethod:(SEL)sel{
    NSLog(@"resolveInstanceMethod");
    if ([NSStringFromSelector(sel) isEqualToString:@"showSelf"]) {
        class_addMethod(self, sel, newFunc, "v@:");
    }
    return [super resolveInstanceMethod:sel];
}
```

其中class_addMethod函数用来向类中动态添加方法，第一个参数为Class对象，第二个参数为方法选择器，第三个参数为IMP类型的函数指针，第四个参数为指定方法的返回值和参数类型。这个参数采用的是C字符串的形式来指定返回值和参数的类型，第1个字符为返回值类型，其后都为参数类型，需要注意，使用这种方式添加方法的时候系统会默认传入两个参数，分别是调用此方法的实例对象和方法选择器，上面示例代码中的"@"表示第1个id类型的参数，":"表示第2个选择器类型的参数，后面我会把字符所表示的参数类型映射表提供给大家。

    抽丝剥茧一下，IMP和SEL并不同，SEL可以理解为函数签名，其与函数名相关联，而IMP是函数所在地址的指针，其定义如下：

```objectivec
typedef void (*IMP)(void /* id, SEL, ... */ ); 
```

简单理解，通过IMP我们可以直接拿到函数的地址，后面会对函数做更深入的剖析，到时候你能就能豁然你开朗。   

    运行工程，根据打印信息可以看到showSelf方法被添加并正常执行了。

第二根救命稻草：

    抛开运行时添加方法这一手段，将resolveInstanceMethod方法删去，是不是我们的程序就必然走进crash的深渊了，其实不然，上帝还会给你另一根救命稻草，当通过运行时添加方法被否定后，系统会接着调用forwardingTargetForSelector方法，这个方法用来对消息进行转发，没错，重点来了，Objective-C中强大的消息转发机制的奥妙就在这里。forwardingTargetForSelector方法需要返回一个id类型的对象，系统会将当前对象服务处理的消息转发给这个方法返回的对象，如果这个返回的对象可以处理，那么程序依然可以很好的执行下去。

    例如，在我们的命令行工程中新添加一个类，命名为SubObject，实现如下：

SubObject.h文件：

```objectivec
#import <Foundation/Foundation.h>

@interface SubObject : NSObject

@end

```

SubObject.m文件：

```objectivec
#import "SubObject.h"

@implementation SubObject
-(void)showSelf{
    NSLog(@"subObject");
}
@end

```

在MyObject类中实现如下方法：

```objectivec
-(id)forwardingTargetForSelector:(SEL)aSelector{
    NSLog(@"forwardingTargetForSelector");
    if ([NSStringFromSelector(aSelector) isEqualToString:@"showSelf"]) {
        return [SubObject new];
    }
    return [super forwardingTargetForSelector:aSelector];
}
```

forwardingTargetForSelector方法可以返回一个对象，Objective-C会将当前对象无法处理的消息转发给这个方法返回的对象，如果返回nil，则表示不进行消息转发，这时你如果还想挽救此次crash，你就需要用到第三根救命稻草了。我们可以这种消息转发的机制来模拟Objective-C中的多继承。

第三根救命稻草：

    如果你不幸错过了前两次拯救未知消息的机会，那么你还有最后一次机会(中国有句古话，事不过三，世间万事也果真如此...)。当消息转发策略也被否定后，系统会调用methodSignatureForSelector方法，这个方法的主要用途是询问这个选择器是否是有效的，我们需要返回一个NSMethodSignature，顾名思义，这个对象是函数签名的抽象。如果我们返回了有效的函数签名，那么接着系统会调用forwardInvocation方法，这里是拯救应用程序的最后一根稻草了，这个函数会直接将消息包装成NSInvocation对象传入，我们直接将其发送给可以处理此消息的对象即可(当然你也可以直接抛弃，不理会这条未知的消息)。

    例如，在MyObject类中将forwardingTargetForSelector方法删去，实现如下两个方法：

```objectivec
//询问此选择器是否是有效的
-(NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector{
    NSLog(@"methodSignatureForSelector");
    if ([NSStringFromSelector(aSelector) isEqualToString:@"showSelf"]) {
       return [[SubObject new] methodSignatureForSelector:aSelector];
    }
    return [super methodSignatureForSelector:aSelector];
}
//处理消息
-(void)forwardInvocation:(NSInvocation *)anInvocation{
    NSLog(@"forwardInvocation");
    if ([NSStringFromSelector(anInvocation.selector) isEqualToString:@"showSelf"]) {
        [anInvocation invokeWithTarget:[SubObject new]];
    }else{
        [super forwardInvocation:anInvocation];
    }
}
```

再次运行工程，程序又被你挽救了一次。

你真的需要救命稻草么？

    通过上面的三根救命稻草，我相信你一定对Objective-C消息机制有了全面而深入的了解，上面的代码也只是为了示例所用，正常情况下，你都不会使用到这些函数(毕竟如果你需要救命稻草，说明你已经落水了)。除非某些特殊需求或者做一些调试框架的开发，否则尽量不要介入消息的发送机制，就像生病就医，发现问题总比逃避治疗要好。顺便说一下，如果你没有使用任何救命稻草，当向某个对象发送了无法处理的消息时，系统会最终调用到NSObject类的doesNotRecognizeSelector方法，这个方法会抛出异常信息，正因如此，你在Xcode的控制台会经常看到如下图所示的crash信息：

![](https://static.oschina.net/uploads/space/2017/0504/114108_Hat9_2340880.png)

你也可以重写这个方法来自定义输出信息，例如：

```objectivec
-(void)doesNotRecognizeSelector:(SEL)aSelector{
    NSLog(@"doesNotRecognizeSelector");
    if ([NSStringFromSelector(aSelector) isEqualToString:@"showSelf"]) {
        NSLog(@"not have a method named showSelf");
        return;
    }
    [super doesNotRecognizeSelector:aSelector];
}
```

下图完整展示了Objective-C整个消息发送与转发机制：

![](https://static.oschina.net/uploads/space/2017/0504/114310_hJpC_2340880.png)

### 三、发送消息的几个函数

#### 1.最重要的两个发送消息函数

    既然Objective-C函数最终的调用都是要转换成消息发送，那么了解下面这些消息发送函数是十分必要的，这些方法都定义在objc/message.h文件中，其中最重要的两个方法是：

```objectivec
//发送消息的函数
/*
self:消息的接收对象
op:方法选择器
...:参数
*/
id objc_msgSend(id self, SEL op, ...);
//发送消息给父类
/*
super:父类对象结构体 
op:方法选择器
...:参数
*/
id objc_msgSendSuper(struct objc_super *super, SEL op, ...);
```

objc\_msgSend函数前面已经有过介绍，objc\_msgSendSuper函数则是从父类中找方法的实现进行执行。需要注意，这个函数非常重要，理解了这个这个函数进行消息发送的原理，你就明白super关键字的某些令人疑惑的行为了。

#### 2.super关键字到底做了什么

    做了这么久的Objective-C开发，你是否真的理解super关键字的含义？你一定会说，这很简单啊，self调用本类的方法，super调用父类的方法。那么我们来看一个小案例：

    在前面创建的命令行工程中新建一个类，使其继承于MyObject类，命名为MyObjectSon，在其中提供两个方法，如下：

MyObjectSon.h文件：

```objectivec
#import "MyObject.h"
@interface MyObjectSon : MyObject
-(void)showClass;
-(void)showSuperClass;
@end
```

MyObjectSon.m文件：

```objectivec
#import "MyObjectSon.h"

@implementation MyObjectSon
-(void)showClass{
    NSLog(@"%@",[self className]);
}
-(void)showSuperClass{
    NSLog(@"%@",[super className]);
}
@end
```

分别调用两个方法，你会惊奇的发现，打印结构都是“MyObjectSon”，super关键字失效了么？非也非也，下面我们来用消息发送机制重新模拟这两个方法的调用。

    首先\[self className\]在调用时会采用前面介绍的消息发送机制先从当前类中找className函数，当前类中并没有提供className函数，所以消息会随着继承链向上传递，找到MyObject类中也没有className函数的实现，会继续向上，最终在NSObject类中找到这个方法，记住，这条消息处理的两个要素是：当前MyObjectSon实例对象作为接收者，NSObject类中的className方法作为调用函数。

    当调用\[super className\]时，首先会使用objc_msgSendSuper方法进行消息的发送，等价于如下代码：

```objectivec
-(void)showSuperClass{
    //创建父类接收对象结构体
    struct objc_super superObj = {self, object_getClass([MyObject new])};
    NSString * name = ((id(*)(struct objc_super*,SEL))objc_msgSendSuper)(&superObj,@selector(className));
    NSLog(@"%@",name);
}
```

objc\_msgSendSuper函数第一个参数为一个父类接收者结构体指针，objc\_super结构体定义如下：

```objectivec
struct objc_super {
    //接收者
    __unsafe_unretained id receiver;
    //接收者类型
    __unsafe_unretained Class super_class;
};
```

在构造objc\_super这个结构体时，receive为接收消息的对象，super\_class为从哪个类中查方法。如此来看一些都清楚了，系统首先从MyObject类中找className方法，没有相应的实现，会继续向上直到找到NSObject类中的className方法，之后进行执行。这条消息处理的两个要素是：当前MyObjectSon实例对象作为接收者，NSObject类中的className方法作为调用函数。这样分析下来，无论是使用self执行的className方法还是使用super执行的className方法，行为实质上是完全一致的！

#### 3.一些辅助的消息发送函数

特殊返回值类型对应不同的发送消息函数：

```objectivec
//返回值为结构体时使用此方法发送消息
void objc_msgSend_stret(id self, SEL op, ...);
void objc_msgSendSuper_stret(struct objc_super *super, SEL op, ...);
//返回值为浮点数时使用此方法发送消息
double objc_msgSend_fpret(id self, SEL op, ...);

```

除了使用SEL方法选择器来发送消息，也可以直接使用Method来发送消息：

```objectivec
//进行函数的调用
/*
receiver:接收者
m:函数
...:参数
*/
id method_invoke(id receiver, Method m, ...);
//返回结构体数据的函数调用
void method_invoke_stret(id receiver, Method m, ...);
```

Method也是一种结构体指针，其定义如下：

```objectivec
struct objc_method {
    //选择器
    SEL method_name                                          OBJC2_UNAVAILABLE;
    //参数类型
    char *method_types                                       OBJC2_UNAVAILABLE;
    //函数地址
    IMP method_imp                                           OBJC2_UNAVAILABLE;
}  
```

示例代码如下：

```objectivec
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        MyObject * obj = [[MyObject alloc]init];
        [obj class];
//为了消除未定义选择器的警告
#pragma clang diagnostic push
#pragma clang diagnostic ignored"-Wundeclared-selector"
        //进行消息发送
        Method method = class_getInstanceMethod([MyObject class], @selector(showSelf:age:));
        ((void(*)(id,Method,NSString *,int))method_invoke)(obj,method,@"珲少",25);
#pragma clang diagnostic pop
        
    }
    return 0;
}
```

下面这些方法可以跳过当前对象，直接进行消息转发：

```objectivec
//跳过当前对象直接进行消息转发机制
id _objc_msgForward(id receiver, SEL sel, ...);
void _objc_msgForward_stret(id receiver, SEL sel, ...);
```

一点建议，上面两个方法都是以下划线开头，这也表明设计者并不想让你直接调用这个方法，确实如此，这两个方法会直接出发对象的消息转发流程，即便当前对象类已经实现了相应的方法也不会进行查找。

###  四、是时候来重温下Runtime了

    所谓运行时是针对于编译时而言的，本篇文章的开头，我们就说过Objective-C是一种极动态的运行时语言。对象的行为是在运行时被决定的，我们前边也了解了有关isa指针即Class的内容，虽然我们并不能直接访问isa指针，但是我们可以通过objc/runtime.h文件中定义的运行时方法来获取或改变类与对象的行为。

#### 1.类相关操作函数

```objectivec
//对OC对象进行内存拷贝 在ARC环境下不可用
/*
obj:要拷贝的对象
size:内存大小
*/
id object_copy(id obj, size_t size);
//进行OC对象内存的释放 在ARC环境下不可用
id object_dispose(id obj);
//获取OC对象的类 注意 这个返回值和isa指针并不是同一个指针
Class object_getClass(id obj); 
//重建对象的类
Class object_setClass(id obj, Class cls); 
//判断一个OC对象是否是类或元类(前面说过类实际上也是对象)
BOOL object_isClass(id obj);
//获取OC对象的类名
const char *object_getClassName(id obj);
//通过类名获取“类”对象
Class objc_getClass(const char *name);
//通过类名获取元类对象
Class objc_getMetaClass(const char *name);
//这个方法也是返回类的定义 只是如果是未注册的 会返回nil
Class objc_lookUpClass(const char *name);
//这个方法也是返回类的定义 只是如果是未注册的 会直接杀死进程
Class objc_getRequiredClass(const char *name);
/*
获取所有已经注册的类 会返回已经注册的类的个数
通常使用如下示例代码：
int numClasses;
        Class * classes = NULL;
        
        numClasses = objc_getClassList(NULL, 0);
        if (numClasses > 0) {
            classes = (Class*)malloc(sizeof(Class) * numClasses);
            numClasses = objc_getClassList(classes, numClasses);
            
            NSLog(@"number of classes: %d", numClasses);
            
            for (int i = 0; i < numClasses; i++) {
                
                Class cls = classes[i];
                NSLog(@"class name: %s", class_getName(cls));
            }
            
            free(classes);
        }
*/
int objc_getClassList(Class *buffer, int bufferCount);
//拷贝所有注册过的类列表 参数为输出类的个数
Class *objc_copyClassList(unsigned int *outCount);
//获取Class类名字符串
const char *class_getName(Class cls);
//判断一个Class是否为元类
BOOL class_isMetaClass(Class cls);
//获取一个类的父类
Class class_getSuperclass(Class cls);
//修改一个类的父类
Class class_setSuperclass(Class cls, Class newSuper);
//获取一个类的版本
int class_getVersion(Class cls);
//设置一个类的版本
void class_setVersion(Class cls, int version);
//获取类的内存布局
size_t class_getInstanceSize(Class cls);
```

上面列举的方法都和类相关，你没看错，通过object_setClass()动态改变对象所属的类，但是需要注意，对象的成员变量并不会受到影响，方法则全部替换为新类的方法。如果你喜欢，你甚至可以运行时动态修改类的父类，这十分酷吧。下面这些方法则与类中的变量有关：

#### 2.变量属性相关操作函数

```objectivec
//获取类额外分配内存的指针
void *object_getIndexedIvars(id obj);
//根据变量名获取实例变量指针
/*
Ivar实质上是一个结构体指针，存放变量信息，如下：
struct objc_ivar {
    //变量名
    char *ivar_name                                          OBJC2_UNAVAILABLE;
    //变量类型
    char *ivar_type                                          OBJC2_UNAVAILABLE;
    int ivar_offset                                          OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
}   
*/
Ivar class_getInstanceVariable(Class cls, const char *name);
//根据变量名获取类变量指针
Ivar class_getClassVariable(Class cls, const char *name);
//获取所有实例变量指针 outCount输出实例变量个数
Ivar *class_copyIvarList(Class cls, unsigned int *outCount);
//通过变量指针获取具体变量的值
id object_getIvar(id obj, Ivar ivar);
//通过变量指针设置具体变量的值
void object_setIvar(id obj, Ivar ivar, id value);
//通过变量指针设置变量的值 并进行强引用
void object_setIvarWithStrongDefault(id obj, Ivar ivar, id value);
//直接通过变量名修改实例变量的值 会返回变量指针
Ivar object_setInstanceVariable(id obj, const char *name, void *value);
//用法同上
Ivar object_setInstanceVariableWithStrongDefault(id obj, const char *name, void *value);
//通过变量名直接获取示例变量的值
Ivar object_getInstanceVariable(id obj, const char *name, void **outValue);
//获取类变量内存布局
const uint8_t *class_getIvarLayout(Class cls);
//设置类变量内存布局
void class_setIvarLayout(Class cls, const uint8_t *layout);
//获取类变量布局 弱引用
const uint8_t *class_getWeakIvarLayout(Class cls);
//同上
void class_setWeakIvarLayout(Class cls, const uint8_t *layout);
//通过属性名获取属性
/*
属性特指使用@prototype定义的
objc_property_t是一个结构体指针，其描述的是属性的信息，如下：
typedef struct {
    const char *name;           /**属性名 */
    const char *value;          /**属性值 */
} objc_property_attribute_t;

*/
objc_property_t class_getProperty(Class cls, const char *name);
//获取所有属性列表
objc_property_t *class_copyPropertyList(Class cls, unsigned int *outCount);
//向类添加一个实例变量
/*
需要注意，已经注册存在的类是不能通过这个方法追加实例变量的
这个方法只能在objc_allocateClassPair函数执行后并且objc_registerClassPair执行前进行调用
即这个函数是用来动态生成类的
*/
BOOL class_addIvar(Class cls, const char *name, size_t size, 
                               uint8_t alignment, const char *types);
//向类中添加属性
BOOL class_addProperty(Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount);
//进行类属性的替换
void class_replaceProperty(Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount);
//获取变量指针对应的变量名
const char *ivar_getName(Ivar v);
//获取编码后的变量类型
const char *ivar_getTypeEncoding(Ivar v);
//获取属性名
const char *property_getName(objc_property_t property);
//获取属性attribute
const char *property_getAttributes(objc_property_t property);
char *property_copyAttributeValue(objc_property_t property, const char *attributeName);
//获取属性attribute列表
objc_property_attribute_t *property_copyAttributeList(objc_property_t property, unsigned int *outCount);
//进行属性关联 这种方式可以为已经存在的类的实例扩展属性
/*
object:要添加属性的对象
key:关联的键
value:添加的属性的值
policy:添加属性的策略
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    //assign
    OBJC_ASSOCIATION_ASSIGN = 0,           /**< Specifies a weak reference to the associated object. */
    //retain nonatimic
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, /**< Specifies a strong reference to the associated object. 
                                            *   The association is not made atomically. */
    //copy nonatomic
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,   /**< Specifies that the associated object is copied. 
                                            *   The association is not made atomically. */
    //retain
    OBJC_ASSOCIATION_RETAIN = 01401,       /**< Specifies a strong reference to the associated object.
                                            *   The association is made atomically. */
    //copy
    OBJC_ASSOCIATION_COPY = 01403          /**< Specifies that the associated object is copied.
                                            *   The association is made atomically. */
};
*/
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy);
//获取关联属性的值
id objc_getAssociatedObject(id object, const void *key);
//移除一个关联的属性
void objc_removeAssociatedObjects(id object);
```

#### 3.方法操作相关函数

```objectivec
//通过选择器获取某个类的实例方法
Method class_getInstanceMethod(Class cls, SEL name);
//通过选择器定义某个类的类方法
Method class_getClassMethod(Class cls, SEL name);
//通过选择器获取某个类的方法函数指针
IMP class_getMethodImplementation(Class cls, SEL name);
//同上
IMP class_getMethodImplementation_stret(Class cls, SEL name);
//判断某个类是否可以相应选择器
BOOL class_respondsToSelector(Class cls, SEL sel);
//获取某个类的实例方法列表
Method *class_copyMethodList(Class cls, unsigned int *outCount);
//为某个类动态添加一个实例方法
/*
cls:添加方法的类
SEL：添加的方法选择器
IMP:方法实现
types:参数类型字符串
*/
BOOL class_addMethod(Class cls, SEL name, IMP imp, const char *types); 
//替换一个方法的实现
/*
cls:类
SEL:要替换实现的选择器
IMP:实现
types:参数类型字符串
*/
IMP class_replaceMethod(Class cls, SEL name, IMP imp, const char *types);
//获取函数的选择器
SEL method_getName(Method m);
//获取函数的实现
IMP method_getImplementation(Method m);
//获取函数的参数类型
const char *method_getTypeEncoding(Method m);
//获取函数的参数个数
unsigned int method_getNumberOfArguments(Method m);
//获取函数的返回值类型
char *method_copyReturnType(Method m);
//拷贝参数类型列表
char *method_copyArgumentType(Method m, unsigned int index);
//获取返回值类型
void method_getReturnType(Method m, char *dst, size_t dst_len) ;
//获取参数类型
void method_getArgumentType(Method m, unsigned int index, char *dst, size_t dst_len);
//获取函数描述信息
/*
objc_method_description结构体描述函数信息 如下：
struct objc_method_description {
    //函数名
    SEL name;               /**< The name of the method */
    //参数类型
    char *types;            /**< The types of the method arguments */
};

*/
struct objc_method_description *method_getDescription(Method m);
//修改某个函数的实现
IMP method_setImplementation(Method m, IMP imp);
//交换两个函数的实现
void method_exchangeImplementations(Method m1, Method m2);
//获取选择器名称
const char *sel_getName(SEL sel);
//通过名称获取选择器
SEL sel_getUid(const char *str);
//注册一个选择器
SEL sel_registerName(const char *str);
//判断两个选择器是否相等
BOOL sel_isEqual(SEL lhs, SEL rhs);
//将block作为IMP的实现
IMP imp_implementationWithBlock(id block);
//获取IMP的实现block
id imp_getBlock(IMP anImp);
//删除IMP的block实现
BOOL imp_removeBlock(IMP anImp);
```

上面列举的函数中很多都用到参数类型的指定，types需要设置为C风格的字符数组，即C字符串，其中第1个字符表示返回值类型，其余字符依次表示参数类型，参数类型与字符的映射表如下：

![](https://static.oschina.net/uploads/space/2017/0505/103650_6rxS_2340880.png)

#### 4.协议相关操作函数

```objectivec
//判断某个类是否遵守某个协议
BOOL class_conformsToProtocol(Class cls, Protocol *protocol);
//拷贝某个类的协议列表
Protocol * __unsafe_unretained *class_copyProtocolList(Class cls, unsigned int *outCount);
//动态向类中添加协议
BOOL class_addProtocol(Class cls, Protocol *protocol);
//通过协议名获取某个协议指针
Protocol *objc_getProtocol(const char *name);
//拷贝所有协议列表
Protocol * __unsafe_unretained *objc_copyProtocolList(unsigned int *outCount);
//判断某个协议是否继承于另一个协议
BOOL protocol_conformsToProtocol(Protocol *proto, Protocol *other);
//判断两个协议是否相同
BOOL protocol_isEqual(Protocol *proto, Protocol *other);
//获取协议名
const char *protocol_getName(Protocol *p);
//获取协议中某个函数的描述
/*
p:协议指针
aSel:方法选择器
isRequiredMethod:是否是必实现的
isInstanceMehod:是否是实例方法
*/
struct objc_method_description protocol_getMethodDescription(Protocol *p, SEL aSel, BOOL isRequiredMethod, BOOL isInstanceMethod);
//获取协议方法描述列表
struct objc_method_description *protocol_copyMethodDescriptionList(Protocol *p, BOOL isRequiredMethod, BOOL isInstanceMethod, unsigned int *outCount);
//获取协议中的属性描述
objc_property_t protocol_getProperty(Protocol *proto, const char *name, BOOL isRequiredProperty, BOOL isInstanceProperty);
//获取协议中的属性描述列表
objc_property_t *protocol_copyPropertyList(Protocol *proto, unsigned int *outCount);
//同上
objc_property_t *protocol_copyPropertyList2(Protocol *proto, unsigned int *outCount, BOOL isRequiredProperty, BOOL isInstanceProperty);
//获取适配协议列表
Protocol * __unsafe_unretained *protocol_copyProtocolList(Protocol *proto, unsigned int *outCount);
//创建一个协议
Protocol *objc_allocateProtocol(const char *name);
//进行协议注册
void objc_registerProtocol(Protocol *proto);
//向协议中添加一个方法描述
void protocol_addMethodDescription(Protocol *proto, SEL name, const char *types, BOOL isRequiredMethod, BOOL isInstanceMethod);
//向协议中添加另一个协议
void protocol_addProtocol(Protocol *proto, Protocol *addition);
//向协议中添加属性描述
void protocol_addProperty(Protocol *proto, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount, BOOL isRequiredProperty, BOOL isInstanceProperty);
```

协议实质也是一个Objective-C对象。

#### 5.动态构建类实例相关函数

```objectivec
//动态创建一个类实例
/*
cls:类名
size_t:分配额外的内存 用来存放类定义之外的变量
*/
id class_createInstance(Class cls, size_t extraBytes);
//同上
id objc_constructInstance(Class cls, void *bytes);
//销毁一个实例
void *objc_destructInstance(id obj);
//动态定义一个类 
/*
superClass:指定父类
name:类名
extraBytes:额外的内存空间
*/
Class objc_allocateClassPair(Class superclass, const char *name,size_t extraBytes);
//进行类的注册 需要注意 要添加属性 必须在注册类之前添加
void objc_registerClassPair(Class cls);
//销毁一个类
void objc_disposeClassPair(Class cls);
```

### 五、运行时的几点应用扩展

    到此本篇文章终于要告一段落了，相信你如果能看到这里，你一定有超凡的耐心。但是切记Objective-C的消息机制配合运行时是可以给开发者极大的元编程自由，但是不适当的使用也会造成破坏性的后果。下面几篇博客从一些方面介绍了Runtime的几点应用，你可以从中管中窥豹，可见一斑。

1.runtime基础应用：[https://my.oschina.net/u/2340880/blog/489072](https://my.oschina.net/u/2340880/blog/489072)

2.使用runtime全局修改UILabel字体：[https://my.oschina.net/u/2340880/blog/538356](https://my.oschina.net/u/2340880/blog/538356)

3.使用runtime自动化归档：[https://my.oschina.net/u/2340880/blog/514330](https://my.oschina.net/u/2340880/blog/514330)

4.代码调试框架的设计：[https://my.oschina.net/u/2340880/blog/504675](https://my.oschina.net/u/2340880/blog/504675)

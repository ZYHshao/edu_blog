---
title: Objective-C关于id引发的一些思考
date: 2017-10-14
categories: Objective-C浅探
tags: []
---
## Objective-C关于id引发的一些思考

    Objective-C是面向对象语言，但其中又并非全部是对象。在初学这门语言时，我常常从意识上将NS开头的类型与C语言原本的那些类型分割开来，假装他们之间没有联系，只关注“类”的世界。然而类终究只是一种应用上的抽象，就像“语法糖”一样，抛开华丽的外表，内部依然是最朴素的结构体和指针。本篇博客的来由源自朋友的一个问题：在ARC环境，performSelector:withObject:方法如何传递非对象类型的数据呢？这个问题乍看起来简单，但要较较真，却也并非那么简单。下面的内容都是有这个简单的问题引出的，如果你感兴趣，在读之前可以先试着解决下上面的疑问。

### 一、还要先说id

    id是Objective-C中定义的一种泛型实现，它可以表示任何对象类型。尽管id看起来是如此简单，但细细琢磨，其却包含了3层意义：

1.作为参数或返回值

    将id类型作为函数的参数或返回值是最浅的一层意义，其增加了函数的灵活性，Foundation框架中也有其大量的应用，例如可变数组追加元素。

2.id类型的参数不会进行类型检查

    这是id类型十分重要的一个特点，声明为id类型的对象就相当于告诉了编译器不进行类型检查(这也是和NSObject类型最大区别)。因此，你可以将id类型的变量赋值给任何对象类型，也可以将任何对象类型的变量赋值给id类型，更重要的是，使用id类型的对象可以调用任意方法，都不会进行类型检查。

3.id<protocol>是一种优雅的编程方式

    由于id类型不会进行编译检查，要约束类型方法实现最好的方式就是通过协议，id<protocol>是一种十分优雅的编程方式，其不再关心类型，只注重约定的实现，Foundation框架中的代理多采用这种方式设计。

    另外，在objc.h文件中，id被定位为如下：

```objectivec
struct objc_object {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
};
//结构体指针
typedef struct objc_object *id;
```

### 二、关于void与void*

    在开发中，void用的最多的地方要数标记Objective-C无返回值的函数，Objective-C函数和C函数不同，其必须有一个确定的返回值类型，如果没有返回值，则需要使用void来标记返回值类型，而C函数是可以不指定返回值类型的，默认的C函数则是返回int类型的值，例如下面的两个函数实际上是完全一样的：

```objectivec
print(){
    printf("cccccc");
    return 0;
}

int print(){
    printf("cccccc");
    return 0;
}
```

void在C语言中还有一大用途在于约束无参函数，例如上面示例的函数虽然没有参数，但是如果你在调用的时候强制传入参数编译器也不出进行错误提醒，如果将函数修改如下，则此函数就完全不能传入参数了：

```objectivec
int print(void){
    printf("cccccc");
    return 0;
}
```

    归根结底，void大多时候用来表示“空”，而void * 则完全不同，它所描述的实际上是任意类型的指针。这里和id很像对不对，虽然id描述的是Objective-C对象但是本质也是指针，那么根据我们的推测，id类型的数据和void*类型的数据是可以进行类型转换的。事实上，在MRC环境下确实如此，ARC环境下则要更复杂一些，由于ARC机制要对Objective-C对象进行引用计数管理，对C指针并不会，因此在ARC环境下编译器是不允许我们直接将id于void*进行进行转换的，例如下面的报错：

![](https://static.oschina.net/uploads/space/2017/1014/174458_SNKM_2340880.png)

### 三、ARC中用__bridge的应用

    前面说过，由于ARC的原因，导致无法在Objective-C对象与C指针类型之间进行直接转换，但是可以通过\_\_bridge来转换，从字面理解，\_\_bridge的作用就是用来桥接。在做Objective-C相关开发时，你一定遇到过CoreFoundation框架与Foundation框架混用的情况，CF框架中的类都是由C语言直接实现的，例如CFString，CFURL等，其虽然可以和NSString，NSURL互用，但在ARC环境在，却必须进行桥接转换，即使用__bridge。上面的代码可以做如下修改：

```objectivec
    int a = 10;
    void * ap = &a;
    id c = (__bridge id)ap;
```

同样，将id类型转换为void *如下：

```objectivec
    NSNumber * num = [[NSNumber alloc]initWithInt:1];
    void * cNumber = (__bridge void *)num;
```

与\_\_bridge相对应的还有\_\_bridge\_transfer与\_\_bridge\_retained，他们的区别是\_\_bridge不会改变对象的所有权，\_\_bridge\_transfer会将对象所有权进行转移，即release掉转换前的Objective-C对象，而\_\_bridge\_retained则是将所有权进行retain强引用。

### 四、解决最初的问题

    再来看我们最初的问题，下面方法的两个参数都是id类型的：

```objectivec
- (id)performSelector:(SEL)aSelector withObject:(id)object1 withObject:(id)object2;
```

虽然我们也可以用其他方式来达到相同的效果，比如修改原函数的参数类型，或者使用NSInvocation来发送消息，一种更简便的方式如下：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    int a = 10;
    [self performSelector:@selector(log:age:) withObject:@"huishao" withObject:(__bridge id)(void*)a];
}

-(void)log:(NSString *)name age:(int)age{
    NSLog(@"%@,%d",name,age);
}
```

最后，总结一点，其实不论任何编程语言，类型检查都是编译时的特性，真正传递的数据依然是在运行时决定的。

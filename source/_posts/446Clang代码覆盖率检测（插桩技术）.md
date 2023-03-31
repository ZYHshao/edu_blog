---
title: Clang代码覆盖率检测（插桩技术）
date: 2022-06-19
categories: 编程珠玑
tags: []
---
# Clang代码覆盖率检测（插桩技术）

Clang的全称是C Language Family Frontend for LLVM，即基于LLVM的C系列语言的前端编译器。iOS应用的前端编译，即是采用Clang完成的。本篇文章，我们主要介绍Clang内置的一个简单的代码覆盖率检测功能，对于iOS开发来说，此功能更多用于Objective-C的方法插桩，为二进制重排提供支持，优化应用启动速度。但代码覆盖率检测功能并不仅仅只能应用与二进制重排，其本质是对于函数级、基本块级或代码边缘级插入回调，我们可以基于这一原理更灵活的实现所需要的功能。

## 1\. Tracing PCs with guards

开启Clang代码覆盖率检查功能，需要配置-fsanitize-coverage编译参数，你可以创建一个iOS模板工程做测试，在Build Settings->Apple Clang - Custom Complier Flags->Other C Flags下面配置。如图：

![](https://oscimg.oschina.net/oscnet/up-704be2272e99bcdcdc523eb6511dc385ac4.png)

trace-pc-guard模式下，所有代码块首部都会被插入如下回调函数：

void \_\_sanitizer\_cov\_trace\_pc\_guard(uint32\_t *guard)

此回调函数是需要开发者自定义的，除此之外，还需要实现对应初始化的回调函数：

void \_\_sanitizer\_cov\_trace\_pc\_guard\_init(uint32\_t \*start, uint32\_t \*stop)

在示例工程的main.m文件中定义这两个回调如下：

```objectivec
void __sanitizer_cov_trace_pc_guard(uint32_t *guard) {
    void *PC = __builtin_return_address(0);
    Dl_info info;
    dladdr(PC, &info);
    printf("%s \n",info.dli_sname);
}

void __sanitizer_cov_trace_pc_guard_init(uint32_t *start, uint32_t *stop) {
    static uint64_t N;
    if (start == stop || *start) return;
    for (uint32_t *x = start; x < stop; x++) {
        *x = ++N;
    }
    printf("INIT Count: %llu \n", N);
}

```

其中，\_\_sanitizer\_cov\_trace\_pc\_guard\_init为初始化回调，通过其中参数可以获取到符号个数，\_\_sanitizer\_cov\_trace\_pc_guard是插桩函数，每个代码块开始调用时，都会首先调用此插桩函数。

直接运行代码，控制台输出如下：

```
INIT Count: 14 
main 
-[AppDelegate application:didFinishLaunchingWithOptions:] 
-[SceneDelegate window] 
-[SceneDelegate setWindow:] 
-[SceneDelegate window] 
-[SceneDelegate window] 
-[SceneDelegate scene:willConnectToSession:options:] 
-[SceneDelegate window] 
-[SceneDelegate window] 
-[SceneDelegate window] 
-[ViewController viewDidLoad] 
-[SceneDelegate sceneWillEnterForeground:] 
-[SceneDelegate sceneDidBecomeActive:] 


```

可以看到，输出的结果就是按照项目中方法的调用顺序排序的。你可能看到有许多重复的符号，这是由于trace-pc-guard设定的，其会对源码中任意的代码块开始执行时进行插桩函数回调，包括if判断，while循环以及Block调用等，例如你可以尝试在ViewController.m文件的viewDidLoad方法中添加一些代码，如下：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    printf("开始Block==================\n");
    void(^block)(void) = ^{

    };
    block();
    printf("开始循环==================\n");
    int n = 3;
    while (n > 0) {
        n--;
    }
    printf("开始分支判断==================\n");
    if (n < 10) {
        n++;
    }
}

```

运行项目，输出效果如下：

```
INIT Count: 18 
main 
-[AppDelegate application:didFinishLaunchingWithOptions:] 
-[SceneDelegate window] 
-[SceneDelegate setWindow:] 
-[SceneDelegate window] 
-[SceneDelegate window] 
-[SceneDelegate scene:willConnectToSession:options:] 
-[SceneDelegate window] 
-[SceneDelegate window] 
-[SceneDelegate window] 
-[ViewController viewDidLoad] 
开始Block==================
__29-[ViewController viewDidLoad]_block_invoke 
开始循环==================
-[ViewController viewDidLoad] 
-[ViewController viewDidLoad] 
-[ViewController viewDidLoad] 
开始分支判断==================
-[ViewController viewDidLoad] 
-[SceneDelegate sceneWillEnterForeground:] 
-[SceneDelegate sceneDidBecomeActive:] 

```

有时候并非所有的代码块都需要插桩，例如做二进制重排时，只需要方法和函数的插桩，也有配置方式，我们后面介绍。

## 2\. Inline 8bit-counters

此模式需要配置成：

```
-fsanitize-coverage=inline-8bit-counters

```

此模式与trace-pc-guard类似，只是其在代码块开始时不会进行回调，而是简单的增加内置计数器的计数。同样，在此模式下，用户需要实现如下自定义函数：

```
void __sanitizer_cov_8bit_counters_init(char *start, char *end) {
  // [start,end) is the array of 8-bit counters created for the current DSO.
  // Capture this array in order to read/modify the counters.
}

```

此函数对应计数器的初始化。

## 3\. Inline bool-flag

此模式与inline-8bit-counters模式类似，需要配置成：

```
-fsanitize-coverage=inline-bool-flag

```

在此模式下，在代码块开始时会将一个内置的布尔值置为true，而不是增加计数器的计数。需要实现如下函数来捕获此变量：

```objectivec
void __sanitizer_cov_bool_flag_init(bool *start, bool *end) {
  // [start,end) is the array of boolean flags created for the current DSO.
  // Capture this array in order to read/modify the flags.
}

```

## 4\. Tracing PCs

此模式在代码块的开始出会回调\_\_sanitizer\_cov\_trace\_pc() 函数，也是插桩回调，此模式可配置为：

```
-fsanitize-coverage=trace-pc

```

对应实现自定义的插桩函数如下：

```objectivec
void __sanitizer_cov_trace_pc(void*a) {
    void *PC = __builtin_return_address(0);
    Dl_info info;
    dladdr(PC, &info);
    printf("%s %p \n",info.dli_sname, info.dli_saddr);
    printf("__sanitizer_cov_trace_pc:%p\n",a);
}

```

对于此模式，我们可以配置一个额外的参数来区别间接调用，例如修改ViewController.m文件中的代码如下：

```objectivec
#import "ViewController.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    printf("开始Block==================\n");
    void(^block)(void) = ^{

    };
    block();
    printf("开始循环==================\n");
    int n = 3;
    while (n > 0) {
        n--;
    }
    printf("开始分支判断==================\n");
    if (n < 10) {
        n++;
    }
    [self log];
}

- (void)log {
    
}

@end

```

新定义了一个log函数，并在ViewDidLoad中进行了调用，配置编译选项如下：

```
-fsanitize-coverage=trace-pc,indirect-calls

```

对应实现间接调用的插桩回调如下：

```
void __sanitizer_cov_trace_pc_indir(void *callee) {
    printf("__sanitizer_cov_trace_pc_indirect:%p\n",callee);
}

```

运行代码，控制台输出如下：

```
main 0x105f5dee0 
__sanitizer_cov_trace_pc:0x1
-[AppDelegate application:didFinishLaunchingWithOptions:] 0x105f5dae0 
__sanitizer_cov_trace_pc:0x6000019241f0
-[SceneDelegate window] 0x105f5e200 
__sanitizer_cov_trace_pc:0x600001b1ffc0
-[SceneDelegate setWindow:] 0x105f5e240 
__sanitizer_cov_trace_pc:0x600001b1ffc0
-[SceneDelegate window] 0x105f5e200 
__sanitizer_cov_trace_pc:0x600001b1ffc0
-[SceneDelegate window] 0x105f5e200 
__sanitizer_cov_trace_pc:0x600001b1ffc0
-[SceneDelegate scene:willConnectToSession:options:] 0x105f5df80 
__sanitizer_cov_trace_pc:0x600001b1ffc0
-[SceneDelegate window] 0x105f5e200 
__sanitizer_cov_trace_pc:0x600001b1ffc0
-[SceneDelegate window] 0x105f5e200 
__sanitizer_cov_trace_pc:0x600001b1ffc0
-[SceneDelegate window] 0x105f5e200 
__sanitizer_cov_trace_pc:0x600001b1ffc0
-[ViewController viewDidLoad] 0x105f5d940 
__sanitizer_cov_trace_pc:0x7fec54f08490
-[ViewController viewDidLoad] 0x105ffa2c8 
__sanitizer_cov_trace_pc_indirect:0x7fff20183600
开始Block==================
-[ViewController viewDidLoad] 0x105ffa2c8 
__sanitizer_cov_trace_pc_indirect:0x105f5da80
__29-[ViewController viewDidLoad]_block_invoke 0x105f5da80 
__sanitizer_cov_trace_pc:0x105f60040
开始循环==================
-[ViewController viewDidLoad] 0x105f5d940 
__sanitizer_cov_trace_pc:0x7fff864ab328
-[ViewController viewDidLoad] 0x105f5d940 
__sanitizer_cov_trace_pc:0x7fff864ab328
-[ViewController viewDidLoad] 0x105f5d940 
__sanitizer_cov_trace_pc:0x7fff864ab328
开始分支判断==================
-[ViewController viewDidLoad] 0x105f5d940 
__sanitizer_cov_trace_pc:0x7fff864ab328
-[ViewController viewDidLoad] 0x105ffa2c8 
__sanitizer_cov_trace_pc_indirect:0x7fff201833c0
-[ViewController log] 0x105f5dab0 
__sanitizer_cov_trace_pc:0x7fec54f08490
-[SceneDelegate sceneWillEnterForeground:] 0x105f5e140 
__sanitizer_cov_trace_pc:0x600001b1ffc0
-[SceneDelegate sceneDidBecomeActive:] 0x105f5e080 
__sanitizer_cov_trace_pc:0x600001b1ffc0

```

## 5\. 不同级别的检测

前面我们介绍的编译模式，会对函数，Block和逻辑代码块进行检测，有时候我们不需要这个细粒度的检测，例如在二进制重排时，我们仅仅想检测方法和函数，只想对方法函数进行插桩，此时就可以配置检测级别参数，支持的级别参数有三种：

1\. edge：默认的级别，细粒度最高的级别，函数，Block和代码块都会被插桩。

2\. bb：基础的块级代码会被插桩。

3\. func：仅仅函数块会被插桩。

通常我们在做二进制重排时，更关注的是函数的调用顺序，使用func等级即可，编译设置如下：

```
-fsanitize-coverage=trace-pc,func

```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> —— 珲少 QQ：316045346
> 
> 同时，如果本篇文章让你觉得有用，欢迎分享给更多朋友，请标明出处。
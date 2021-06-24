---
title: Objective-C中编写省略参数的多参函数
date: 2015-05-04
categories: iOS逻辑初窥
tags: []
---
## Objective-C中编写省略参数的多参数函数

### 引语：

在Object-C中，我们会遇到很多像NSLog这样的函数，其中参数的个数不确定，由程序员自由控制，在初始化数组，字典等方面应用广泛，那么，这类的函数是如何实现的呢？我们怎么编写我们自己的省略参数的函数呢？当然，这不是唯一的多参函数的处理方法，你也可以通过一个字典或者数组传递参数。但C为我们提供的这样的一种机制，无疑是最方便的。

### 一、了解几个概念

va_list

C语言中定义的一个指针，用于指向当前的参数。

va_start(ap,param)

这个宏是初始化参数列表，其中第一个参数是va_list对象，第二个参数是参数列表的第一个参数。

va_arg(ap, type)

一个用于取出参数的宏，这个宏的第一个参数是va_list对象，第二个参数是要取出的参数类型。

va_end(ap)

这个宏用于关闭取参列表

### 二、多参函数的取参原理

在编写我们自己的多参函数之前，明白函数的取参原理是十分重要的，首先，函数的参数是被放入我们内存的栈段的，而且放入的顺序是从后往前放入，比如如果一个函数参数如下：

void func(int a,int b,int c,int d)

那么传递参数的时候参数d先入栈，接着是c、b、a。如此这样，在取参的时候，根据堆栈的取值原则，则取值顺序为a、b、c、d。所以在原理上，只要我们知道第一个参数的地址和每个参数的类型，我们就可以将参数都取出来。而上面介绍的几个宏，就是帮助我们做这些的。  
 

### 三、声明与实现省略参数的多参函数

"..."这个符号就是我们用来实现省略参数函数的符号。例如我们模拟实现一个log函数如下：

```
-(void)myLog:(NSString *)str,...{//省略参数的写法
    va_list list;//创建一个列表指针对象
    va_start(list, str);//进行列表的初始化，str为省略前的第一个参数，及...之前的那个参数
    NSString * temStr = str;
    while (temStr!=nil) {//如果不是nil，则继续取值
         NSLog(@"%@",temStr);
         temStr = va_arg(list, NSString*);//返回取到的值，并且让指针指向下一个参数的地址
    }
    va_end(list);//关闭列表指针
}
```

注意，调用时，我们必须在参数的最后加上nil这个判断结束的条件：

```
[self myLog:@"312",@"321", nil];//必须有nil
```

### 四、一点补充

细心的你可能发现了，这里的nil是我们在调用函数时手动加上的，可是系统的许多函数在我们调用时，系统直接帮我们加上了参数结尾的那个nil，例如

 NSArray \* array = \[NSArray arrayWithObjects:(id), nil\]

这是如何做到的呢？我们只需要在函数的声明里加上一个宏，就可以实现这个功能，修改如下：

```
-(void)myLog:(NSString *)str,...NS_REQUIRES_NIL_TERMINATION{//这里加上一个宏
    va_list list;
    va_start(list, str);
    NSString * temStr = str;
    while (temStr!=nil) {
         NSLog(@"%@",temStr);
         temStr = va_arg(list, NSString*);
    }
    va_end(list);
}
```

顾名思义，这个宏的作用就是在结束位置加上我们需要的nil。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

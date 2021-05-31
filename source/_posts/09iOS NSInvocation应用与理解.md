---
title: iOS NSInvocation应用与理解
date: 2015-04-10   
categories: iOS编程技巧
tags: [NSInvocation,iOS编程]              
---
IOS中有一个类型是SEL，它的作用很相似与函数指针，通过performSelector:withObject:函数可以直接调用这个消息。但是perform相关的这些函数，有一个局限性，其参数数量不能超过2个，否则要做很麻烦的处理，与之相对，NSInvocation也是一种消息调用的方法，并且它的参数没有限制。这两种直接调用对象消息的方法，在IOS4.0之后，大多被block结构所取代，只有在很老的兼容性系统中才会使用，简单用法总结如下：

一、初始化与调用

在官方文档中有明确说明，NSInvocation对象只能使用其类方法来初始化，不可使用alloc/init方法。它执行调用之前，需要设置两个方法：setSelector: 和setArgument:atIndex：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    SEL myMethod = @selector(myLog);
    //创建一个函数签名，这个签名可以是任意的,但需要注意，签名函数的参数数量要和调用的一致。
    NSMethodSignature * sig  = [NSNumber instanceMethodSignatureForSelector:@selector(init)];
    //通过签名初始化
    NSInvocation * invocatin = [NSInvocation invocationWithMethodSignature:sig];
    //设置target
    [invocatin setTarget:self];
    //设置selecteor
    [invocatin setSelector:myMethod];
    //消息调用
    [invocatin invoke];
    
}
-(void)myLog{
    NSLog(@"MyLog");
}
```

注意：签名函数的参数数量要和调用函数的一致。测试后发现，当签名函数参数数量大于被调函数时，也是没有问题的。

调用多参数的方法，我们可以这样写：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    SEL myMethod = @selector(myLog:parm:parm:);
    NSMethodSignature * sig  = [[self class] instanceMethodSignatureForSelector:myMethod];
    NSInvocation * invocatin = [NSInvocation invocationWithMethodSignature:sig];
    [invocatin setTarget:self];
    [invocatin setSelector:myMethod2];
    int a=1;
    int b=2;
    int c=3;
    [invocatin setArgument:&a atIndex:2];
    [invocatin setArgument:&b atIndex:3];
    [invocatin setArgument:&c atIndex:4];
    [invocatin invoke];
}
-(void)myLog:(int)a parm:(int)b parm:(int)c{
    NSLog(@"MyLog%d:%d:%d",a,b,c);
}
```

注意：1、这里设置参数的Index 需要从2开始，因为前两个被selector和target占用。下面这样写也没有任何问题：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    SEL myMethod = @selector(myLog:parm:parm:);
    SEL myMethod2 = @selector(myLog);
    NSMethodSignature * sig  = [[self class] instanceMethodSignatureForSelector:myMethod];
    NSInvocation * invocatin = [NSInvocation invocationWithMethodSignature:sig];
    ViewController * view = self;
    [invocatin setArgument:&view atIndex:0];
    [invocatin setArgument:&myMethod2 atIndex:1];
    int a=1;
    int b=2;
    int c=3;
    [invocatin setArgument:&a atIndex:2];
    [invocatin setArgument:&b atIndex:3];
    [invocatin setArgument:&c atIndex:4];
    [invocatin retainArguments];
    [invocatin invoke];
}
-(void)myLog:(int)a parm:(int)b parm:(int)c{
    NSLog(@"MyLog%d:%d:%d",a,b,c);
}
```

2、这里的传参方式必须是传递参数地址。

二、NSInvocation的返回值

NSInvocation对象，是可以有返回值的，然而这个返回值，并不是其所调用函数的返回值，需要我们手动设置：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    SEL myMethod = @selector(myLog:parm:parm:);
    NSMethodSignature * sig  = [[self class] instanceMethodSignatureForSelector:myMethod];
    NSInvocation * invocatin = [NSInvocation invocationWithMethodSignature:sig];
    [invocatin setTarget:self];
    [invocatin setSelector:myMethod2];
    ViewController * view = self; 
    int a=1;
    int b=2;
    int c=3;
    [invocatin setArgument:&view atIndex:0];
    [invocatin setArgument:&myMethod2 atIndex:1];
    [invocatin setArgument:&a atIndex:2];
    [invocatin setArgument:&b atIndex:3];
    [invocatin setArgument:&c atIndex:4];
    [invocatin retainArguments];
    //我们将c的值设置为返回值
    [invocatin setReturnValue:&c];
    int d;
    //取这个返回值
    [invocatin getReturnValue:&d];
    NSLog(@"%d",d);
    
}
-(int)myLog:(int)a parm:(int)b parm:(int)c{
    NSLog(@"MyLog%d:%d:%d",a,b,c);
    return a+b+c;
}
```

注意：这里的操作传递的都是地址。如果是OC对象，也是取地址。

三、关于内存

可以注意到\- (void)retainArguments;这个方法，它会将传入的所有参数以及target都retain一遍。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

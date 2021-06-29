---
title: iOS多线程编程之二——NSOperation与NSOperationQueue
date: 2015-05-19
categories: iOS逻辑初窥
tags: []
---
## iOS多线程编程之二——NSOperation与NSOperationQueue

### 一、NSOperation解析

NSOperation是基于Objective-C封装的一套管理与执行线程操作的类。这个类是一个抽象类，通常情况下，我们会使用NSInvocationOperation和NSBlockOperation这两个子类进行多线程的开发，当然我们也可以写继承于NSOperation的类，封装我们自己的操作类。

#### 1、NSOperation抽象类中提供的逻辑方法

操作开始执行

```
- (void)start;
```

在子类中可以重写这个方法，实现执行的方法

```
- (void)main;
```

取消执行

```
- (void)cancel;
```

获取当操作状态的几个属性

```
@property (readonly, getter=isCancelled) BOOL cancelled;//当前操作是否取消执行
@property (readonly, getter=isExecuting) BOOL executing;//当前操作是否正在执行
@property (readonly, getter=isFinished) BOOL finished;//当前操作是否执行结束
@property (readonly, getter=isAsynchronous) BOOL asynchronous;//当前操作是否在异步线程中
@property (readonly, getter=isReady) BOOL ready;//当前操作是否已经准备好
```

阻塞当前线程直到操作完成

```
- (void)waitUntilFinished;
```

设置在操作队列中的优先级

```
@property NSOperationQueuePriority queuePriority;
```

其中NSOperationQueuePriority的枚举如下：

```
typedef NS_ENUM(NSInteger, NSOperationQueuePriority) {
    NSOperationQueuePriorityVeryLow = -8L,//优先级很低
    NSOperationQueuePriorityLow = -4L,//优先级低
    NSOperationQueuePriorityNormal = 0,//优先级普通
    NSOperationQueuePriorityHigh = 4,//优先级高
    NSOperationQueuePriorityVeryHigh = 8//优先级非常高
};
```

  
设置操作完成后的回调block

```
@property (copy) void (^completionBlock)(void);
```

设置操作的优先级

```
@property double threadPriority;
```

设置操作的名称

```
@property (copy) NSString *name;
```

#### 2、带block的操作类实例——NSBlockOperation

NSBlockOperation是NSOperation的一个子类，其可以异步的执行多个block，当所有的block都完成时，这个操作才算完成。

初始化方法：

```
+ (instancetype)blockOperationWithBlock:(void (^)(void))block;
```

在操作中添加block

```
- (void)addExecutionBlock:(void (^)(void))block;
```

添加进去的block的数组

```
@property (readonly, copy) NSArray *executionBlocks;
```

示例如下：

```
NSBlockOperation * opera = [NSBlockOperation blockOperationWithBlock:^{
        for (int i=0; i<10; i++) {
            NSLog(@"%@=%d",[NSThread currentThread],i);
        }
    }];
    [opera addExecutionBlock:^{
        for (int i=0; i<10; i++) {
            NSLog(@"%@=%d",[NSThread currentThread],i);
        }
    }];
    [opera start];
```

打印情况如下，可以看出，两个block块的执行是异步的：

![](http://static.oschina.net/uploads/space/2015/0519/112940_KMcN_2340880.png)

#### 3、使用NSInvocationOperation调用方法

根据选择器创建一个对象

```
- (instancetype)initWithTarget:(id)target selector:(SEL)sel object:(id)arg;
```

通过Invocation创建一个对象

```
- (instancetype)initWithInvocation:(NSInvocation *)inv;
```

这个类执行的操作是与调用它的线程同步的，示例如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.  
    NSInvocationOperation * operation = [[NSInvocationOperation alloc]initWithTarget:self selector:@selector(log) object:nil];
    [operation start];
        for (int i=0; i<10; i++) {
        NSLog(@"%@=%d",[NSThread currentThread],i);
        }  
}


-(void)log{
    for (int i=0; i<100; i++) {
        NSLog(@"%@=%d",[NSThread currentThread],i);
    }
}
```

通过打印结果可以看出其执行的同步性。

### 二、操作之间的依赖关系

依赖关系和优先级的作用很像，却也不同。如果一个操作A依赖于另一个操作B，那么只有当B操作完成后，A操作才会执行。操作添加依赖的

添加一个依赖：

```
- (void)addDependency:(NSOperation *)op;
```

删除一个依赖

```
- (void)removeDependency:(NSOperation *)op;
```

原则上说，一个操作对象的依赖可以添加多个，并且当所有依赖都执行完成后才会执行这个操作。

### 三、NSOperationQueue操作队列

NSOperationQueue是操作队列类，通过上面的介绍，我们已经可以理解操作，并且操作默认的执行方式是串行的，尽管NSBlockOperation中的block块间是并行执行的，但其和外部操作依然是串行的。如果将操作放入操作队列中，则默认为并行执行的。

示例如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    NSOperationQueue * queue = [[NSOperationQueue alloc]init];
    NSInvocationOperation * op1 = [[NSInvocationOperation alloc]initWithTarget:self selector:@selector(log) object:nil];
    [queue addOperation:op1];
    for (int i=0; i<10; i++) {
        NSLog(@"%@=%d",[NSThread currentThread],i);
    }
}

-(void)log{
    for (int i=0; i<10; i++) {
        NSLog(@"%@=%d",[NSThread currentThread],i);
    }
}
```

打印信息如下：

![](http://static.oschina.net/uploads/space/2015/0519/152753_oft5_2340880.png)

可以看出来，队列的操作是在一个新的线程中执行的，并且操作队列之中的操作也都是异步执行的。

在操作队列中添加一个操作任务：

```
- (void)addOperation:(NSOperation *)op;
```

在队列中插入一组操作任务，后面的参数设置是否队列中得任务都执行完成后再执行这一组操作：

```
- (void)addOperations:(NSArray *)ops waitUntilFinished:(BOOL)wait;
```

在队列中添加一个block操作

```
- (void)addOperationWithBlock:(void (^)(void))block;
```

获取操作队列中的所有操作的数组

```
@property (readonly, copy) NSArray *operations;
```

获取操作队列中操作的个数

```
@property (readonly) NSUInteger operationCount;
```

设置队列最大并行操作数量

```
@property NSInteger maxConcurrentOperationCount;
```

设置是否暂停队列任务执行

```
@property (getter=isSuspended) BOOL suspended;
```

设置队列名字

```
@property (copy) NSString *name;
```

设置队列的优先级别（iOS8后支持）

```
@property NSQualityOfService qualityOfService;
```

取消队列中所有操作任务

```
- (void)cancelAllOperations;
```

阻塞当前线程，直到队列中所有任务完成

```
- (void)waitUntilAllOperationsAreFinished;
```

获取当前执行的队列

```
+ (NSOperationQueue *)currentQueue;
```

获取主线程中的操作队列

```
+ (NSOperationQueue *)mainQueue;
```

### 四、队列中操作的执行顺序法则

1、决定于依赖关系，只有当这个操作的依赖全部执行完成后，它才会被执行。

2、影响于优先级，优先级高的会先执行。

如有疏漏 欢迎指正

学习使用 欢迎转载

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

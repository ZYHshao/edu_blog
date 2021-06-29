---
title: iOS多线程编程之三——GCD的应用
date: 2015-05-21
categories: iOS逻辑初窥
tags: []
---
## iOS多线程编程之三——GCD的应用

### 一、引言

在软件开发中使用多线程可以大大的提升用户体验度，增加工作效率。iOS系统中提供了多种分线程编程的方法，在前两篇博客都有提及：

NSThread类进行多线程编程：[http://my.oschina.net/u/2340880/blog/416524](http://my.oschina.net/u/2340880/blog/416524)。

NSOperation进行多线程操作编程：[http://my.oschina.net/u/2340880/blog/416782](http://my.oschina.net/u/2340880/blog/416782)。

上两个进行多线程编程的机制都是封装于Object-C的类与方法。这篇博客将讨论的Grand Central Dispatch(GCD)机制，则是基于C语言的，相比上面两种机制，GCD更加高效，并且线程有系统管理，会自动运用多核运算。因为这些优势，GCD是apple推荐我们使用的多线程解决方案。

### 二、GCD的调度机制

GCD机制中一个很重要的概念是调度队列，我们对线程的操作实际上是由调度队列完成的。我们只需要将要执行的任务添加到合适的调度队列中即可。

#### 1、调度队列的类型

调度队列有三种类型：

（1）主队列

其中的任务在主线程中执行，因为其会阻塞主线程，所以这是一个串行的队列。可以通过dispatch\_get\_main_queue()方法得到。

（2）全局并行队列

队列中任务的执行方式是严格按照先进先出的模式进行了。如果是串行的队列，则当一个任务结束后，才会开启另一个任务，如果是并行队列，则任务的开启顺序是和添加顺序一致的。系统为iOS应用自动创建了四个全局共享的并发队列。使用如下函数获得：

dispatch\_get\_global_queue(long identifier, unsigned long flags);

其中第一个参数是这个队列的id，系统的四个全局队列默认的优先级不同，这个参数可填的定义如下：

```
#define DISPATCH_QUEUE_PRIORITY_HIGH 2//优先级最高的全局队列
#define DISPATCH_QUEUE_PRIORITY_DEFAULT 0//优先级中等的全局队列
#define DISPATCH_QUEUE_PRIORITY_LOW (-2)//优先级低的全局队列
#define DISPATCH_QUEUE_PRIORITY_BACKGROUND INT16_MIN//后台的全局队列 优先级最低
```

这个函数的第二个参数，按照官方文档的说法是有待未来使用，现在我们都填0即可。

（3）自定义队列

上面的两种队列都是系统为我们创建好的，我们只需要获取到他们，将任务添加即可。当然，我们可可以创建我们自己的队列，包括串行的和并行的。使用如下方法创建：

```
dispatch_queue_t queue = dispatch_queue_create("myQueue", DISPATCH_QUEUE_SERIAL);
```

其中，第一个参数是这个队列的名字，第二个参数决定创建的是串行的还是并行的队列。填写DISPATCH\_QUEUE\_SERIAL或者NULL创建串行队列，填写DISPATCH\_QUEUE\_CONCURRENT创建并行队列。

#### 2、添加任务到队列中

使用dispatch_sync(dispatch\_queue\_t queue, dispatch\_block\_t block)函数或者dispatch_async(dispatch\_queue\_t queue, dispatch\_block\_t block)函数来同步或者异步的执行任务，示例如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    dispatch_queue_t queue = dispatch_queue_create("myQueue", DISPATCH_QUEUE_SERIAL);
    dispatch_sync(queue, ^{
        NSLog(@"%@:1",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"%@:2",[NSThread currentThread]);
    });
}
```

打印结果如下：

![](http://static.oschina.net/uploads/space/2015/0521/135803_9ro1_2340880.png)

可以看出第一个任务在主线程中执行，第二个在分线程中执行。

### 三、队列调度机制的更多技巧

通过上面的演示，我们已经可以运用队列进行多线程的执行任务，但是GCD的强大之处远远不止如此。

#### 1、使用队列组

如果有这样三个任务，A与B是没有关系的，他们可以并行执行，C必须在A,B结束之后才能执行，当然，实现这样的逻辑并不困难，使用KVO就可以实现，但是使用队列组处理这样的逻辑，代码会更加清晰简单。

可以使用dispatch\_group\_create()创建一个队列组，使用如下函数将队列添加到队列组中：

```
void dispatch_group_async(dispatch_group_t group,
    dispatch_queue_t queue,
    dispatch_block_t block);
```

队列组中的队列是异步执行的，示例如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    //创建一个队列组
    dispatch_group_t group=dispatch_group_create();
    创建一个异步队列
    dispatch_queue_t queue=dispatch_queue_create("myQueue", DISPATCH_QUEUE_CONCURRENT);
    dispatch_group_async(group, queue, ^{
        for (int i=0; i<10; i++) {
            NSLog(@"%@:%d",[NSThread currentThread],i);
        }
    });
    dispatch_group_async(group, queue, ^{
        for (int i=0; i<10; i++) {
            NSLog(@"%@:%d",[NSThread currentThread],i);
        }
    });
    //阻塞线程直到队列任务完成
    dispatch_group_wait(group,DISPATCH_TIME_FOREVER);
    for (int i=0; i<10; i++) {
        NSLog(@"over:%d",i);
    }
}
```

打印出来的信息如下：

![](http://static.oschina.net/uploads/space/2015/0521/143809_4Hxj_2340880.png)

可以看出，队列中的任务是异步执行的，并且等待队列组中队列任务全部执行后才执行后面的任务。这样的做法在实际应用中我们很少使用，通常我们会把后续的任务在放在异步中执行，做法如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    //创建一个队列组
    dispatch_group_t group=dispatch_group_create();
    //创建一个队列
    dispatch_queue_t queue=dispatch_queue_create("myQueue", DISPATCH_QUEUE_CONCURRENT);
    //添加队列任务到队列组
    dispatch_group_async(group, queue, ^{
        for (int i=0; i<10; i++) {
            NSLog(@"%@:%d",[NSThread currentThread],i);
        }
    });
    dispatch_group_async(group, queue, ^{
        for (int i=0; i<10; i++) {
            NSLog(@"%@:%d",[NSThread currentThread],i);
        }
    });
    //队列组任务执行完后执行的任务
    dispatch_group_notify(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        for (int i=0; i<10; i++) {
            NSLog(@"over:%d",i);
        }
    });
    for (int i=0; i<10; i++) {
         NSLog(@"Finish:%d",i);
    }
   
}
```

打印信息如下：

![](http://static.oschina.net/uploads/space/2015/0521/144444_9Vrz_2340880.png)

可以看出GCD的强大了吧，复杂的任务逻辑关系因为GCD变得十分清晰简单。

#### 2、循环机制

一开始我们就提到，GCD相比NSOperation的优势在于多核心的应用，更深得挖掘出了硬件的性能。GCD在多核方面的一个明显的特点就是循环机制。

```
 dispatch_apply(10, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^(size_t i) {
        NSLog(@"%@:%zu",[NSThread currentThread],i);
    });
```

打印结果如下：

![](http://static.oschina.net/uploads/space/2015/0521/145607_C60q_2340880.png)

可以看出，程序的运行效率又会高许多。

#### 3、消息传递机制

dispatch\_source\_t类型的对象可以用来传递和接受某个消息，然后执行block方法，示例如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    //创建一个数据对象，DISPATCH_SOURCE_TYPE_DATA_ADD的含义表示数据变化时相加
    dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_DATA_ADD, 0, 0, dispatch_get_main_queue());
    //创建接收数据变化的句柄
    dispatch_source_set_event_handler(source, ^{
        NSLog(@"%lu",dispatch_source_get_data(source));
    });
    //启动
    dispatch_resume(source);
    //设置数据
    dispatch_source_merge_data(source, 1);
    //这步执行完之后会执行打印方法
}
```

#### 4、发送和等待信号

GCD中还有一个重要的概念是信号量。它的用法法消息的传递有所类似，通过代码来解释：

```
    //创建一个信号，其中的参数为信号的初始值
    dispatch_semaphore_t singer = dispatch_semaphore_create(0);
    //发送信号，使信号量+1
    dispatch_semaphore_signal(singer);
    //等待信号，当信号量大于0时执行后面的方法，否则等待，第二个参数为等待的超时时长，下面设置的为一直等待
    dispatch_semaphore_wait(singer, DISPATCH_TIME_FOREVER);
    NSLog(@"123");
```

通过发送信号，可以试信号量+1，每次执行过等待信号后，信号量会-1；如此，我们可以很方便的控制不同队列中方法的执行流程。

#### 5、挂起和开启任务队列

GCD还提供了暂停与开始任务的方法，使用

void dispatch_suspend(dispatch\_object\_t object);

可以将队列或者队列组进行暂时的挂起，使用

void dispatch_resume(dispatch\_object\_t object);

将队列或者队列组重新开启。

需要注意的是，暂停队列时，队列中正在执行的任务并不会被中断，会挂起未开启的任务。  
 

### 四、关于内存管理

GCD虽然是基于C语言封装的框架，使用了面向对象的思想。因此，它的内存管理是需要我们注意的，不论是ARC或者MRC，我们都应该手动去处理这些对象。还好，GCD的内存管理思路和Object—C是兼容的，我们使用dispatch\_retain()和dispatch\_release()来将引用对象的计数进行加减。这一点十分重要，切记切记。

疏漏之处 欢迎指正

学习使用 欢迎转载

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: Objective-C中runtime机制的应用
date: 2015-08-07
categories: iOS逻辑初窥
tags: []
---
## Objective-C中runtime机制的应用

### 一、初识runtime

        Objective-C是一种动态语言，所谓动态语言，是在程序执行时动态的确定变量类型，执行变量类型对应的方法的。因此，在Object-C中常用字符串映射类的技巧来动态创建类对象。因为OC的动态语言特性，我们可以通过一些手段，在程序运行时动态的更改对象的变量甚至方法，这就是我们所说的runtime机制。

### 二、你还有什么办法操作这样的变量么？

        首先，我们先来看一个例子，这里有我创建的一个MyObject类：

```
//.h===========================
@interface MyObject : NSObject
{
    @private
    int privateOne;
    NSString * privateTow;;
}
@end
//=============================
//.m===========================
@interface MyObject()
{
    @private
    NSString * privateThree;
}
@end
@implementation MyObject
- (instancetype)init
{
    self = [super init];
    if (self) {
        privateOne=1;
        privateTow=@"Tow";
        privateThree=@"Three";
    }
    return self;
}
-(NSString *)description{
    return [NSString stringWithFormat:@"one=%d\ntow=%@\nthree=%@\n",privateOne,privateTow,privateThree];
}
@end
//=============================
```

这个类是相当的安全，首先，在头文件中没有提供任何的方法接口，我们没有办法使用点语法做任何操作，privateOne和PrivateTow两个变量虽然声明在了头文件中，却是私有类型的，通过指针的方式我们虽然可以看到他们，却不能做任何读取修改的操作，xcode中的提示如下：

![](http://static.oschina.net/uploads/space/2015/0807/101507_o20G_2340880.png)

![](http://static.oschina.net/uploads/space/2015/0807/101507_hsiW_2340880.png)

他会告诉我们，这是一个私有的变量，我们不能使用。对于privateThree，我们更是束手无策，不仅不能使用，我们甚至都看不到它的存在。那么对于这种情况，你有什么办法操作这些变量么？对，是时候展现真正的技术了：runtime!

### 三、通过runtime获取对象的变量列表

        要操作对象的变量，我们首先应该要捕获这些变量，让他们无处遁形。无论声明在头文件或是实现文件，无论类型是公开的还是私有的，只要声明了这个变量，系统就会为其分配空间，我们就可以通过runtime机制捕获到它，代码如下：

```
#import "ViewController.h"
#import "MyObject.h"
//包含runtime头文件
#import <objc/runtime.h>
@interface ViewController ()
@end
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    //我们先声明一个unsigned int型的指针，并为其分配内存
    unsigned int * count = malloc(sizeof(unsigned int));
    //调用runtime的方法
    //Ivar：方法返回的对象内容对象，这里将返回一个Ivar类型的指针
    //class_copyIvarList方法可以捕获到类的所有变量，将变量的数量存在一个unsigned int的指针中
    Ivar * mem = class_copyIvarList([MyObject class], count);
    //进行遍历
    for (int i=0; i< *count ; i++) {
        //通过移动指针进行遍历
        Ivar var = * (mem+i);
        //获取变量的名称
        const char * name = ivar_getName(var);
        //获取变量的类型
        const char * type = ivar_getTypeEncoding(var);
        NSLog(@"%s:%s\n",name,type);
    }
    //释放内存
    free(count);
    //注意处理野指针
    count=nil;
}
- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

@end
```

打印结果如下，其中i表示int型：

![](http://static.oschina.net/uploads/space/2015/0807/103618_uYAs_2340880.png)

是不是小吃惊了一下，无论变量在哪里，只要它在，就让它无处遁形。

### 四、让我找到你，就让我改变你！

        仅仅能够获得变量的类型和名字或许并没有什么卵用，没错，我们获取变量的目的不是为了观赏，而是为了操作它，这对runtime来说，也是小事一碟。代码如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    //获取变量
    unsigned int  count;
    Ivar * mem = class_copyIvarList([MyObject class],&count);
    //创建对象
    MyObject * obj = [[MyObject alloc]init];
    NSLog(@"before runtime operate:%@",obj);
    //进行变量的设置
    object_setIvar(obj, mem[0],10);
    object_setIvar(obj, mem[1], @"isTow");
    object_setIvar(obj, mem[2], @"isThree");
    NSLog(@"after runtime operate:%@",obj);
    
}
```

Tip:在修改int型变量的时候，你或许会遇到一个问题，ARC下，编译器不允许你将int类型的值赋值给id，在buildset中将Objective-C Automatic Reference Counting修改为No即可。

打印效果如下：

![](http://static.oschina.net/uploads/space/2015/0807/113224_lHbT_2340880.png)

可以看到，那些看似非常安全的变量被我们修改了。

### 五、让我看看你的方法吧

        变量通过runtime机制我们可以取到和改变值，那么我们再大胆一点，试试那些私有的方法，首先我们在MyObject类中添加一些方法，我们只实现，并不声明他们：

```
@interface MyObject()
{
    @private
    NSString * privateThree;
}
@end
@implementation MyObject
- (instancetype)init
{
    self = [super init];
    if (self) {
        privateOne=1;
        privateTow=@"Tow";
        privateThree=@"Three";
    }
    return self;
}
-(NSString *)description{
    return [NSString stringWithFormat:@"one=%d\ntow=%@\nthree=%@\n",privateOne,privateTow,privateThree];
}
-(NSString *)method1{
    return @"method1";
}
-(NSString *)method2{
    return @"method2";
}
```

这样的方法我们在外面是无法调用他们的，和操作变量的思路一样，我们先要捕获这些方法：

```
    //获取所有成员方法
    Method * mem = class_copyMethodList([MyObject class], &count);
    //遍历
    for(int i=0;i<count;i++){
        SEL name = method_getName(mem[i]);
        NSString * method = [NSString stringWithCString:sel_getName(name) encoding:NSUTF8StringEncoding];
        NSLog(@"%@\n",method);
    }
```

打印如下：

![](http://static.oschina.net/uploads/space/2015/0807/120623_BAL8_2340880.png)

得到了这些方法名，我们大胆的调用即可：

```
    MyObject * obj = [[MyObject alloc]init];
    NSLog(@"%@",[obj method1]);
```

Tip:这里编译器不会给我们方法提示，放心大胆的调用即可。

### 六、动态的为类添加方法

        这个runtime机制最强大的部分要到了，试想，如果我们可以动态的向类中添加方法，那将是一件多么令人激动的事情，注意，这里是动态的添加，和类别的最大不同在于这种方式是运行时才决定是否添加方法的。

```
- (void)viewDidLoad {
    [super viewDidLoad];
    //添加一个新的方法，第三个参数是返回值的类型v是void,i是int，：是SEL，对象是@等
    class_addMethod([MyObject class], @selector(method3), (IMP)logHAHA, "v");
    
    unsigned int count = 0;
    Method * mem = class_copyMethodList([MyObject class], &count);
    for(int i=0;i<count;i++){
        SEL name = method_getName(mem[i]);
        NSString * method = [NSString stringWithCString:sel_getName(name) encoding:NSUTF8StringEncoding];
        NSLog(@"%@\n",method);
    }
    
    MyObject * obj = [[MyObject alloc]init];
    //运行这个方法
     [obj performSelector:@selector(method3)];
    
}
//方法的实现
void logHAHA(){
    NSLog(@"HAHA");
}
```

运行结果如下：

![](http://static.oschina.net/uploads/space/2015/0807/135458_HecD_2340880.png)

从前五行可以看出，方法已经加进去了，从最后一行可以看出，执行没有问题。

### 七、做点小手脚

        程序员总是得寸进尺的，现在，我们要做点事情，用我们的函数替换掉类中的函数：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    MyObject * obj = [[MyObject alloc]init];
    //替换之前的方法
    NSLog(@"%@", [obj method1]);
    //替换
    class_replaceMethod([MyObject class], @selector(method1), (IMP)logHAHA, "v");
    [obj method1];
    
}
void logHAHA(){
    NSLog(@"HAHA");
}
```

打印如下：

![](http://static.oschina.net/uploads/space/2015/0807/140636_WuVm_2340880.png)

这次够cool吧，通过这个方法，我们可以把系统的函数都搞乱套。当然，runtime还有许多很cool的方法：

id object\_copy(id obj, size\_t size)

拷贝一个对象

id object_dispose(id obj)

释放一个对象

const char *object_getClassName(id obj)

获取对象的类名

ive

void method_exchangeImplementations(Method m1, Method m2)  
交换两个方法的实现

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

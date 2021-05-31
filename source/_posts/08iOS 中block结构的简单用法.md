---
title: iOS 中block结构的简单用法
date: 2015-04-09   
categories: iOS编程技巧
tags: [Block,iOS编程]              
---
自从block出现之后，很多API都开始采用这样的结构，由此可见，block确实有许多优势存在，这里将一些简单用法总结如下：

一、如何声明一个block变量

我们通过^符号来声明block类型，形式如下：

void (^myBlock)();

其中第一个void是返回值，可以是任意类型，中间括号中^后面的是这个block变量的名字，我把它命名为myBlock，最后一个括号中是参数，如果多参数，可以写成如下样式：

int (^myBlock)(int,int);

同样，你也可以给参数起名字：

int (^myBlock)(int a,int b);

很多时候，我们需要将我们声明的block类型作为函数的参数，也有两种方式：

1、-(void)func:(int (^)(int a,int b))block；

第二种方式是通过typedef定义一种新的类型，这也是大多数情况下采用的方式：

2、typedef int (^myBlock)(int a,int b) ;

-(void)func:(myBlock)block ;

二、如何实现一个block

既然block可以被声明为变量，那么就一定可以实现它，就像其他类型变量的赋值。我自己对block的理解为它是一断代码块，所以给它赋值赋便是一段代码段：

```
typedef int (^myBlock)(int,int) ;
@interface ViewController ()
{
    myBlock block1;
}
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    block1 =^(int a, int b){
        return a+b;
    };
   NSLog(@"%d",block1(1,1));
}
```

这里打印的结果是2，从这里可以发现block和函数的功能很像。

注意：1、在上面的代码里 block1是一个对象，如果直接打印将打印对象地址

        2、block()，加上后面的括号才是执行block语句块

三、block中访问对象的微妙关系

1、如果你在一个block块中仅仅访问对象，而不是对他进行修改操作，是没有任何问题的：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    int tem=2;
    block1 = ^(int a,int b){
        int count= tem+1;
        return count;
    };
    NSLog(@"%d",block1(1,1));
}
```

而如果我在block块中直接修改，编译器会报错：

```
  block1 = ^(int a,int b){
        tem+=1;
        return tem+1;
    };
```

为什么会出现这样的情况，根据猜测，可能是block内部将访问的变量都备份了一份，如果我们在内部修改，外部的变量并不会被修改，我们可以通过打印变量的地址来证明这一点：

```
- (void)viewDidLoad {
    [super viewDidLoad]; 
    int tem=2;
    NSLog(@"%p",&tem);
    block1 = ^(int a,int b){
        NSLog(@"%p",&tem);
        return tem+1;
    };
    NSLog(@"%d",block1(1,1)); 
}
```

打印结果如下：

![](http://static.oschina.net/uploads/space/2015/0409/144646_DHeU_2340880.png)

可以看出，变量的地址已经改变。

2、__block 做了什么

为了可以在block块中访问并修改外部变量，我们常会把变量声明成__block类型，通过上面的原理，可以发现，其实这个关键字只做了一件事，如果在block中访问没有添加这个关键字的变量，会访问到block自己拷贝的那一份变量，它是在block创建的时候创建的，而访问加了这个关键字的变量，则会访问这个变量的地址所对应的变量。我们可以通过代码来证明：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    int tem=2;
    block1 = ^(int a,int b){  
        return tem+a+b;
    };
    tem=4;
    NSLog(@"%d",block1(1,1));
    
    block1 = ^(int a,int b){  
        return tem+a+b;
    };
     __block int tem2=2;
    tem2=4;
     NSLog(@"%d",block1(1,1));
}
```

结果：

![](http://static.oschina.net/uploads/space/2015/0409/145816_uxqT_2340880.png)

3、一点点扩展

由此，我们可以理解，如果block中操作的对象是指针，那么直接可以进行修改，这包括OC对象，如果不是，则需要用__block关键字修饰。

4、关于引用计数

在block中访问的对象，会默认retain：

```
    UIImage * number;
    number = [[UIImage alloc]init] ;
    NSLog(@"%ld",CFGetRetainCount((__bridge CFTypeRef)number));
    block1 = ^(int a,int b){
        NSLog(@"%ld",CFGetRetainCount((__bridge CFTypeRef)number));
    };
    NSLog(@"%ld",CFGetRetainCount((__bridge CFTypeRef)number));
```

结果如下：

![](http://static.oschina.net/uploads/space/2015/0409/152914_JKev_2340880.png)

而添加__block的对象不会被retain;

注意：如果我们访问类的成员变量，或者通过类方法来访问对象，那么这些对象不会被retain，而类对象会被return，最常见的时self:

```
typedef void(^myBlock)(int,int) ;
@interface ViewController2 ()
{
     myBlock block1;
     __block UIImage * number; 
}
@end
@implementation ViewController2
-(void)dealloc{
    NSLog(@"dealloc %@",self.class);
    NSLog(@"%ld",CFGetRetainCount((__bridge CFTypeRef)number));
}
- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor=[UIColor whiteColor];
    number = [[UIImage alloc]init] ;
    NSLog(@"%ld",CFGetRetainCount((__bridge CFTypeRef)number));
    block1 = ^(int a,int b){
        NSLog(@"%ld",CFGetRetainCount((__bridge CFTypeRef)number));
    };
    //block1(1,1);
    NSLog(@"%ld",CFGetRetainCount((__bridge CFTypeRef)number));
    
    UIButton * btn = [UIButton buttonWithType:UIButtonTypeCustom];
    btn.frame=CGRectMake(100, 100, 100, 100);
    btn.backgroundColor=[UIColor redColor];
    [self.view addSubview:btn];
    [btn addTarget:self action:@selector(click) forControlEvents:UIControlEventTouchUpInside];
}
-(void)click{
    [self dismissViewControllerAnimated:YES completion:nil];
}
```

打印结果：

![](http://static.oschina.net/uploads/space/2015/0409/153406_LBTn_2340880.png)

可以看出，UIImage对象没有被retain,而self也将循环引用，造成内存泄露。解决方法如下：

```
 number = [[UIImage alloc]init] ;
    NSLog(@"%ld",CFGetRetainCount((__bridge CFTypeRef)number));
    UIImage * im = number;
    block1 = ^(int a,int b){
        NSLog(@"%ld",CFGetRetainCount((__bridge CFTypeRef)im));
    };
    NSLog(@"%ld",CFGetRetainCount((__bridge CFTypeRef)number));
```

打印结果：

![](http://static.oschina.net/uploads/space/2015/0409/153752_wKKJ_2340880.png)

注意：根据这个机制，如果我们将block用来传值，在block不用时，务必要置为nil,而在实现block的方法里，务必要释放;我们通过代码来解释：

首先，创建三个ViewController，为ViewController1，ViewController2，ViewController3；

1、在ViewController1中创建一个按钮，跳转ViewController2

2、在ViewController2中：

```
#import "ViewController2.h"
#import "ViewController3.h"
@interface ViewController2 ()
{
     UIButton * im;
}
@end

@implementation ViewController3
-(void)dealloc{
    NSLog(@"dealloc %@",self.class);
}
- (void)viewDidLoad {
    [super viewDidLoad];
    UIButton * btn = [UIButton buttonWithType:UIButtonTypeCustom];
    btn.frame=CGRectMake(300, 300, 100, 100);
    btn.backgroundColor=[UIColor redColor];
    [btn addTarget:self action:@selector(click) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:btn];
    im = [[UIButton alloc]initWithFrame:CGRectMake(100, 100, 100, 100)];
    im.backgroundColor=[UIColor blackColor];
    [im addTarget:self action:@selector(rele) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:im];
}
-(void)rele{
    [self dismissViewControllerAnimated:YES completion:nil];
}
-(void)click{
    ViewController3 * con = [[ViewController3 alloc]init];
    [con setBlock:^{
        im.backgroundColor=[UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    }];
    [self presentViewController:con animated:YES completion:nil];
}
```

3、在ViewController3中：

```
#import "ViewController3.h"
void (^myBlock)();
@implementation ViewController3
-(void)setBlock:(void(^)())block{
    myBlock = [block copy];
}
-(void)dealloc{
    NSLog(@"dealloc %@",self.class);
}
- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor=[UIColor whiteColor];
    myBlock();   
    UIButton * btn = [UIButton buttonWithType:UIButtonTypeCustom];
    btn.frame=CGRectMake(100, 100, 100, 100);
    btn.backgroundColor=[UIColor redColor];
    [self.view addSubview:btn];
    [btn addTarget:self action:@selector(click) forControlEvents:UIControlEventTouchUpInside];
}
-(void)click{
    [self dismissViewControllerAnimated:YES completion:nil];
}
```

通过打印信息，我们会发现，ViewController2不被释放，原因是其成员变量im被block中retain没有释放，我们这样做：

```
@interface ViewController2 ()
{
    UIButton * im;
    ViewController3 * tem;
}
-(void)rele{
    [tem setBlock:nil];
    [self dismissViewControllerAnimated:YES completion:nil];
}
-(void)click{
    ViewController3 * con = [[ViewController2 alloc]init];
    tem=con;
    [con setBlock:^{
        im.backgroundColor=[UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    }];
    [self presentViewController:con animated:YES completion:nil];
}
```

这样就解决了内存问题。

四、关于block的作用域

应避免将花括号中的block用于外面，如果需要，你可以将这个block声明为全局的。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

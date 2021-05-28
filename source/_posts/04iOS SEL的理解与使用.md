---
title: iOS SEL的理解与使用
date: 2015-04-05   
categories: iOS编程技巧
tags: [SEL,iOS编程]              
---
   有很多人，认为block的推广可取代代理设计模式，其实block并不能取代代理，代理的模式可以让代码逻辑性更强，更整洁，也会有更高的可读性和可扩展性。相比之下，我觉得block更多的是取代了选择器@selector。

   @selector是什么？我们要首先明白SEL，SEL并不是一种对象类型，我们通过xCode的字体颜色就可以判断出来，它是一个关键字，就像int，long一样，它声明了一种类型：类方法指针。其实就可以理解为一个函数指针。比如，我们生命一个叫myLog的函数指针：

```
#import "ViewController.h"

@interface ViewController ()
{
    SEL myLog;
}
@end
```

声明出了这个指针，我们该如何给它传递这个函数呢？有两种方式：

1、在编译时，使用@selector来取得函数

现在，我们应该明白@selector是什么了，它是一个编译标示，我们通过它来取到相应函数。

```
@interface ViewController ()
{
    SEL myLog;
}
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    myLog = @selector(myLogL);
    //通过performSelector来执行方法
   [self performSelector:myLog];//打印 “myLog”
   
}
-(void)myLogL{
    NSLog(@"myLog");
}
```

2、在运行时，通过NSSelectorFromString方法来取到相应函数：

```
#import "ViewController.h"

@interface ViewController ()
{
    SEL myLog;
}
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    myLog = NSSelectorFromString(@"myLogN");
    [self performSelector:myLog];
   
}


-(void)myLogN{
    NSLog(@"myLog");
}
```

这两种方式的差别在于，编译时的方法如果没有找到相应函数，xcode会报错，而运行时的方法不会。

至于SEL的应用，我相信最广泛的便是target——action设计模式了。我们来简单模拟一下系统button的工作原理：

我们先创建一个继承于UIButton的类：

.h文件：

```
#import <UIKit/UIKit.h>

@interface Mybutton : UIButton
-(void)addMyTarget:(id)target action:(SEL)action;
@end
```

.m文件

```
#import "Mybutton.h"

@implementation Mybutton
{
    SEL _action;
    id _target;
}
-(void)addMyTarget:(id)target action:(SEL)action{
    _target=target;
    _action=action;
}
-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{
    [_target performSelector:_action];
}


@end
```

在外部：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    Mybutton * btn = [[Mybutton alloc]initWithFrame:CGRectMake(100, 100, 60, 60)];
    btn.backgroundColor=[UIColor redColor];
    [btn addMyTarget:self action:@selector(click)];
    [self.view addSubview:btn];
}

-(void)click{
    NSLog(@"点击了btn");
}
```

当然，如果要调用参数，系统提供的默认参数不超过两个，如果参数很多，一种是我们可以通过字典传参，另一种方法比较复杂，在这里先不讨论。

错误之处，欢迎指正

欢迎转载，注明出处

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

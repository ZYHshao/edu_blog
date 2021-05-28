---
title: iOS事件响应控制
date: 2015-04-04   
categories: iOS编程技巧
tags: [事件,iOS编程]              
---
    以前遇到一个项目，一个UIImageView对象上面有一个UIButton对象，然而项目的需求需要在点击 button的同时，UIImageView也接收到点击事件，在不使用代理和通知方法的前提下，通过事件响应链的原理，我们也可以很便捷的解决这个问题。

    在处理这个问题之前，我们应该先清楚IOS的事件响应机制到底是个什么样的原理。

首先，这个事件响应的机制是分为两个部分的。

1、先在视图层级关系中找到应该响应事件的那个视图。

这一步是什么意思，其实很简单，就是找到你所触摸点对应的那个最上层的视图，它的工作原理是这样的：当用户发出事件后，会产生一个触摸事件，系统会将该事件加入到一个由UIApplication管理的事件队列中，UIApplication会取出队列中最前面的事件，发消息给UIWindow，然后UIWindow会对其所有子视图调用hitTest:withEvent:这个方法，这个方法会返回一个UIView的对象，这个方法在执行的时候，它会调用当前视图的pointInside:withEvent:这个方法，如果触摸事件在当前视图范围内，pointInside:withEvent:会返回YES，否则会返回NO；如果返回YES，则会遍历当前视图的所有子视图，统统发送hitTest:withEvent:这个消息，如果返回NO,则hitTest:withEvent:方法返回nil；

上面说起来有些绕，其实就是：hitTest:withEvent:方法会一层一层的向上找，若最上层响应的子视图pointInside:withEvent:返回YES，则返回此子视图，如果所有的都返回nil，则返回当前视图本身self。

例如：我们建两个文件，一个继承于UIButton，一个继承于UIImageView，我们在UIImageView里的代码如下：

```
#import "MyImageView.h"

@implementation MyImageView
- (instancetype)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        self.backgroundColor=[UIColor redColor];
    }
    return self;
}
//在这里，我们重写了这个方法，让它直接返回自身，而不是继续向下寻找应该响应事件的视图
-(UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event{
    return self;
}

-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{
    NSLog(@"点击了Image");
}
```

然后将他们创建在一个View上：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    MyImageView * image = [[MyImageView alloc]initWithFrame:CGRectMake(60, 80, 200, 200)];
    MyButton * btn =[UIButton buttonWithType:UIButtonTypeSystem];
    btn.frame=CGRectMake(20, 20, 40, 40);
    [btn setTitle:@"button" forState:UIControlStateNormal];
    [image addSubview:btn];
    [self.view addSubview:image];
    // Do any additional setup after loading the view, typically from a nib.
}
```

我们运行，点击这个Btn，会打印如下的信息：![](http://static.oschina.net/uploads/space/2015/0404/223950_55Ql_2340880.png)

可以证明，在事件视图寻找中，UIImageView我们重写hitTest:withEvent:方法后，切断了寻找链，如果我们这个做：

```
-(UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event{
    return nil;
}
```

你会发现，UIImageView也不再接收事件。  
2、寻找到应该响应的视图后，会进行消息处理，这个处理的方式是通过消息处理链来做的。如果它自身不能处理消息，会通过nextResponder将消息传递给下一个处理者，默认只要有一个view将消息处理了，这个消息处理传递链将不再传递。

现在，我们把刚才UIimageView里重写的hitTest:withEvent:方法注释掉，给btn添加一个点击方法，同时将用户交互关闭：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    MyImageView * image = [[MyImageView alloc]initWithFrame:CGRectMake(60, 80, 200, 200)];
    MyButton * btn =[UIButton buttonWithType:UIButtonTypeSystem];
    image.userInteractionEnabled=YES;
    btn.frame=CGRectMake(20, 20, 40, 40);
    [btn setTitle:@"button" forState:UIControlStateNormal];
    [image addSubview:btn];
    [self.view addSubview:image];
    
    
    [btn addTarget:self action:@selector(click) forControlEvents:UIControlEventTouchUpInside];
     btn.userInteractionEnabled=NO;
    // Do any additional setup after loading the view, typically from a nib.
}

-(void)click{
    NSLog(@"btn被点击了");
}
```

这样，我们的UIImageView又可以响应事件了，原因是事件处理传递链向下传递了。

现在，在回到我们刚开始的问题，如何让btn响应的同时imageView也响应，我们这样做：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    MyImageView * image = [[MyImageView alloc]initWithFrame:CGRectMake(60, 80, 200, 200)];
    image.userInteractionEnabled=YES;
    MyButton * btn =[UIButton buttonWithType:UIButtonTypeSystem];
    btn.frame=CGRectMake(20, 20, 40, 40);
    [btn setTitle:@"button" forState:UIControlStateNormal];
    [image addSubview:btn];
    [self.view addSubview:image];
    
    
    [btn addTarget:self action:@selector(click:) forControlEvents:UIControlEventTouchUpInside];
    btn.userInteractionEnabled=NO;
    // Do any additional setup after loading the view, typically from a nib.
}

-(void)click:(UIButton *)btn{
    NSLog(@"btn被点击了");
    //响应链继续传递
    [btn.nextResponder touchesBegan:nil withEvent:nil];
    
}
```

结果如下：

![](http://static.oschina.net/uploads/space/2015/0404/230656_r8XY_2340880.png)

虽然最终，我们完成了这个需求，可是我建议你最好不要这么干，因为这样的逻辑是违背现实生活中人们的行为认知的，更重要的是，我们的项目最后也确实改掉了这样的逻辑~~~

错误之处，欢迎指正

欢迎转载，注明出处

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

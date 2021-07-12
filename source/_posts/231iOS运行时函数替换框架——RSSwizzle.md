---
title: iOS运行时函数替换框架——RSSwizzle
date: 2016-06-30
categories: iOS第三方库
tags: []
---
## iOS运行时函数替换框架——RSSwizzle

### 一、引言

        Objective-C是的运行时特性在iOS开发中应用广泛，通过runtime方法，开发者可以在运行时动态为类添加方法，修改类的方法，系统的class\_addMethod()方法和class\_replaceMethod()方法可以十分简单的添加和修改方法，然而，直接使用这两个函数有时并不安全，其主要问题有如下几点：

1.在进行动态函数修改的时候，有可能其他线程也在做同样的操作。

2.在继承中，子类执行父类替换的方法会出现问题。

3.函数的替换必须依靠_cmd参数。

4.可能会出现命名冲突。

有关Objective-C运行时的相关内容可在如下博客中查看：[http://my.oschina.net/u/2340880/blog/489072](http://my.oschina.net/u/2340880/blog/489072)。

        RSSwizzle框架可以解决上面所有问题，在要求比较高的项目中如果需要使用到运行时函数替换的需求，可以直接使用这个框架。git地址如下：

[https://github.com/rabovik/RSSwizzle](https://github.com/rabovik/RSSwizzle)。

### 二、RSSwizzle的使用

        RSSwizzle中提供了两种使用方式，一种是通过调用类方法来实现函数的替换，另一种是使用RSSwizzle定义的宏来进行函数的替换。使用类方法的方式示例如下：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];  
 //替换类方法 
   [RSSwizzle swizzleClassMethod:NSSelectorFromString(@"log") inClass:NSClassFromString(@"ViewController") newImpFactory:^id(RSSwizzleInfo *swizzleInfo) {
        return ^(__unsafe_unretained id self){
            NSLog(@"Class log Swizzle");
        };
    }];
}
+(void)log{
    NSLog(@"Class log");
}

```

这个函数用来替换类方法，第1个参数为要替换的函数选择器，第2个参数为要替换此函数的类，block参数中需要返回一个方法函数，这个函数为要替换成的函数，要和原函数类型相同。在类中的函数默认都会有一个名为self的id参数。进行实例函数的替换实例代码如下：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    //进行实例方法的替换
    /*
     第一个参数为要替换的函数，第二个参数为要替换方法的类，第三个的block中返回替换后的方法，第四个参数设置替换模式，最后一个参数是此替换操作的标识符
    */
    [RSSwizzle swizzleInstanceMethod:NSSelectorFromString(@"touchesBegan:withEvent:") inClass:NSClassFromString(@"ViewController") newImpFactory:^id(RSSwizzleInfo *swizzleInfo) {
        return ^(__unsafe_unretained id self,NSSet* touches,UIEvent* event){
            NSLog(@"text Swizzle");
        };
    } mode:RSSwizzleModeAlways key:@"key"];
}
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    NSLog(@"test");
    [ViewController log];
}
```

替换模式的枚举RSSwizzleMode意义如下：

```objectivec
typedef NS_ENUM(NSUInteger, RSSwizzleMode) {
    //任何情况下 始终执行替换操作
    RSSwizzleModeAlways = 0,
    //相同key标识的替换操作只会被执行一次
    RSSwizzleModeOncePerClass = 1,
    //相同key标识的替换操作在子类父类中只会被执行一次
    RSSwizzleModeOncePerClassAndSuperclasses = 2
};
```

使用宏的模式进行方法替换操作的代码更加简单，示例如下：

```objectivec
    //进行类方法的替换
    /*
     第1个参数为要替换方法的类 第二个参数为要替换的方法选择器 第三个参数为方法的返回值类型，第四个参数为方法的参数列表，最后一个参数为要替换的方法代码块
     */
    RSSwizzleClassMethod(NSClassFromString(@"ViewController"), NSSelectorFromString(@"log"), RSSWReturnType(void), RSSWArguments(), RSSWReplacement(
                                                                                                                                    {
                                                                                                                                //先执行原始方法
                                                                                                                                        RSSWCallOriginal();                                                                    NSLog(@"Class log Swillze");                                                                             }));
    //进行实例方法的替换
    /*
     第一个参数为要替换方法的类，第二个参数为要替换的方法选择器，第三个参数为返回值类型，第四个参数为参数列表 第五个参数为要替换的代码块，第六个参数为执行模式，最后一个参数为key值标识。
     */
    RSSwizzleInstanceMethod(NSClassFromString(@"ViewController"),NSSelectorFromString(@"touchesBegan:withEvent:"), RSSWReturnType(void), RSSWArguments(NSSet* touchs,UIEvent * event), RSSWReplacement({
        NSLog(@"test Swizzle");
    }), RSSwizzleModeAlways, @"key");

```

在宏内，可以直接调用RSSWCallOriginal()来执行替换前的原始函数，十分方便。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

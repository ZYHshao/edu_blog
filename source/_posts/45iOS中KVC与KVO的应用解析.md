---
title: iOS中KVC与KVO的应用解析
date: 2015-05-05
categories: iOS逻辑初窥
tags: []
---
## iOS中KVC与KVO的应用解析

### 一、NSKeyValueCoding（KVC）

#### 1、从一个小例子引入

KVC键值编码是Object-C为我们提供的一种对成员变量赋值的方法。在探讨其方法之前，我们先来看一个小例子：

首先，创建一个数据模型model类：

```
//.h文件
#import <Foundation/Foundation.h>
@interface Model : NSObject
{
    @public//将成员变量设置为公有的 以便其他文件有访问权限
    NSString * str;
}
@end
```

我们在其他文件中有两种方法str进行赋值和取值：

```
    Model * model = [[Model alloc]init];
    model->str=@"312";//普通方法赋值
    [model setValue:@"321" forKey:@"str"];//kvc赋值
    NSLog(@"%@",model->str);//普通方法取值
    NSLog(@"%@",[model valueForKey:@"str"]);//kvc取值
```

同样的，对于用@property声明的变量，使用kvc的效果和使用点语法，setter，getter方法的效果是一样的。

#### 2、KVC有关函数方法详解

通过上面的例子，我们已经可以简单了解KVC是干什么的了，下面是一些常用方法。

\+ (BOOL)accessInstanceVariablesDirectly;

这个方法类似一个开关，默认返回为YES，表示支持KVC方式赋值，也可以在子类中将其重写，如果返回为NO，则再进行KVC会抛出异常。

\- (id)valueForKey:(NSString *)key;

通过键取值

\- (void)setValue:(id)value forKey:(NSString *)key;

通过字符串键给成员变量赋值

\- (BOOL)validateValue:(inout id *)ioValue forKey:(NSString *)inKey error:(out NSError **)outError;

系统默认实现的方法，验证一个键值是否有效

\- (NSMutableArray *)mutableArrayValueForKey:(NSString *)key;

将取到的值放入一个可变数组中

\- (NSMutableOrderedSet *)mutableOrderedSetValueForKey:(NSString *)key NS\_AVAILABLE(10\_7, 5_0);

将取到的值放入可变的有序集合中

\- (NSMutableSet *)mutableSetValueForKey:(NSString *)key;

将取到的值放入可变的集合中

\- (id)valueForKeyPath:(NSString *)keyPath;  
\- (void)setValue:(id)value forKeyPath:(NSString *)keyPath;

上面这两个方法分别是通过路径赋值与取值，数据结构类似地图，比如在model类中有一个成员变量model2，在Model2类中有一个字符串，我们可以通过如下的方式赋值取值

```
//Model.h
#import "Model2.h"
@interface Model : NSObject
{
    @public
    NSString * str;
    Model2 * model2;
}
//Model2.h
@interface Model2 : NSObject
{
@public
    NSString * str2;
}
@end
//其他文件
    Model * model = [[Model alloc]init];
    Model2 * model2 = [[Model2 alloc]init];
    model->model2=model2;
    [model setValue:@"123" forKeyPath:@"model2.str2"];
    NSLog(@"%@",[model valueForKeyPath:@"model2.str2"]);
```

\- (NSMutableArray *)mutableArrayValueForKeyPath:(NSString *)keyPath;  
\- (NSMutableOrderedSet *)mutableOrderedSetValueForKeyPath:(NSString *)keyPath NS\_AVAILABLE(10\_7, 5_0);  
\- (NSMutableSet *)mutableSetValueForKeyPath:(NSString *)keyPath;

上面三个方法与前面类似，只是是从路径取值的。

\- (id)valueForUndefinedKey:(NSString *)key;

这个方法可以获取没有提前定义的成员变量的值，比如运行时创建的，下面这个方法是给未定义的成员变量赋值

\- (void)setValue:(id)value forUndefinedKey:(NSString *)key;

注意：这两个方法默认的实现会抛出异常，子类必须重写才能使用。

\- (void)setNilValueForKey:(NSString *)key;

将成员变量置为nil

\- (NSDictionary *)dictionaryWithValuesForKeys:(NSArray *)keys;

根据键值获取键值对字典

\- (void)setValuesForKeysWithDictionary:(NSDictionary *)keyedValues;

通过字典对成员变量同意赋值，经常使用

### 二、NSKeyValueObservingCustomization（KVO）

KVO是一种消息监听机制，可以在某个量发生变化的时候将消息传送给监听者，因此广泛用于传值，界面低耦合等逻辑中。KVO机制的核心是以下三个方法：

\- (void)addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(void *)context;

使用这个方法注册一个监听者，参数解释如下：

observer：监听者对象

keyPath：监听的参数

options：监听选项

context：参数传递

监听的选项枚举如下：

```
typedef NS_OPTIONS(NSUInteger, NSKeyValueObservingOptions) {
    NSKeyValueObservingOptionNew = 0x01,//回调的字典中存放新值
    NSKeyValueObservingOptionOld = 0x02,//回调的字典中存放旧值
    NSKeyValueObservingOptionInitial ,//值改变前进行回调
    NSKeyValueObservingOptionPrior//改变前后都进行回调

};
//回调字典后面会解释
```

\- (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath context:(void *)context ;  
\- (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath;

这两个方法都是用来移除监听者

\- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context;

这个方法是监听对象数据改变时回调的方法，change是一个字典，字典中根据监听的选项不同，存放不同的值（新或者旧）。context是传递的参数。

代码示例：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
     model = [[Model alloc]init];
    //添加监听者
    [model addObserver:self forKeyPath:@"str" options:NSKeyValueObservingOptionNew context:@"321"];
    [model setValue:@"qw" forKey:@"str"];
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context{
    if ([keyPath isEqualToString:@"str"]) {
        NSLog(@"%@",context);
    }
}
```

学习使用 欢迎转载

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: Objective-C中通过下标的方式访问自定义数据模型中属性
date: 2016-03-07
categories: Objective-C浅探
tags: []
---
## Objective-C中通过下标的方式访问自定义数据模型中属性

      在Objective-C中，可以通过下标来访问数组中的元素，如果数组是NSMutableArray类型的可变数组，则还可以通过下标来对数组中的元素进行赋值操作。例如：

```
    NSMutableArray * array = [[NSMutableArray alloc]init];
    array[0] = @"one";
    NSString * str = array[0];
    NSLog(@"%@",str);
```

       对于Objective-C中的字典对象，可以通过键值下标的方式来进行访问，例如：

```
    NSMutableDictionary * dic = [[NSMutableDictionary alloc]init];
    dic[@"name"] = @"name";
    NSLog(@"%@",dic[@"name"]);
```

      对于开发者自定义的的数据结构，一般会采用getter与setter方法来对其属性进行访问，虽然官方文档上没有提及，实际上，可以通过实现一些方法，来使自定义的数据模型支持使用下标来进行访问。

      创建一个数据模型类，使其继承自NSObject，如下：

MyModel.h

```
@interface MyModel : NSObject
@end
```

MyModel.m

```
@implementation MyModel
{
    NSString * _index0;
    NSString * _index1;
    NSString * _value;
}
//通过下标获取属性值
-(id) objectAtIndexedSubscript:(NSUInteger)idx {
    return [self valueForKey:[NSString stringWithFormat:@"_index%lu",idx]];
}
//通过下标设置属性值

- (void)setObject:(id)anObject atIndexedSubscript:(NSUInteger)index{
    [self setValue:anObject forKey:[NSString stringWithFormat:@"_index%lu",index]];
}
//通过键值下标获取属性
-(id) objectForKeyedSubscript:(id)key {
    return [self valueForKey:key];
}
//通过键值下标设置属性
- (void)setObject:(id)object forKeyedSubscript:(id < NSCopying >)aKey{
    [self setValue:object forKey:aKey];
}
@end
```

使用如下代码进行测试：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    id model = [[MyModel alloc]init];
    model[@"_value"] = @"name";
    model[0] = @"one";
    model[1] = @"two";
    NSLog(@"%@,%@,%@",model[0],model[1],model[@"_value"]);
}
```

这里有一点需要注意，若使用下标访问属性这种方法，必须将model声明为id类型，否则会影响编译。

    在打印信息的可以看到，模型数据的设置和获取都没有问题，这种方法可以完全解放.h文件，如上所示，我们在数据模型的.h文件中一行代码都没有编写即可完成与MyModel模型数据的交互。然而其也有很大的弊端，代码的易调试和可读性都大大的降低，因此，没有特殊需求，一般不要使用这种方式来构建模型。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

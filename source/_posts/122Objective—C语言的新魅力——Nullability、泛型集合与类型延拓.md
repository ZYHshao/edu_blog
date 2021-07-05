---
title: Objective—C语言的新魅力——Nullability、泛型集合与类型延拓
date: 2015-10-09
categories: Objective-C浅探
tags: []
---
## Objective—C语言的新魅力

### 一、引言

        在Xcode7中，iOS9的SDK已经全面兼容了Objective-C的一些新特性和新功能。这些功能都只作用于编译期，对程序的运行并没有影响，因此，它可以很好的向下进行兼容，无缝的衔接低版本的iOS系统，那么这些特性有什么样的用处呢，作为开发者，我保证你一定会爱上他们，如果你可以将这些新特性都应用于你的开发，你的开发效率和代码质量，相比之前，会有一个很大的提升。

### 二、Nullability检测的支持

        在swift语言中，通过!和?可以将对象声明成Optional，用于在开发中标记这个对象是否可以为空。在OC中，以前是没有这样的功能的，因此我们在开发中会经常遇到因为某个函数应该返回实例而返回了空导致的崩溃。Nullability的主要用武之地，就是在这里，它可以起到提示开发者做是否为空得判断的提示。

        打开Xcode7，系统的框架中已经支持了Nullability，如下：

```
@property (nullable, nonatomic, readonly) ObjectType firstObject;
@property (nullable, nonatomic, readonly) ObjectType lastObject;
```

这是NSArray中的两个属性，其中nullable关键字说明了这里可能返回空的值。

如果仅仅是在返回值中给开发者一些提示，你可能觉得应用并不大，是的，对开发者最大的帮助是这一特性可以用于函数的参数中，这样我们在调用函数时起到的提示作用，将是非常重要的，越是多人合作的项目，作用也越大。

例如：

```
-(void)setValue:(NSNumber * _Nonnull )number{
    
}
```

我们在调用函数时，如果传入了空值，编译器会给我们警告：

![](http://static.oschina.net/uploads/space/2015/1009/111954_24Wh_2340880.png)

注意：

这一特性在Xcode6.3中就已经支持，但在Xcode7中又做了一些写法上的小改动，例如，在Xcode6.3中这样写：

```
-(void)setValue:( nonnull NSNumber *  )number{
    
}
```

而在Xcode7中提倡我们使用第一种写法。

与之相关的几个关键字如下：

修饰参数

nonnull：不可为空

nullable: 可以为空

null_unspecified:不确定是否可以为空(极少情况)

在属性的声明中，还会有如下一个修饰符：

null_resettable:set方法可以为nil，get方法不可返回nil

一点提示：

你可以发现，iOS9的SDK中已经完全兼容使用了这些特性，并且nonnull的使用会比nullable广泛的多，因此，系统提供了这样一对宏：

#define NS\_ASSUME\_NONNULL\_BEGIN \_Pragma("clang assume_nonnull begin")

#define NS\_ASSUME\_NONNULL\_END   \_Pragma("clang assume_nonnull end")

我们在这对宏之间定义的变量都会加上nonnull的修饰符，只有我们特殊声明nullable的才需要手动写。  
 

### 三、泛型集合的支持

        这一特性和Nullability一样，只作用于编译期，是为我们开发者服务的另一重要特性。还记得，在Xcode7之前，依然是为了方便多人开发，我经常会在框架中写这样的一个空得宏：

![](http://static.oschina.net/uploads/space/2015/1009/115158_v00Q_2340880.png)

在开发时如下使用，做到提示伙伴我这个数组中是什么东西的作用：

```
@interface ViewController ()
{
    NSArray __TYPE__FIT_TO__CLASS(NSString) *  array;
}
@end
```

当然，所有这些都是我自己的自导自演，编译器并不会鸟我，我在这个数组中加其他的东西，它也不会介意，所有这些只是我和我的伙伴们约定的一种一厢情愿。所以，当我看到Xcode7中的集合类型时，我着实兴奋了一下。

#### 1、有类型约定的集合

        在Xcode7中，我们可以给集合类型添加一个泛型的约定，如下：

```
 NSMutableArray<NSString *> *array = [[NSMutableArray alloc]init];
```

声明了这样一个数组后，就好比我告诉了编译器，这个数组中的数据类型都是NSString*类型的，现在非常好，如果我这个数组中元素的方法，会出现如下的提示：

![](http://static.oschina.net/uploads/space/2015/1009/115930_cwIt_2340880.png)

激动吧，使用点语法可以访问到数组中泛型的方法了，还有更加诱人的：

![](http://static.oschina.net/uploads/space/2015/1009/120233_TINB_2340880.png)

在我们向这个数组中追加元素的时候，编译器将元素的类型提示了出来，并且将FromArray方法中需要的元素类型也提示了出来。

同样，如果我们向这个数组中追加类型不匹配的元素，如下：

```
    NSMutableArray<NSString *> *array = [[NSMutableArray alloc]init];
    [array addObject:@1];
```

编译器会给我们一个这样的警告：

![](http://static.oschina.net/uploads/space/2015/1009/135543_fJ9L_2340880.png)

#### 2、关于一个类型通配符

        观察Xcode7中iOS系统的类，我们可以发现这么一个好玩的东西：ObjectType。它既不是一个类型，也不是关键字，然而却大量存在，如下是系统的NSMutableArray的头文件：

```
@interface NSMutableArray<ObjectType> : NSArray<ObjectType>
- (void)addObject:(ObjectType)anObject;
- (void)insertObject:(ObjectType)anObject atIndex:(NSUInteger)index;
- (void)removeLastObject;
- (void)removeObjectAtIndex:(NSUInteger)index;
- (void)replaceObjectAtIndex:(NSUInteger)index withObject:(ObjectType)anObject;
- (instancetype)init NS_DESIGNATED_INITIALIZER;
- (instancetype)initWithCapacity:(NSUInteger)numItems NS_DESIGNATED_INITIALIZER;
- (nullable instancetype)initWithCoder:(NSCoder *)aDecoder NS_DESIGNATED_INITIALIZER;
@end
```

这个ObjectType其实只是一个类型标识符，它具体怎么写并不重要，只是系统中都约定使用了ObjectType，你也可以在自己的类中按自己的喜好来命名，这个东西有怎样的用处，我用文字描述不清楚，我们可以通过自己来定义一个集合类来理解：

创建一个类，继承于NSObject，我取名叫MyArray：

```
//这个类型通配符只能在interfave里使用，作用域为@interface到@end之间
//这里我使用Type来做这个通配符
@interface MyArray<Type> : NSObject
@property(nonatomic,strong,nonnull)NSMutableArray<Type> *array;
-(void)addObject:(nonnull Type)obj;
@end
```

实现如下：

```
- (instancetype)init
{
    self = [super init];
    if (self) {
        _array = [[NSMutableArray alloc]init];
    }
    return self;
}
-(void)addObject:(id)obj{
    [_array addObject:obj];
}
-(NSString *)description{
    NSMutableString * str = [[NSMutableString alloc]init];
    for (int i=0; i<_array.count; i++) {
        [str appendString:[NSString stringWithFormat:@"%@\n",_array[i]]];
    }
    return str;
}
```

我们在使用这个自定义的集合类型时，就会有和系统一样的效果了：

![](http://static.oschina.net/uploads/space/2015/1009/142215_YVEn_2340880.png)

#### 3、关于多参数的泛型集合

        多参数的泛型集合，有一个非常好的例子，就是NSDictionary，在Xcode7中我们可以这样写字典：

![](http://static.oschina.net/uploads/space/2015/1009/142650_Pp0U_2340880.png)

可以看到，字典键值的类型编译器为我们提示了出来，结合上面类型通配符的使用，对于多参的集合，将参数类型用“,”隔开即可。

#### 4、协变性与逆变性

        因为有了泛型集合的概念，相比之前，我们的类型实际上更加复杂了，比如还拿我们自定义的集合类型来举例：

```
    MyArray<NSString *> * array;
    MyArray<NSMutableString *>*muArray;
```

array和muArray在编译器看来已经是不同的类型，如果我们强行转换，会报如下的警告：

![](http://static.oschina.net/uploads/space/2015/1009/143910_eIfB_2340880.png)

因此，就有了逆变和协变这个概念：

__covariant :子类型指针可以向父类型指针转换

__contravariant:父类型指针可以向子类型转换

上面的情况，我们将自定义的类做如下修改，就不会出现警告：

```
@interface MyArray<__covariant Type> : NSObject
@property(nonatomic,strong,nonnull)NSMutableArray<Type> *array;
-(void)addObject:(nonnull Type)obj;
@end
```

### 四、类型延拓符的应用

        在开发中，开发者经常会遇到这样的情况，例如通过tag获取某些UI控件时，viewWithTag方法通常会返回给我们一个UIView类型的指针，这就需要开发者手动的强转一下，十分麻烦。新增加的__kindof修饰符可以帮助我们解除这个烦恼。我们还从自定义的那个数组类开刀，对其添加一个属性：

```
@interface MyArray<__covariant Type> : NSObject
@property(nonatomic,strong,nonnull)NSMutableArray<Type> *array;
@property(nonnull,strong,nonatomic)NSMutableArray<UIView *> * viewArray;
-(void)addObject:(nonnull Type)obj;
@end
```

创建一个自定义的数组对象，并向其中添加一个UIButton，我们会看到有如下一个警告：

![](http://static.oschina.net/uploads/space/2015/1009/145521_aJLa_2340880.png)

这也是我们开发中常遇到的问题，对吧，以前需要强转。但是以后就不需要了，我们在声明这个数组时加上一个__kindof修饰符：

```
@property(nonnull,strong,nonatomic)NSMutableArray<__kindof UIView *> * viewArray;
```

警告就消失了，很cool吧。

这个修饰符就是告诉编译器，这里可以返回UIView的子类指针。

### 五、结语

         虽然这些优点在swift中早有体现，但就我个人而言，我对OC的感情会更深一些，也更加愿意接受OC的改变和成长，大家都说swift的趋势势在必行，我只想说，swift很优秀，OC亦然。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

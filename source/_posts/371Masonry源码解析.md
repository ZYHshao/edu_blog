---
title: Masonry源码解析
date: 2018-04-20
categories: iOS第三方库
tags: []
---
## Masonry源码解析

    Masonry的核心依然是使用原生的NSLayoutConstraint类来进行添加约束，通过统一的封装和链式函数式编程的方式让开发者添加约束布局更加方便。

### 一、核心的View+MASAdditions类别

    这个类别是Masonry中用来添加，更新和重置约束的核心类别。其中提供了我们最常用的布局函数。首先从类别命名上也可以看出，此类别扩展的类是通过宏来设置的：

```objectivec
@interface MAS_VIEW (MASAdditions)

```

MAS_VIEW宏做到了平台屏蔽的作用，在iOS上，其为UIView，在MacOS上其实NSView。

    MASAdditions类别中定义了许多布局属性，例如上，下，左，右边距，宽度高度等等。这些属性被抽象为MASViewAttribute对象，关于这个对象，后面会具体介绍。

```objectivec
//左
@property (nonatomic, strong, readonly) MASViewAttribute *mas_left;
//上
@property (nonatomic, strong, readonly) MASViewAttribute *mas_top;
//右
@property (nonatomic, strong, readonly) MASViewAttribute *mas_right;
//下
@property (nonatomic, strong, readonly) MASViewAttribute *mas_bottom;
//前
@property (nonatomic, strong, readonly) MASViewAttribute *mas_leading;
//后
@property (nonatomic, strong, readonly) MASViewAttribute *mas_trailing;
//宽
@property (nonatomic, strong, readonly) MASViewAttribute *mas_width;
//高
@property (nonatomic, strong, readonly) MASViewAttribute *mas_height;
//水平中心
@property (nonatomic, strong, readonly) MASViewAttribute *mas_centerX;
//垂直中心
@property (nonatomic, strong, readonly) MASViewAttribute *mas_centerY;
//基线
@property (nonatomic, strong, readonly) MASViewAttribute *mas_baseline;
//这个是一个链式编程的通用转换方法，使用这个属性将系统的NSLayoutAttribute转换成抽象的MASViewAttribute对象
@property (nonatomic, strong, readonly) MASViewAttribute *(^mas_attribute)(NSLayoutAttribute attr);

//基线相关
@property (nonatomic, strong, readonly) MASViewAttribute *mas_firstBaseline;
@property (nonatomic, strong, readonly) MASViewAttribute *mas_lastBaseline;

//安全区 相关
@property (nonatomic, strong, readonly) MASViewAttribute *mas_safeAreaLayoutGuide API_AVAILABLE(ios(11.0),tvos(11.0));
@property (nonatomic, strong, readonly) MASViewAttribute *mas_safeAreaLayoutGuideTop API_AVAILABLE(ios(11.0),tvos(11.0));
@property (nonatomic, strong, readonly) MASViewAttribute *mas_safeAreaLayoutGuideBottom API_AVAILABLE(ios(11.0),tvos(11.0));
@property (nonatomic, strong, readonly) MASViewAttribute *mas_safeAreaLayoutGuideLeft API_AVAILABLE(ios(11.0),tvos(11.0));
@property (nonatomic, strong, readonly) MASViewAttribute *mas_safeAreaLayoutGuideRight API_AVAILABLE(ios(11.0),tvos(11.0));

//关联的key值
@property (nonatomic, strong) id mas_key;
```

下面是3个最常使用的布局方法：

```objectivec
//创建约束
- (NSArray *)mas_makeConstraints:(void(NS_NOESCAPE ^)(MASConstraintMaker *make))block;
//更新约束
- (NSArray *)mas_updateConstraints:(void(NS_NOESCAPE ^)(MASConstraintMaker *make))block;
//重新创建约束
- (NSArray *)mas_remakeConstraints:(void(NS_NOESCAPE ^)(MASConstraintMaker *make))block;
```

这3个函数的具体实现基本一致，其核心流程都是：关闭视图Autoresizing特性->创建约束生成器->配置约束生成器->回调开发者约束设置->进行约束加载。这3个函数不同的地方只在配置约束生成器部分，配置了updateExisting参数为YES，表示要进行已有约束的更新，配置了removeExisting为YES表示要重新创建约束。约束生成器被抽象为MASConstraintMaker对象，下面来具体看这个类。

### 二、MASConstraintMaker约束生成器

    MASConstraint类主要用来构建约束对象。其中虽然和MASAdditions扩展类似，也是定义了约束属性对象，但是其所有的Get方法都被重新实现了，当我们通过Get方法调用约束属性时，会执行下面核心函数：

```objectivec
- (MASConstraint *)constraint:(MASConstraint *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {
    //创建属性
    MASViewAttribute *viewAttribute = [[MASViewAttribute alloc] initWithView:self.view layoutAttribute:layoutAttribute];
    //创建约束对象
    MASViewConstraint *newConstraint = [[MASViewConstraint alloc] initWithFirstViewAttribute:viewAttribute];
    if ([constraint isKindOfClass:MASViewConstraint.class]) {
        //进行复合
        //replace with composite constraint
        NSArray *children = @[constraint, newConstraint];
        MASCompositeConstraint *compositeConstraint = [[MASCompositeConstraint alloc] initWithChildren:children];
        compositeConstraint.delegate = self;
        [self constraint:constraint shouldBeReplacedWithConstraint:compositeConstraint];
        return compositeConstraint;
    }
    if (!constraint) {
        newConstraint.delegate = self;
        [self.constraints addObject:newConstraint];
    }
    //将约束对象返回
    return newConstraint;
}
```

上面函数的设计可以巧妙的实现复合约束，例如make.width.height.equalTo(@100)这样一条约束，实际上从width开始后面的属性都被复合进了MASCompositeConstraint对象。约束的属性创建出来后，需要对其进行值的设置，下面来看MASViewConstraint对象。

### 三、MASConstraint约束对象

    MASViewConstraint类继承自MASConstraint类，MASConstraint类还有一个子类为MASCompositeConstraint类。MASConstraint中定义了基础的约束值设置方法，都是采用block回调的方式，因此可以进行链式编程：

```objectivec
//位置
- (MASConstraint * (^)(MASEdgeInsets insets))insets;
//尺寸偏移
- (MASConstraint * (^)(CGSize offset))sizeOffset;
//中心位置偏移
- (MASConstraint * (^)(CGPoint offset))centerOffset;
//比例 *
- (MASConstraint * (^)(CGFloat multiplier))multipliedBy;
//比例 /
- (MASConstraint * (^)(CGFloat divider))dividedBy;
//优先级
- (MASConstraint * (^)(MASLayoutPriority priority))priority;
//直接设置为低优先级
- (MASConstraint * (^)(void))priorityLow;
//直接设置为中优先级
- (MASConstraint * (^)(void))priorityMedium;
//直接设置为高优先级
- (MASConstraint * (^)(void))priorityHigh;
//设置绝对等于
- (MASConstraint * (^)(id attr))equalTo;
//大于等于
- (MASConstraint * (^)(id attr))greaterThanOrEqualTo;
//小于等于
- (MASConstraint * (^)(id attr))lessThanOrEqualTo;
```

阅读这个属性的Get方法，你会发现他们最后都返回了当前对象本身，这也是为链式编程所准备，MASConstraint中还有两个属性比较有趣：

```objectivec
- (MASConstraint *)with;
- (MASConstraint *)and;
```

这两个属性没有实际的作用也没有任何影响，他们的实现是直接返回当前对象，增强代码可读性。

    MASConstraint类中的install和uninstall函数是核心的约束添加方法，其中会进行系统原生约束对象的转换添加或者删除操作。核心的install函数解析如下：

```objectivec
- (void)install {
    //如果已经被加载 直接返回
    if (self.hasBeenInstalled) {
        return;
    }
    //如果系统layout对象已经创建 直接添加之后 返回
    if ([self supportsActiveProperty] && self.layoutConstraint) {
        self.layoutConstraint.active = YES;
        [self.firstViewAttribute.view.mas_installedConstraints addObject:self];
        return;
    }
    //获取布局的视图与属性
    MAS_VIEW *firstLayoutItem = self.firstViewAttribute.item;
    NSLayoutAttribute firstLayoutAttribute = self.firstViewAttribute.layoutAttribute;
    MAS_VIEW *secondLayoutItem = self.secondViewAttribute.item;
    NSLayoutAttribute secondLayoutAttribute = self.secondViewAttribute.layoutAttribute;
    //如果不是尺寸布局并且 相对视图不存在 默认对父视图进行相对布局
    if (!self.firstViewAttribute.isSizeAttribute && !self.secondViewAttribute) {
        secondLayoutItem = self.firstViewAttribute.view.superview;
        secondLayoutAttribute = firstLayoutAttribute;
    }
    //创建布局对象
    MASLayoutConstraint *layoutConstraint
        = [MASLayoutConstraint constraintWithItem:firstLayoutItem
                                        attribute:firstLayoutAttribute
                                        relatedBy:self.layoutRelation
                                           toItem:secondLayoutItem
                                        attribute:secondLayoutAttribute
                                       multiplier:self.layoutMultiplier
                                         constant:self.layoutConstant];
    //设置key和优先级
    layoutConstraint.priority = self.layoutPriority;
    layoutConstraint.mas_key = self.mas_key;
    //设置约束对象对用于的视图
    if (self.secondViewAttribute.view) {
        //获取共同的父视图
        MAS_VIEW *closestCommonSuperview = [self.firstViewAttribute.view mas_closestCommonSuperview:self.secondViewAttribute.view];
        NSAssert(closestCommonSuperview,
                 @"couldn't find a common superview for %@ and %@",
                 self.firstViewAttribute.view, self.secondViewAttribute.view);
        self.installedView = closestCommonSuperview;
    } else if (self.firstViewAttribute.isSizeAttribute) {
        self.installedView = self.firstViewAttribute.view;
    } else {
        self.installedView = self.firstViewAttribute.view.superview;
    }


    MASLayoutConstraint *existingConstraint = nil;
    //更新约束的操作
    if (self.updateExisting) {
        existingConstraint = [self layoutConstraintSimilarTo:layoutConstraint];
    } 
    if (existingConstraint) {
        // just update the constant
        existingConstraint.constant = layoutConstraint.constant;
        self.layoutConstraint = existingConstraint;
    } else {
        //添加约束
        [self.installedView addConstraint:layoutConstraint];
        self.layoutConstraint = layoutConstraint;
        [firstLayoutItem.mas_installedConstraints addObject:self];
    }
}
```

### 四、一个小技巧

    Masonry中的一个函数值得我们学习，其作用是对任何类型的值进行一层对象包装，函数如下：

```objectivec
//值包装宏
#define MASBoxValue(value) _MASBoxValue(@encode(__typeof__((value))), (value))

static inline id _MASBoxValue(const char *type, ...) {
    va_list v;
    va_start(v, type);
    id obj = nil;
    //进行类型判断 结构体包装成NSValue 基本类型包装成NSNumber
    if (strcmp(type, @encode(id)) == 0) {
        id actual = va_arg(v, id);
        obj = actual;
    } else if (strcmp(type, @encode(CGPoint)) == 0) {
        CGPoint actual = (CGPoint)va_arg(v, CGPoint);
        obj = [NSValue value:&actual withObjCType:type];
    } else if (strcmp(type, @encode(CGSize)) == 0) {
        CGSize actual = (CGSize)va_arg(v, CGSize);
        obj = [NSValue value:&actual withObjCType:type];
    } else if (strcmp(type, @encode(MASEdgeInsets)) == 0) {
        MASEdgeInsets actual = (MASEdgeInsets)va_arg(v, MASEdgeInsets);
        obj = [NSValue value:&actual withObjCType:type];
    } else if (strcmp(type, @encode(double)) == 0) {
        double actual = (double)va_arg(v, double);
        obj = [NSNumber numberWithDouble:actual];
    } else if (strcmp(type, @encode(float)) == 0) {
        float actual = (float)va_arg(v, double);
        obj = [NSNumber numberWithFloat:actual];
    } else if (strcmp(type, @encode(int)) == 0) {
        int actual = (int)va_arg(v, int);
        obj = [NSNumber numberWithInt:actual];
    } else if (strcmp(type, @encode(long)) == 0) {
        long actual = (long)va_arg(v, long);
        obj = [NSNumber numberWithLong:actual];
    } else if (strcmp(type, @encode(long long)) == 0) {
        long long actual = (long long)va_arg(v, long long);
        obj = [NSNumber numberWithLongLong:actual];
    } else if (strcmp(type, @encode(short)) == 0) {
        short actual = (short)va_arg(v, int);
        obj = [NSNumber numberWithShort:actual];
    } else if (strcmp(type, @encode(char)) == 0) {
        char actual = (char)va_arg(v, int);
        obj = [NSNumber numberWithChar:actual];
    } else if (strcmp(type, @encode(bool)) == 0) {
        bool actual = (bool)va_arg(v, int);
        obj = [NSNumber numberWithBool:actual];
    } else if (strcmp(type, @encode(unsigned char)) == 0) {
        unsigned char actual = (unsigned char)va_arg(v, unsigned int);
        obj = [NSNumber numberWithUnsignedChar:actual];
    } else if (strcmp(type, @encode(unsigned int)) == 0) {
        unsigned int actual = (unsigned int)va_arg(v, unsigned int);
        obj = [NSNumber numberWithUnsignedInt:actual];
    } else if (strcmp(type, @encode(unsigned long)) == 0) {
        unsigned long actual = (unsigned long)va_arg(v, unsigned long);
        obj = [NSNumber numberWithUnsignedLong:actual];
    } else if (strcmp(type, @encode(unsigned long long)) == 0) {
        unsigned long long actual = (unsigned long long)va_arg(v, unsigned long long);
        obj = [NSNumber numberWithUnsignedLongLong:actual];
    } else if (strcmp(type, @encode(unsigned short)) == 0) {
        unsigned short actual = (unsigned short)va_arg(v, unsigned int);
        obj = [NSNumber numberWithUnsignedShort:actual];
    }
    va_end(v);
    return obj;
}


```

其中@encode()是一个编译时特性，其可以将传入的类型转换为标准的OC类型字符串。

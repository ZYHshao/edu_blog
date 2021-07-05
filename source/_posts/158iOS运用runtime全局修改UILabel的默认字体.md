---
title: iOS运用runtime全局修改UILabel的默认字体
date: 2015-12-02
categories: iOS逻辑初窥
tags: []
---
## iOS运用runtime全局修改UILabel的默认字体

### 一、需求背景介绍

        在项目比较成熟的基础上，遇到了这样一个需求，应用中需要引入新的字体，需要更换所有Label的默认字体，但是同时，对于一些特殊设置了字体的label又不需要更换。乍看起来，这个问题确实十分棘手，首先项目比较大，一个一个设置所有使用到的label的font工作量是巨大的，并且在许多动态展示的界面中，可能会漏掉一些label，产生bug。其次，项目中的label来源并不唯一，有用代码创建的，有xib和storyBoard中的，这也将浪费很大的精力。这种情况下，我们可能会有下面两种处理方式。

### 二、处理方式

#### 1、使用框架

        创建我们自己的BaseLabel类，在其中进行默认字体的设置，并且并不影响在使用过程中特殊设置字体的label，这种方式可以满足我们的需求，但是并不适于我们的场景，项目已经成熟，重建一个label基类，来让所有的UILabel都换成它的工作量不会比重新设置所有label字体的工作量小太多。但这也是有优势的，至少如果下次再换字体，我们就不用麻烦了。

#### 2、使用runtime替换UILabel初始化方法

        这是最简单方便的方法，我们可以使用runtime机制替换掉UILabel的初始化方法，在其中对label的字体进行默认设置。因为Label可以从initWithFrame、init和nib文件三个来源初始化，所以我们需要将这三个初始化的方法都替换掉。

首先，我们创建一个UILabel的类别：

```
#import <UIKit/UIKit.h>

@interface UILabel (YHBaseChangeDefaultFont)

@end
```

在其中加入如下代码：

```
#import "UILabel+YHBaseChangeDefaultFont.h"
#import <objc/runtime.h>
@implementation UILabel (YHBaseChangeDefaultFont)
/**
 *每个NSObject的子类都会调用下面这个方法 在这里将init方法进行替换，使用我们的新字体
 *如果在程序中又特殊设置了字体 则特殊设置的字体不会受影响 但是不要在Label的init方法中设置字体
 *从init和initWithFrame和nib文件的加载方法 都支持更换默认字体
 */
+(void)load{
    //只执行一次这个方法
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];
        // When swizzling a class method, use the following:
        // Class class = object_getClass((id)self);
        //替换三个方法
        SEL originalSelector = @selector(init);
        SEL originalSelector2 = @selector(initWithFrame:);
        SEL originalSelector3 = @selector(awakeFromNib);
        SEL swizzledSelector = @selector(YHBaseInit);
        SEL swizzledSelector2 = @selector(YHBaseInitWithFrame:);
        SEL swizzledSelector3 = @selector(YHBaseAwakeFromNib);
        
        
        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method originalMethod2 = class_getInstanceMethod(class, originalSelector2);
        Method originalMethod3 = class_getInstanceMethod(class, originalSelector3);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
        Method swizzledMethod2 = class_getInstanceMethod(class, swizzledSelector2);
        Method swizzledMethod3 = class_getInstanceMethod(class, swizzledSelector3);
        BOOL didAddMethod =
        class_addMethod(class,
                        originalSelector,
                        method_getImplementation(swizzledMethod),
                        method_getTypeEncoding(swizzledMethod));
        BOOL didAddMethod2 =
        class_addMethod(class,
                        originalSelector2,
                        method_getImplementation(swizzledMethod2),
                        method_getTypeEncoding(swizzledMethod2));
        BOOL didAddMethod3 =
        class_addMethod(class,
                        originalSelector3,
                        method_getImplementation(swizzledMethod3),
                        method_getTypeEncoding(swizzledMethod3));
        
        if (didAddMethod) {
            class_replaceMethod(class,
                                swizzledSelector,
                                method_getImplementation(originalMethod),
                                method_getTypeEncoding(originalMethod));
           
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
        if (didAddMethod2) {
            class_replaceMethod(class,
                                swizzledSelector2,
                                method_getImplementation(originalMethod2),
                                method_getTypeEncoding(originalMethod2));
        }else {
            method_exchangeImplementations(originalMethod2, swizzledMethod2);
        }
        if (didAddMethod3) {
            class_replaceMethod(class,
                                swizzledSelector3,
                                method_getImplementation(originalMethod3),
                                method_getTypeEncoding(originalMethod3));
        }else {
            method_exchangeImplementations(originalMethod3, swizzledMethod3);
        }
    });
    
}
/**
 *在这些方法中将你的字体名字换进去
 */
- (instancetype)YHBaseInit
{
    id __self = [self YHBaseInit];
    UIFont * font = [UIFont fontWithName:@"这里输入你的字体名字" size:self.font.pointSize];
    if (font) {
        self.font=font;
    }
    return __self;
}

-(instancetype)YHBaseInitWithFrame:(CGRect)rect{
    id __self = [self YHBaseInitWithFrame:rect];
    UIFont * font = [UIFont fontWithName:@"这里输入你的字体名字" size:self.font.pointSize];
    if (font) {
        self.font=font;
    }
    return __self;
}
-(void)YHBaseAwakeFromNib{
    [self YHBaseAwakeFromNib];
    UIFont * font = [UIFont fontWithName:@"这里输入你的字体名字" size:self.font.pointSize];
    if (font) {
        self.font=font;
    }
    
}

@end
```

在上面的方法中写入我们想要UILabel默认显示的字体，我们分别从init，initWithFrame和nib文件创建一个UILabel添加到视图上，不做任何其他的操作：

```
UILabel * label = [[UILabel alloc]initWithFrame:CGRectMake(20, 100, 280, 30)];
    label.text = @"你是从initWithFrame来的label";
    UILabel * label2 = [[UILabel alloc]init];
    label2.frame= CGRectMake(20, 200, 280, 30);
    label2.text = @"你是从init来的label";
    [self.view addSubview:label];
    [self.view addSubview:label2];
```

![](http://static.oschina.net/uploads/space/2015/1202/145711_OTN6_2340880.png)

运行效果如下，可以看出，字体全部换掉了：

![](http://static.oschina.net/uploads/space/2015/1202/145930_bU46_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

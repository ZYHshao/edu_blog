---
title: iOS开发CoreAnimation解读之五——高级动画技巧
date: 2015-12-06
categories: iOS逻辑初窥
tags: []
---
## iOS开发CoreAnimation解读之五——高级动画技巧

### 一、事务类

        CoreAnimation中还有一个非常重要的类：CATransaction事物类，这个可以同时设置多个layer层的动画效果。可以通过隐式和显式两种方式来进行动画操作。

### 二、CATransaction属性

        对layer层的属性操作，都会形成隐式动画，要使用隐式动画，需要关闭layer层的animation动画属性，使用下面的方法：

```
//关闭animation动画效果，开启隐式动画
+ (BOOL)disableActions;
+ (void)setDisableActions:(BOOL)flag;
```

CATransaction用类方式通过设置key-value来进行动画的属性设置：

```
+ (nullable id)valueForKey:(NSString *)key;
+ (void)setValue:(nullable id)anObject forKey:(NSString *)key;
```

支持的key值如下：

```
//设置动画持续时间
 NSString * const kCATransactionAnimationDuration;
 //设置停用animation类动画
 NSString * const kCATransactionDisableActions;
 //设置动画时序效果
 NSString * const kCATransactionAnimationTimingFunction;
 //设置动画完成后的回调
 NSString * const kCATransactionCompletionBlock;
```

除了隐式的展示动画外，也可以显式的通过调用CATransaction的相关方法进行显示的提交动画：

```
//动画开始
+ (void)begin;
//提交动画
+ (void)commit;
//立即进行动画渲染 一般不需调用
+ (void)flush;
//下面这两个方法用于动画事物的加锁与解锁 在多线程动画中，保证修改属性的安全
+ (void)lock;
+ (void)unlock;
```

示例如下：

```
    [CATransaction begin];
    [CATransaction setValue:@1 forKey:kCATransactionAnimationDuration];
    layer.backgroundColor = [UIColor blueColor].CGColor;
    [CATransaction commit];
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

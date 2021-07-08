---
title: iOS中动态更新补丁策略JSPatch运用基础二
date: 2016-03-30
categories: JSPatch
tags: []
---
## iOS中动态更新补丁策略JSPatch运用基础二

### 一、引言

    上篇博客中介绍了iOS开发中JSPatch引擎进行动态热修复的一些基础功能，其中包括向Objective-C类中添加类方法与成员方法、添加临时成员变量，使用JavaScript调用原生的Objective-C属性和方法等。本篇博客将基于上一篇继续介绍Objective-C中的一些特殊数据类型在JavaScript文件中的使用方法，博客中大部分内容扩展自JSPatch开源git的wiki：[https://github.com/bang590/JSPatch](https://github.com/bang590/JSPatch)。

iOS中动态更新补丁策略JSPatch运用基础一：[http://my.oschina.net/u/2340880/blog/646688](http://my.oschina.net/u/2340880/blog/646688)。

### 二、JavaScript与Objective-C交互的几种常用类型

#### 1.结构体 

    在Objective-C代码中，我们经常会使用到结构体，JSPatch中原生支持的结构体有如下几种：CGPoint，CGSize，CGRect，NSRange。并且这几种结构体在进行界面操作时也会经常使用到。

    对于CGRect类型，JavaScript使用如下代码创建：

```
    var view = require('UIView').alloc().init()
    view.setFrame({x:100,y:100,width:100,height:100})
```

    对于CGPoint类型，JavaScript使用如下代码创建：

```
    view.setCenter({x:200,y:200})
```

    对于CGSize类型，JavaScript使用如下代码创建：

```
    var size = {width:200,height:200}
    view.setFrame({x:100,y:100,width:size.width,height:size.height})
```

    对于NSRange类型，JavaScript使用如下代码创建：

```
    var range = {location: 0, length: 1}
```

#### 2.选择器Selector

    对于Objective-C中的方法选择器Selector，在JavaScript中使用字符串的形式创建，例如：

```
self.performSelector_withObject("func:", 1)
```

#### 3.关于空对象

    在JavaScript中，null与undefined都对应于Objective-C中的nil，Objective-C中的NSNull空对象，在JavaScript中使用nsnull来代替。

#### 4.在Objective-C与JavaScript中进行block的交互

     在JavaScript与Objective-C进行block交互有两种方式，一种是在JavaScript文件中调用Objective-C中的block，一种是将JavaScript文件中的函数块作为block参数传递给Objective-C。

    在JavaScript文件中使用Objective-C中的block十分简单，因为JavaScript中没有block的概念，Objective-C会被自动转换为函数，示例如下：

Objective-C：

```
typedef void(^block)(NSString * str);
@interface ViewController ()
@end
@implementation ViewController
-(block)getBlock{
    block  block = ^(NSString * str){NSLog(@"%@",str);};
    return block;
}
@end
```

JavaScript：

```
defineClass("ViewController", {
            viewDidAppear: function(animated) {
             var func = self.getBlock()
                func("123")
            }
            })
```

    在JavaScript文件中将func作为参数block传递给Objective-C就复杂一些，需要使用block()方法进行包装，例如：

Objective-C：

```
@interface ViewController ()
@end
@implementation ViewController

-(void)run:(void(^)(NSString * str))block{
    block(@"123");
}
@end
```

JavaScript：

```
defineClass("ViewController", {
            viewDidAppear: function(animated) {
            //run 方法中需要传入一个block
            self.run(block("NSString*",function(str){console.log(str)}))
            }
            })
```

在使用block()方法对JavaScript中的Func进行包装时，block(param1,param2)有两个参数，第1个参数设置func中的参数类型，如果有多个参数，使用逗号分割；第2个参数为func函数体。

注意：在block()包装的func中不可以使用self指针，如果需要使用self，需要在block外进行临时变量的转换，示例如下：

```
defineClass("ViewController", {
            viewDidAppear: function(animated) {
            //run 方法中需要传入一个block
            var slf = self
            self.run(block("NSString*",
                           function(str){
                           console.log(str)
                           slf.log(str)
                           }))
            }
            })
```

    在JavaScript中分别使用\_\_weak()与\_\_strong来声明弱引用与强引用对象，例如：

```
  var slf = __weak(self)
  var stgSef = __strong(self)
```

#### 5.关于GCD与枚举

    在JSPatch中，可以使用如下JavaScript代码来调用GCD方法：

```
//阻塞当前线程一定时间
dispatch_after(1.0, function(){ 
})
//为主线程添加异步任务
dispatch_async_main(function(){  
})
//为主线程添加同步任务
dispatch_sync_main(function(){  
})
//向全局队列中添加任务
dispatch_async_global_queue(function(){  
})
```

    JSPatch中不可以直接使用Objective-C中定义的枚举，但是可以用其枚举的真实值进行传递。例如：

```
//UIControlEventTouchUpInside的值是1<<6
btn.addTarget_action_forControlEvents(self, "handleBtn", 1<<6);
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

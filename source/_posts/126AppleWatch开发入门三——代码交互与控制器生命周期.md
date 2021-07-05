---
title: AppleWatch开发入门三——代码交互与控制器生命周期
date: 2015-10-14
categories: Apple Watch开发手记
tags: []
---
## AppleWatch开发入门三——代码交互与控制器生命周期

### 一、引言

        在前两篇博客中，讨论了关于watch开发中框架与界面布局相关，然而主要的逻辑，终究还是要通过代码来实现的，在我们创建了项目之后，就会生成InterfaceController这个文件，它就是我们storyBoard中的入口视图控制器。

### 二、代码交互与控制器声明周期

        storyBoard中的控件我们可以通过拖拽的方式关联到文件中，Action和Outlet两种关联方式基本可以达到我们修改控件和处理业务逻辑的需求。

        WKInterfaceController类似于iOS中的ViewController，是watch中主要用于展示界面的controller，我们的控件也都是基于这个容器中显示。在模板中，系统为我们提供了三个函数，这三个函数体现了watch一个界面的声明周期，如下：

```
    //这个函数在初始化界面时会触发，通过context可以实现界面的传值
    override func awakeWithContext(context: AnyObject?) {
        super.awakeWithContext(context)
    
    }
    //这个函数在界面即将展现时触发 类似于iOS中的ViewWillApear
    override func willActivate() {
        // This method is called when watch view controller is about to be visible to user
        super.willActivate()
    }
    //这个函数在界面消失后触发，类似于iOS中的ViewDidDisAppear
    override func didDeactivate() {
        // This method is called when watch view controller is no longer visible
        super.didDeactivate()
    }
```

### 三、watch中的界面跳转与传值

        与iOS类似，watchOS的界面跳转也有两种方式：model和push。同样，我们也可以通过storyBoard或者代码来进行跳转。

#### 1、通过代码跳转与传值

        我们创建两个InterfaceController，界面如下：

![](http://static.oschina.net/uploads/space/2015/1014/171755_SufM_2340880.png)

通过代码跳转，我们需要给第二个controller设置一个id标识符：

![](http://static.oschina.net/uploads/space/2015/1014/171930_Lude_2340880.png)

在按钮触发的方法中，如下跳转：

```
 @IBAction func `switch`(value: Bool) {
         //这里的context是传值的上下文
         //在awakeWithContext方法中会将这个值取到
        pushControllerWithName("InterfaceControllerTwo", context: "我是传的值")
    }
```

#### 2、在storyBoard中设置跳转关系

        我们也可以直接在storyBoard中设置界面的跳转，按住control，拖拽按钮到要跳转的controller，会出现push和model菜单，选择后，当我们触发按钮方法时，就会跳转：

![](http://static.oschina.net/uploads/space/2015/1014/172351_njzr_2340880.png)

通过这种方式进行的跳转，在执行跳转之前，会执行如下这个函数：

```
override func contextForSegueWithIdentifier(segueIdentifier: String) -> AnyObject? {
        return "我是值"
    }
```

这个设置的返回值就是context上下文传递的值。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

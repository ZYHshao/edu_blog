---
title: iOS中动态更新补丁策略JSPatch运用基础一
date: 2016-03-24
categories: JSPatch
tags: []
---
## iOS中动态更新补丁策略JSPatch运用基础

        JSPatch是GitHub上一个开源的框架，其可以通过Objective-C的run-time机制动态的使用JavaScript调用与替换项目中的Objective-C属性与方法。其框架小巧，代码简洁，并且通过系统的JavaScriptCore框架与Objective-C进行交互，这使其在安全性和审核风险上都有很强的优势。Git源码地址：[https://github.com/bang590/JSPatch](https://github.com/bang590/JSPatch)。

### 一、从一个官方的小demo看起

        通过cocoapods将JSPath集成进一个Xcode工程中，在AppDelegate类的中编写如下代码：

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    //开始初始化引擎
    [JPEngine startEngine];
    //读取js文件
    NSString *sourcePath = [[NSBundle mainBundle] pathForResource:@"demo" ofType:@"js"];
    NSString *script = [NSString stringWithContentsOfFile:sourcePath encoding:NSUTF8StringEncoding error:nil];
    //运行js文件
    [JPEngine evaluateScript:script];
    self.window = [[UIWindow alloc]initWithFrame:[UIScreen mainScreen].bounds];
    self.window.rootViewController = [[ViewController alloc]init];
    [self.window addSubview:[self genView]];
    [self.window makeKeyAndVisible];
    return YES;
}

- (UIView *)genView
{
    UIView * view= [[UIView alloc] initWithFrame:CGRectMake(0, 0, 320, 320)];
    view.backgroundColor = [UIColor redColor];
    return view;
}
```

在工程中添加一个js文件，编写如下：

```
    require('UIView, UIColor, UILabel')
    //要替换函数的类
    defineClass('AppDelegate', {
            //替换函数
                //要替换函数的名称
                genView: function() {
                    var view = self.ORIGgenView();
                    view.setBackgroundColor(UIColor.greenColor())
                    var label = UILabel.alloc().initWithFrame(view.frame());
                    label.setText("JSPatch");
                    label.setTextAlignment(1);
                    view.addSubview(label);
                    return view;
            }
    });
```

运行工程，可以看到genView方法被替换成了js文件中的方法，原本红色的视图被修改成了绿色。

### 二、使用JavaScript代码向Objective-C中修改或添加方法

        JSPatch引擎中支持3中方式进行JavaScript代码的调用，分别是使用JavaScript字符串进行代码运行，读取本地的JavaScript文件进行代码运行和获取网络的JavaScript文件进行代码运行。例如，如果想要通过JavaScript代码在项目中弹出一个警告框，在Objective-C代码中插入如下代码：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // ‘\’符用于进行换行
    [JPEngine evaluateScript:@"\
     var alertView = require('UIAlertView').alloc().init();\
     alertView.setTitle('Alert');\
     alertView.setMessage('AlertView from js'); \
     alertView.addButtonWithTitle('OK');\
     alertView.show(); \
     "];
}
```

        开发者也可以动态在Objective-C类文件中添加方法，例如在ViewController类中编写如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor whiteColor];
    [JPEngine startEngine];
    NSString *sourcePath = [[NSBundle mainBundle] pathForResource:@"demo" ofType:@"js"];
    NSString *script = [NSString stringWithContentsOfFile:sourcePath encoding:NSUTF8StringEncoding error:nil];
    [JPEngine evaluateScript:script];
    [self performSelectorOnMainThread:@selector(creatView) withObject:nil waitUntilDone:nil];
}
```

JavaScript文件代码如下：

```
 require('UIView, UIColor, UILabel')
    defineClass('ViewController', {
            // replace the -genView method
                creatView: function() {
                    var view = UIView.alloc().initWithFrame({x:20, y:20, width:100, height:100});
                    view.setBackgroundColor(UIColor.greenColor());
                    var label = UILabel.alloc().initWithFrame({x:0, y:0, width:100, height:100});
                    label.setText("JSPatch");
                    label.setTextAlignment(1);
                    view.addSubview(label);
                self.view().addSubview(view)
            }
    });
```

除了上面的代码，在ViewController.m文件中没有编写任何其他的方法，运行工程，可以看到程序并没有崩溃，ViewController执行了creatView方法。

        通过上面的示例，我们发现使用JSPatch可以做一些十分有趣的事。对于iOS应用来说，通过官方渠道AppStore进行应用程序的发布要通过人工审核，有时这个审核周期会非常长，如果在开发者在编写代码时留下了一些小漏洞，应用一旦上线，若要修改掉这个bug就十分艰难了。有了JSPatch，我们可以想象，如果可以定位到线上应用有问题的方法，使用JS文件来修改掉这个方法，这将是多么cool的一件事，事实上，JSPatch的主要用途也是可以实现线上应用极小问题的hotfix。

### 三、JavaScript与Objective-C交互的基础方法

        要使用JSPatch来进行Objective-C风格的方法编写，需要遵守一些JavaScript与Objective-C交互的规则。

#### 1.在JavaScript文件中使用Objective-C类

   在编写JavaScript代码时如果需要用到Objective-C的类，必须先对这个类进行require引用，例如，如果需要使用UIView这个类，需要在使用前进行如下引用：

```
require('UIView')
```

同样也可以一次对多个Objective-C类进行引用：

```
require('UIView, UIColor, UILabel')
```

还有一种更加简便的写法，直接在使用的时候对其进行引用：

```
require('UIView').alloc().init()
```

#### 2.在JavaScript文件中进行Objective-C方法的调用

    在进行Objective-C方法的调用时，分为两种，一种是调用类方法，一种是调用类的对象方法。

调用类方法：通过类名打点的方式来调用类方法，格式类似如下，括号内为参数传递：

```
UIColor.redColor()
```

调用实例方法：通过对象打点的方式调用类的实例方法，格式如下，括号内为参数传递：

```
view.addSubview(label)
```

对于Objective-C中的多参数方法，转化为JavaScript将参数分割的位置以_进行分割，参数全部放入后面的括号中，以逗号分割，示例如下：

```
view.setBackgroundColor(UIColor.colorWithRed_green_blue_alpha(0,0.5,0.5,1))
```

对于Objective-C类的属性变量，在JavaScript中只能使用getter与setter方法来访问，示例如下：

```
label.setText("JSPatch")
```

提示：如果原Objective-C的方法中已经包含了_符号，则在JavaScript中使用__代替。

#### 3.在JavaScript中操作与修改Objective-C类

    JSPatch的最大应用是在应用运行时动态的操作和修改类。

重写或者添加类的方法：

在JavaScript中使用defineClass来定义和修改类中的方法，其编写格式如下所示：

```
/*
classDeclaration:要添加或者重写方法的类名 字符串  如果此类不存在 则会创建新的类
instanceMethods:要添加或者重写的实例方法 {}
classMethods:要添加或者重写的类方法 {}
*/
defineClass(classDeclaration, instanceMethods, classMethods)
```

示例如下:

```
defineClass('ViewController', {
            // replace the -genView method
                newFunc: function() {
                    //编写实例方法
                    self.view().setBackgroundColor(UIColor.redColor())
                }
    
            },{

                myLoad:function(){
                    //编写类方法
                }

            }
            )
```

如果在重写了类中的方法后要调用原方法，需要使用ORIG前缀，示例如下：

```
defineClass('ViewController', {
            // replace the -genView method
                viewDidLoad: function() {
                    //编写实例方法
                    self.ORIGviewDidLoad()
                }
    
            }
            )
```

对于Objective-C中super关键字调用的方法，在JavaScript中可以使用self.super()来调用，例如：

```
defineClass('ViewController', {
            // replace the -genView method
                viewDidLoad: function() {
                    //编写实例方法
                    self.super().viewDidLoad()
                }
    
            }
            )
```

同样JSPatch也可以为类添加临时属性，用于在方法间参数传递，使用set\_Prop\_forKey()来添加属性，使用getProp()来获取属性，注意，JSPatch添加的属性不能使用Objective-C的setter与getter方法访问，如下：

```
defineClass('ViewController', {
            // replace the -genView method
                viewDidLoad: function() {
                    //编写实例方法
                    self.super().viewDidLoad()
                    self.setProp_forKey("JSPatch", "data")
                },
                touchesBegan_withEvent(id,touch){
                    self.getProp("data")
                    self.view().setBackgroundColor(UIColor.redColor())
                }
    
            }
            )
```

关于为类添加协议的遵守，和Objective-C中遵守协议的方式一致，如下：

```
defineClass("ViewController2: UIViewController <UIAlertViewDelegate>", {
            viewDidAppear: function(animated) {
            var alertView = require('UIAlertView')
            .alloc()
            .initWithTitle_message_delegate_cancelButtonTitle_otherButtonTitles(
                                                                                "Alert",
                                                                                "content",
                                                                                self,
                                                                                "OK",
                                                                                null
                                                                                )
            alertView.show()
            },
            alertView_clickedButtonAtIndex:function(alertView, buttonIndex) {
            console.log('clicked index ' + buttonIndex)
            }
            })
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

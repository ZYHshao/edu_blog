---
title: iOS对UIViewController生命周期和属性方法的解析
date: 2015-11-01
categories: iOS之UI控件
tags: []
---
## iOS对UIViewController生命周期和属性方法的解析

### 一、引言

        作为MVC设计模式中的C，Controller一直扮演着项目开发中最重要的角色，它是视图和数据的桥梁，通过它的管理，将数据有条有理的展示在我们的View层上。iOS中的UIViewController是UIKit框架中最基本的一个类。从第一个UI视图到复杂完整项目，都离不开UIViewController作为基础。基于UIViewController的封装和扩展，也能够出色的完成各种复杂界面逻辑。这篇博客，旨在讨论UIViewController的生命周期和属性方法，在最基础的东西上，往往会得到意想不到的惊喜。

### 二、UIViewController的生命周期

        要了解UIViewController，先要弄清楚其生命周期。在面向对象的语言中，是对象，就一定要有生命周期，UIViewController也不例外，生命周期管理Controller的作用范围和时间，也管理其内对象的作用范围和时间。首先，UIViewController中与其生命周期有关的几个函数如下：

```
//类的初始化方法
+ (void)initialize;
//对象初始化方法
- (instancetype)init;
//从归档初始化
- (instancetype)initWithCoder:(NSCoder *)coder;
//加载视图
-(void)loadView;
//将要加载视图
- (void)viewDidLoad;
//将要布局子视图
-(void)viewWillLayoutSubviews;
//已经布局子视图
-(void)viewDidLayoutSubviews;
//内存警告
- (void)didReceiveMemoryWarning;
//已经展示
-(void)viewDidAppear:(BOOL)animated;
//将要展示
-(void)viewWillAppear:(BOOL)animated;
//将要消失
-(void)viewWillDisappear:(BOOL)animated;
//已经消失
-(void)viewDidDisappear:(BOOL)animated;
//被释放
-(void)dealloc;
```

上面这么多的函数，乍一看什么复杂，其实关系什么明朗，除了initialize,init和initWithCoder不是存在所有对象的声明周期中，其他函数都会在UIViewController的声明周期中有序的被调用。那么具体的调用顺序是怎样的呢，最好的办法是实践一下，通过编号打印，结果如下：

![](http://static.oschina.net/uploads/space/2015/1031/230611_fCjT_2340880.png)

这是一个ViewController完整的声明周期，其实里面还有好多地方需要我们注意一下：

1：initialize函数并不会每次创建对象都调用，只有在这个类第一次创建对象时才会调用，做一些类的准备工作，再次创建这个类的对象，initalize方法将不会被调用，对于这个类的子类，如果实现了initialize方法，在这个子类第一次创建对象时会调用自己的initalize方法，之后不会调用，如果没有实现，那么它的父类将替它再次调用一下自己的initialize方法，以后创建也都不会再调用。因此，如果我们有一些和这个相关的全局变量，可以在这里进行初始化。

2：init方法和initCoder方法相似，只是被调用的环境不一样，如果用代码进行初始化，会调用init，从nib文件或者归档进行初始化，会调用initCoder。

3：loadView方法是开始加载视图的起始方法，除非手动调用，否则在ViewController的生命周期中没特殊情况只会被调用一次。

4：viewDidLoad方法是我们最常用的方法的，类中成员对象和变量的初始化我们都会放在这个方法中，在类创建后，无论视图的展现或消失，这个方法也是只会在将要布局时调用一次。

5：viewWillAppear：视图将要展现时会调用。

6：viewWillLayoutSubviews：在viewWillAppear后调用，将要对子视图进行布局。

7：viewDidLayoutSubviews：已经布局完成子视图。

8：viewDidAppare：视图完成显示时调用。

9：viewWillDisappear：视图将要消失时调用。

10：viewDidDisappear：视图已经消失时调用。

11：dealloc：controller被释放时调用。

注意：经过测试，从nib文件加载的controller，只要不释放，在每次viewWillAppare时都会调用layoutSubviews方法，有时甚至会在viewDidAppare后在调用一次layoutSubviews，而重点是从代码加载的则只会在开始调用一次，之后都不会，所以注意，在layoutSubviews中写相关的布局代码十分危险。

### 三、从storyBoard加载UIViewController实例的传值陷阱

        我们知道，当我们从StoryBoard中加载ViewController时，我们在Controller中拖拽的视图是可以被初始化的，这里面有一点需要我们注意，如果我们需要向controller中视图进行传值设置，通过以下方法得到的Controller中，视图还没有被初始化创建出来：

```
 ViewController2 * viewController2 = [[UIStoryboard storyboardWithName:@"Main" bundle:[NSBundle mainBundle]] instantiateViewControllerWithIdentifier:@"ViewController2"];
```

我们可以在ViewController2的storyBoard中拉一个label，然后关联到头文件中，如下打印，会发现我们得到controller时，里面的视图对象并没有进行创建:

```
ViewController2 * viewController2 = [[UIStoryboard storyboardWithName:@"Main" bundle:[NSBundle mainBundle]] instantiateViewControllerWithIdentifier:@"ViewController2"];
    NSLog(@"%@",viewController2.label);
    [self presentViewController:viewController2 animated:YES completion:nil];
```

打印如下：

![](http://static.oschina.net/uploads/space/2015/1031/235109_8Fzf_2340880.png)

可以想象，如果我们这时候需要对label进行一些属性设置，必然失败。有人提出可以在创建后，手动调以下loadView方法，我们试一下，结果如下：

![](http://static.oschina.net/uploads/space/2015/1031/235311_SUDq_2340880.png)

可以看到，手动调用loadView后，label是被创建了出来，但是暴漏了一个更严重的问题，系统不在调用ViewDidLoad方法，这是十分有风险的，因为我们大部分的初始化代码都会放在这个方法里，所以手动调用loadView是一种错误的方法，apple文档声明对于loadView方法，我们从来都不要手动直接调用，那么我们如何实现创建后对成员对象进行传值设置呢，iOS9中增加了这样一个方法：

```
- (void)loadViewIfNeeded NS_AVAILABLE_IOS(9_0);
```

这个方法十分有用，调用这个方法，会将视图创建出来，并且不会忽略viewDidLoad的调用。

在iOS9中，UIViewController还增加了下面一个布尔值的属性，可以同来判断controller的view是否已经加载完成：

```
@property(nullable, nonatomic, readonly, strong) UIView *viewIfLoaded NS_AVAILABLE_IOS(9_0);
```

### 四、UIViewController与StroyBoard的相关相互方法

        对于ViewConroller，我们一般有两种方式创建，一种是用纯代码的方式，一种是与StoryBoard关联，在UIViewController中，有许多方法方便我们与StoryBoard进行交互联系。

#### 1、ViewController直接在StoryBoard中进行跳转的传值

        在StoryBoard中进行界面跳转是十分方便的，我们在StoryBoard中拉入两个ViewController，在一个上面添加一个按钮，点住按钮按住control，将鼠标拉到第二个controller上，会出现如下的跳转选项：

![](http://static.oschina.net/uploads/space/2015/1101/104048_G4Hk_2340880.png)

我们选择一个后，就会在两个controller之间建立一个跳转连接。当我们运行点击按钮后，会自动从第一个controller跳转到第二个controller。在UIViewController中有如下方法可以对是否跳转进行控制：

```
- (BOOL)shouldPerformSegueWithIdentifier:(NSString *)identifier sender:(nullable id)sender NS_AVAILABLE_IOS(6_0);
```

这个方法如果返回NO，自动跳转将不能进行，会被拒绝，需要注意的是，这个方法只会在自动的跳转时被调用，我们手动使用代码跳转StoryBoard中的连接关系时是不会被调用的，我们后面讨论。

        在执行过上述方法后，如果返回YES，系统还会在执行如下一个方法，作为跳转前的准备，我们可以在这个方法中进行一些传值操作，这个方法无论使我们手动进行跳转还是storyboard中自动跳转，都会被执行：

```
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(nullable id)sender NS_AVAILABLE_IOS(5_0);
```

sugur对象中封装了相关的ViewController，可以使用segue.destinationViewController获取。

        segue在StoryBoard中除了用来自动正向跳转外，我们还可以进行反向的跳转，类似pop和dismiss方法，这种segue被称为unwind sugue。例如，我们有一个controller1和一个controllert2，要使用unwind segue从2返回1，我们需要在2中实现如下格式的方法：

```
- (IBAction)unwindSegueToViewController:(UIStoryboardSegue *)segue {
    NSLog(@"unwindSegueToViewController");
}
```

这个方法中的返回值必须为IBAction，参数必须是UIStoryboardSegue，方法名我们可以自己定义，之后在StoryBoard中的ViewController1中的Exit选项中，我们会发现多了一个这样的方法：

![](http://static.oschina.net/uploads/space/2015/1101/132323_g3LU_2340880.png)

我们可以把它连接到viewController2中的一个按钮上：

![](http://static.oschina.net/uploads/space/2015/1101/132400_8n8t_2340880.png)

这样，当我们点击viewController2中的按钮时，就会返回到我们第一个ViewController1中了。

当然，在使用unwind segue方法时，也是会有一些回调帮助我们进行跳转前的设置和传值，UIViewController如下方法会在跳转前调用，返回NO，则不能进行跳转：

```
-(BOOL)canPerformUnwindSegueAction:(SEL)action fromViewController:(UIViewController *)fromViewController withSender:(id)sender{
    NSLog(@"canPerformUnwindSegueAction");
    return YES;
}
```

之后会执行我们自定义的unwindSegue方法，这个方法中我们可以什么都不写，模式是会进行跳转的。

#### 2、使用代码跳转Storyboard中的controller

        我们除了在Storyboard中拉拉扯扯可以进行控制器的跳转外，我们也可以使用代码来跳转Storyboard中segue连接关系。

        在Storyboard中两个控制器间建立一个segue联系，我们可以取一个名字：

![](http://static.oschina.net/uploads/space/2015/1101/133108_LLnB_2340880.png)

在触发跳转的方法中，使用如下方法进行跳转，这里面的参数id就是我们取得segue的id：

```
- (void)performSegueWithIdentifier:(NSString *)identifier sender:(nullable id)sender NS_AVAILABLE_IOS(5_0);
```

下面三个属性我们可以获取controller的nib文件名，其storyBoard和其Bundle:

```
@property(nullable, nonatomic, readonly, copy) NSString *nibName;  
@property(nullable, nonatomic, readonly, strong) NSBundle *nibBundle; 
@property(nullable, nonatomic, readonly, strong) UIStoryboard *storyboard NS_AVAILABLE_IOS(5_0);
```

### 五、UIViewController之间的一些从属关系

        这部分的内容和方法可能我们接触用到的并不多，但是在某些情况下，使用这些方法可以大大的方便某些逻辑。

#### 1、parentViewController

        UIViewController里面封装了一个数组，可以存放其子ViewController，系统中使用的例子就是导航和tabBar这类的控制器，我们使用如下方法可以直接访问这些父的controller：

```
@property(nullable,nonatomic,weak,readonly) UIViewController *parentViewController;
```

#### 2、模态跳转中Controller的从属

        在我们进行控制器的跳转时，只要控制器没有被释放，我们都可以顺藤摸瓜的找到它，使用如下两个方法：

```
//其所present的contller，比如，A和B两个controller，A跳转到B，那么A的presentedViewController就是B
@property(nullable, nonatomic,readonly) UIViewController *presentedViewController  NS_AVAILABLE_IOS(5_0);
//和上面的方法刚好相反，比如，A和B两个controller，A跳转到B，那么B的presentingViewController就是A
@property(nullable, nonatomic,readonly) UIViewController *presentingViewController NS_AVAILABLE_IOS(5_0);
```

了解了上面方法我们可以知道，对于反向传值这样的问题，我们根本不需要代理，block，通知等这样的复杂手段，只需要获取跳转到它的Controller，直接设置即可。举个例子，我们需要在第二个界面消失后，改变第一个界面的颜色，在第二个controller中只需要下面的代码即可实现 ：

```
    self.presentingViewController.view.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    [self dismissViewControllerAnimated:YES completion:nil];
```

### 六、UIViewController的模态跳转及动画特效

        单纯的UIViewController中，我们使用最多的是如下的两个方法，一个向前跳转，一个向后返回:

```
- (void)presentViewController:(UIViewController *)viewControllerToPresent animated: (BOOL)flag completion:(void (^ __nullable)(void))completion NS_AVAILABLE_IOS(5_0);
- (void)dismissViewControllerAnimated: (BOOL)flag completion: (void (^ __nullable)(void))completion NS_AVAILABLE_IOS(5_0);
```

从方法中，我们可以看到，有animated这个参数，来选择是否有动画特效，默认的动画特效是像抽屉一样从手机屏幕的下方向上弹起，当然，这个效果我们可以进行设置，UIViewController有如下一个属性来设置动画特效：

```
@property(nonatomic,assign) UIModalTransitionStyle modalTransitionStyle NS_AVAILABLE_IOS(3_0);
```

注意，这个要设置的是将要跳转到的controller，枚举如下：

```
typedef NS_ENUM(NSInteger, UIModalTransitionStyle) {
    UIModalTransitionStyleCoverVertical = 0,//默认的，从下向上覆盖
    UIModalTransitionStyleFlipHorizontal ,//水平翻转
    UIModalTransitionStyleCrossDissolve,//溶解
    UIModalTransitionStylePartialCurl ,从下向上翻页
};
```

除了跳转的效果，还有一个属性可以设置弹出的controler的填充效果，但是这个属性只在pad上有效，在iphone上无效，都是填充到整个屏幕：

```
@property(nonatomic,assign) UIModalPresentationStyle modalPresentationStyle NS_AVAILABLE_IOS(3_2);
//枚举如下
typedef NS_ENUM(NSInteger, UIModalPresentationStyle) {
        UIModalPresentationFullScreen = 0,//填充整个屏幕
        UIModalPresentationPageSheet,//留下状态栏
        UIModalPresentationFormSheet,//四周留下变暗的空白
        UIModalPresentationCurrentContext ,//和跳转到它的控制器保持一致
        UIModalPresentationCustom NS_ENUM_AVAILABLE_IOS(7_0),//自定义
        UIModalPresentationOverFullScreen NS_ENUM_AVAILABLE_IOS(8_0),
        UIModalPresentationOverCurrentContext NS_ENUM_AVAILABLE_IOS(8_0),
        UIModalPresentationPopover NS_ENUM_AVAILABLE_IOS(8_0) __TVOS_PROHIBITED,
        UIModalPresentationNone NS_ENUM_AVAILABLE_IOS(7_0) = -1,         
};
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

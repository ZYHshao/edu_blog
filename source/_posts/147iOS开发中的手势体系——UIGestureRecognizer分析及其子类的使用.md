---
title: iOS开发中的手势体系——UIGestureRecognizer分析及其子类的使用
date: 2015-11-06
categories: iOS逻辑初窥
tags: []
---
## iOS开发中的手势体系——UIGestureRecognizer分析及其子类的使用

### 一、引言

        在iOS系统中，手势是进行用户交互的重要方式，通过UIGestureRecognizer类，我们可以轻松的创建出各种手势应用于app中。关于UIGestureRecognizer类，是对iOS中的事件传递机制面向应用的封装，将手势消息的传递抽象为了对象。有关消息传递的一些讨论，在前面的博客中有提到：

iOS事件响应控制：[http://my.oschina.net/u/2340880/blog/396161](http://my.oschina.net/u/2340880/blog/396161)。

### 二、手势的抽象类——UIGestureRecognizer

        UIGestureRecognizer将一些和手势操作相关的方法抽象了出来，但它本身并不实现什么手势，因此，在开发中，我们一般不会直接使用UIGestureRecognizer的对象，而是通过其子类进行实例化，iOS系统给我们提供了许多用于我们实例的子类，这些我们后面再说，我们先来看一下，UIGestureRecognizer中抽象出了哪些方法。

#### 1、统一的初始化方法

        UIGestureRecognizer类为其子类准备好了一个统一的初始化方法，无论什么样的手势动作，其执行的结果都是一样的：触发一个方法，可以使用下面的方法进行统一的初始化：

```
- (instancetype)initWithTarget:(nullable id)target action:(nullable SEL)action;
```

当然，如果我们使用alloc-init的方式，也是可以的，下面的方法可以为手势添加触发的selector：

```
- (void)addTarget:(id)target action:(SEL)action;
```

与之相对应的，我们也可以将一个selector从其手势对象上移除：

```
- (void)removeTarget:(nullable id)target action:(nullable SEL)action;
```

上面两个方法是十分有意思的，因为addTarget方式的存在，iOS系统允许一个手势对象可以添加多个selector触发方法，并且触发的时候，所有添加的selector都会被执行，我们以点击手势示例如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
     UITapGestureRecognizer * ges = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(click:)];
    [ges addTarget:self action:@selector(haha)];
     [self.view addGestureRecognizer:ges];
}
-(void)click:(UIGestureRecognizer *)ges{
    
    NSLog(@"第一个手势的触发方法");
    
}
-(void)haha{
    NSLog(@"haha");
}
```

运行后点击屏幕，打印如下，说明两个方法都触发了：

![](http://static.oschina.net/uploads/space/2015/1106/163324_QXWZ_2340880.png)

#### 2、手势状态

        UIgestureRecognizer类中有如下一个属性，里面枚举了一些手势的当前状态:

```
@property(nonatomic,readonly) UIGestureRecognizerState state;
```

枚举值如下：

```
typedef NS_ENUM(NSInteger, UIGestureRecognizerState) {
    UIGestureRecognizerStatePossible,   // 默认的状态，这个时候的手势并没有具体的情形状态
    UIGestureRecognizerStateBegan,      // 手势开始被识别的状态
    UIGestureRecognizerStateChanged,    // 手势识别发生改变的状态
    UIGestureRecognizerStateEnded,      // 手势识别结束，将会执行触发的方法
    UIGestureRecognizerStateCancelled,  // 手势识别取消
    UIGestureRecognizerStateFailed,     // 识别失败，方法将不会被调用
    UIGestureRecognizerStateRecognized = UIGestureRecognizerStateEnded 
};
```

#### 3、常用属性和方法

```
//设置代理，具体的协议后面会说
@property(nullable,nonatomic,weak) id <UIGestureRecognizerDelegate> delegate; 
//设置手势是否有效
@property(nonatomic, getter=isEnabled) BOOL enabled;
//获取手势所在的view
@property(nullable, nonatomic,readonly) UIView *view; 
//获取触发触摸的点
- (CGPoint)locationInView:(nullable UIView*)view; 
//设置触摸点数
- (NSUInteger)numberOfTouches; 
//获取某一个触摸点的触摸位置
- (CGPoint)locationOfTouch:(NSUInteger)touchIndex inView:(nullable UIView*)view;
```

下面的几个BOOL值的属性，对于手势触发的控制也十分重要：

##### （1）

```
@property(nonatomic) BOOL cancelsTouchesInView;
```

上面的属性默认为YES，当这个属性设置为YES时，如果识别到了手势，系统将会发送touchesCancelled:withEvent:消息在其时间传递链上，终止触摸事件的传递，设置为NO，则不会终止事件的传递，举个例子来说，可能会更加清楚一些如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
     UIPanGestureRecognizer * ges = [[UIPanGestureRecognizer alloc]initWithTarget:self action:@selector(click:)];;
    [self.view addGestureRecognizer:ges];
    ges.cancelsTouchesInView=NO;
}
-(void)touchesMoved:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    NSLog(@"123");
}
-(void)click:(UIGestureRecognizer *)ges{
    NSLog(@"第一个手势的触发方法");
}
```

上面我们使用了拖拽手势和touchesMoved两个触发方式，当我们把cancelTouchesInView设置为NO时，在屏幕上滑动，会发现两种方式都在触发，打印如下：

![](http://static.oschina.net/uploads/space/2015/1106/172510_jxXR_2340880.png)

如果我们将cancelTouchesInView改为YES，当手势触发时，将取消触摸消息的触发：

![](http://static.oschina.net/uploads/space/2015/1106/172613_3gaA_2340880.png)

##### （2）

```
@property(nonatomic) BOOL delaysTouchesBegan;
```

通过上面的例子，我们知道，在一个手势触发之前，是会一并发消息给事件传递链的，delaysTouchesBgan属性用于控制这个消息的传递时机，默认这个属性为NO，此时在触摸开始的时候，就会发消息给事件传递链，如果我们设置为YES，在触摸没有被识别失败前，都不会给事件传递链发送消息。

##### （3）

```
@property(nonatomic) BOOL delaysTouchesEnded;
```

这个属性设置手势识别结束后，是立刻发送touchesEnded消息到事件传递链或者等待一个很短的时间后，如果没有接收到新的手势识别任务，再发送。

#### 4、手势间的互斥处理

        有一点需要注意，同一个View上是可以添加多个手势对象的，默认这个手势是互斥的，一个手势触发了就会默认屏蔽其他相似的手势动作，例如：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
     UITapGestureRecognizer * ges = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(click:)];;
    
    //view.backgroundColor = [UIColor redColor];
   
    //ges.delegate=self;
    [self.view addGestureRecognizer:ges];
    
    UITapGestureRecognizer * ges2 = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(click1:)];
//    ges2.delegate=self;
        [self.view addGestureRecognizer:ges2];
}


-(void)click:(UIGestureRecognizer *)ges{
    
    NSLog(@"第一个手势的触发方法");
    
}
-(void)click1:(UIGestureRecognizer *)ges1{
    NSLog(@"第二个手势的触发方法");
    
    
}
```

我们添加的两个手势都是单机手势，会产生冲突，触发是很随机的，如果我们想设置一下当手势互斥时要优先触发的手势，可以使用如下的方法：

```
- (void)requireGestureRecognizerToFail:(UIGestureRecognizer *)otherGestureRecognizer;
```

这个方法中第一个参数是需要时效的手势，第二个是生效的手势。

### 三、UIGestureRecognizerDelegate

        前面我们提到过关于手势对象的协议代理，通过代理的回调，我们可以进行自定义手势，也可以处理一些复杂的手势关系，其中方法如下：

```
//手指触摸屏幕后回调的方法，返回NO则不再进行手势识别，方法触发等
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch;
//开始进行手势识别时调用的方法，返回NO则结束，不再触发手势
- (BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer;
//是否支持多时候触发，返回YES，则可以多个手势一起触发方法，返回NO则为互斥
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer;
//下面这个两个方法也是用来控制手势的互斥执行的
//这个方法返回YES，第一个手势和第二个互斥时，第一个会失效
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRequireFailureOfGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer NS_AVAILABLE_IOS(7_0);
//这个方法返回YES，第一个和第二个互斥时，第二个会失效
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldBeRequiredToFailByGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer NS_AVAILABLE_IOS(7_0);
```

### 四、点击手势——UITapGestureRecognizer

        点击手势十分简单，支持单击和多次点击，在我们手指触摸屏幕并抬起手指时会进行触发，其中有如下两个属性我们可以进行设置：

```
//设置点击次数，默认为单击
@property (nonatomic) NSUInteger  numberOfTapsRequired; 
//设置同时点击的手指数
@property (nonatomic) NSUInteger  numberOfTouchesRequired;
```

### 五、捏合手势——UIPinchGestureRecognizer

        捏合手势是当我们双指捏合和扩张会触发动作的手势，我们可以设置的属性如下：

```
//设置缩放比例
@property (nonatomic)          CGFloat scale; 
//设置捏合速度
@property (nonatomic,readonly) CGFloat velocity;
```

### 六、拖拽手势——UIPanGestureRecognzer

        当我们点中视图进行慢速拖拽时会触发拖拽手势的方法。

```
//设置触发拖拽的最少触摸点，默认为1
@property (nonatomic)          NSUInteger minimumNumberOfTouches; 
//设置触发拖拽的最多触摸点
@property (nonatomic)          NSUInteger maximumNumberOfTouches;  
//获取当前位置
- (CGPoint)translationInView:(nullable UIView *)view;            
//设置当前位置
- (void)setTranslation:(CGPoint)translation inView:(nullable UIView *)view;
//设置拖拽速度
- (CGPoint)velocityInView:(nullable UIView *)view;
```

### 七、滑动手势——UISwipeGestureRecognizer

        滑动手势和拖拽手势的不同之处在于滑动手势更快，拖拽比较慢。

```
//设置触发滑动手势的触摸点数
@property(nonatomic) NSUInteger                        numberOfTouchesRequired; 
//设置滑动方向
@property(nonatomic) UISwipeGestureRecognizerDirection direction;  
//枚举如下
typedef NS_OPTIONS(NSUInteger, UISwipeGestureRecognizerDirection) {
    UISwipeGestureRecognizerDirectionRight = 1 << 0,
    UISwipeGestureRecognizerDirectionLeft  = 1 << 1,
    UISwipeGestureRecognizerDirectionUp    = 1 << 2,
    UISwipeGestureRecognizerDirectionDown  = 1 << 3
};
```

### 八、旋转手势——UIRotationGestureRecognizer

        进行旋转动作时触发手势方法。

```
//设置旋转角度
@property (nonatomic)          CGFloat rotation;
//设置旋转速度 
@property (nonatomic,readonly) CGFloat velocity;
```

###  九、长按手势——UILongPressGestureRecognizer

        进行长按的时候触发的手势方法。

```
//设置触发前的点击次数
@property (nonatomic) NSUInteger numberOfTapsRequired;    
//设置触发的触摸点数
@property (nonatomic) NSUInteger numberOfTouchesRequired; 
//设置最短的长按时间
@property (nonatomic) CFTimeInterval minimumPressDuration; 
//设置在按触时时允许移动的最大距离 默认为10像素
@property (nonatomic) CGFloat allowableMovement;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

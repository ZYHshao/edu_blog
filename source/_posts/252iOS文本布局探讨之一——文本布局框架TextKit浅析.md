---
title: iOS文本布局探讨之一——文本布局框架TextKit浅析
date: 2016-09-08
categories: iOS逻辑初窥
tags: []
---
## iOS文本布局探讨之一——文本布局框架TextKit浅析

### 一、引言

        在iOS开发中，处理文本的视图控件主要有4中，UILabel，UITextField，UITextView和UIWebView。其中UILabel与UITextField相对简单，UITextView是功能完备的文本布局展示类，通过它可以进行复杂的富文本布局，UIWebView主要用来加载网页或者pdf文件，其可以进行HTML,CSS和JS等文件的解析。

        TextKit是一个偏上层的开发框架，在iOS7以上可用，使用它开发者可以方便灵活处理复杂的文本布局，满足开发中对文本布局的各种复杂需求。TextKit实际上是基于CoreText的一个上层框架，其是面向对象的，如果TextKit中提供的API无法满足需求，可以使用CoreText中的API进行更底层的开发。

        官方文档中的一张图片很确切，经常会被用来描述TextKit框架在iOS系统文本渲染中所处的位置。

![](http://static.oschina.net/uploads/space/2016/0907/144209_lTBy_2340880.png)

### 二、TextKit框架的结构

        界面在进行文本的渲染时，有下面几个必要条件：

1.要渲染展示的内容。

2.将内容渲染在某个视图上。

3.内容渲染在视图上的尺寸位置和形状。

在TextKit框架中，提供了几个类分别对应处理上述的必要条件：

1.NSTextStorage对应要渲染展示的内容。

2.UITextView对应要渲染的视图。

3.NSTextContainer对应渲染的尺寸位置和形状信息。

除了上述3个类之外，TextKit框架中的NSLayoutManager类作为协调者来进行布局操作。

上述关系如下图所示：

![](http://static.oschina.net/uploads/space/2016/0907/151329_DAqp_2340880.png)

### 三、使用TextKit进行文本布局流程

        个人理解，TextKit主要用于更精细的处理文本布局以及进行复杂的图文混排布局，使用TextKit进行文本的布局展示十分繁琐，首先需要将显示内容定义为一个NSTextStorage对象，之后为其添加一个布局管理器对象NSLayoutManager，在NSLayoutManager中，需要进行NSTextContainer的定义，定义多了NSTextContainer对象则会将文本进行分页。最后，将要展示的NSTextContainer绑定到具体的UITextView视图上。

示例代码如下：

```objectivec
    //定义Container
    NSTextContainer * container = [[NSTextContainer alloc]initWithSize:CGSizeMake(150, 200)];
    //定义布局管理类
    NSLayoutManager * layoutManager = [[NSLayoutManager  alloc]init];
    //将container添加进布局管理类管理
    [layoutManager addTextContainer:container];
    //定义一个Storage
    NSTextStorage * storage = [[NSTextStorage alloc]initWithString:@"The NSTextContainer class defines a region where text is laid out. An NSLayoutManager uses NSTextContainer to determine where to break lines, lay out portions of text, and so on."];
    //为Storage添加一个布局管理器
    [storage addLayoutManager:layoutManager];
    //将要显示的container与视图TextView绑定
    UITextView * textView = [[UITextView alloc]initWithFrame:self.view.frame textContainer:container];
    [self.view addSubview:textView];

```

上面代码演示的过程如下图所示：

![](http://static.oschina.net/uploads/space/2016/0907/165945_QdHp_2340880.png)

需要注意，TextKit进行布局的核心思路是最终的视图对应一个文本块Container，并不是一段文本内容Storage，LayoutManager会将完整的内容根据其中Container的尺寸进行分页，TextView根据需要显示的部分进行Container的选择。

### 四、了解NSTextContainer类

        NSTextContainer可以简单理解为创建一个文本区块，文本内容将在这个区块中进行渲染，其中常用属性与方法如下：

```objectivec
//初始化方法 设置区块的尺寸
- (instancetype)initWithSize:(CGSize)size;
//与其绑定的layoutManager 需要注意，不是设置这个属性 使用[NSLayoutManager addTextContainer:]方式来进行绑定
@property(nullable, assign, NS_NONATOMIC_IOSONLY) NSLayoutManager *layoutManager;
//替换绑定的布局管理类对象
- (void)replaceLayoutManager:(NSLayoutManager *)newLayoutManager;
//获取区块尺寸
@property(NS_NONATOMIC_IOSONLY) CGSize size;
//设置从区块中剔除某一区域
@property(copy, NS_NONATOMIC_IOSONLY) NSArray<UIBezierPath *> *exclusionPaths;
//设置截断模式 需要注意 这个属性的设置只是会影响此区块的最后一行的截断模式
@property(NS_NONATOMIC_IOSONLY) NSLineBreakMode lineBreakMode;
//设置每行文本左右空出的间距
@property(NS_NONATOMIC_IOSONLY) CGFloat lineFragmentPadding;
//设置TextView上可输入的文本最大行数
@property(NS_NONATOMIC_IOSONLY) NSUInteger maximumNumberOfLines;

//这个方法用于提供给子类进行重写 这里返回的Rect是可以布局文本的区域
- (CGRect)lineFragmentRectForProposedRect:(CGRect)proposedRect atIndex:(NSUInteger)characterIndex writingDirection:(NSWritingDirection)baseWritingDirection remainingRect:(nullable CGRect *)remainingRect;
//这个BOOL值的属性决定Container的宽度是否自适应TextView的宽度
@property(NS_NONATOMIC_IOSONLY) BOOL widthTracksTextView;
//这个BOOL值的属性决定Container的高度是否自适应TextView的高度
@property(NS_NONATOMIC_IOSONLY) BOOL heightTracksTextView;
```

上面所列举的方法中，exclusionPaths属性十分强大，通过设置它，可以将布局区域内剔出一块区域不进行布局，示例代码如下：

```objectivec
 [super viewDidLoad];
    NSTextContainer * container = [[NSTextContainer alloc]initWithSize:CGSizeMake(300, 500)];

    UIBezierPath * path = [UIBezierPath bezierPathWithArcCenter:self.view.center radius:70 startAngle:0 endAngle:M_PI*2 clockwise:YES];
    container.exclusionPaths = @[path];
    container.lineBreakMode = NSLineBreakByCharWrapping;
    NSLayoutManager * layoutManager = [[NSLayoutManager  alloc]init];
    [layoutManager addTextContainer:container];
    NSTextStorage * storage = [[NSTextStorage alloc]initWithString:@"The NSTextContainer class defines a region where text is laid out. An NSLayoutManager uses NSTextContainer to determine where to break lines, lay out portions of text, and so on. An NSTextContainer object normally defines rectangular regions, but you can define exclusion paths inside the text container to create regions where text does not flow. You can also subclass to create text containers with nonrectangular regions, such as circular regions, regions with holes in them, or regions that flow alongside graphics.The NSTextContainer class defines a region where text is laid out. An NSLayoutManager uses NSTextContainer to determine where to break lines, lay out portions of text, and so on. An NSTextContainer object normally defines rectangular regions, but you can define exclusion paths inside the text container to create regions where text does not flow. You can also subclass to create text containers with nonrectangular regions, such as circular regions, regions with holes in them, or regions that flow alongside graphics.An NSLayoutManager uses NSTextContainer to determine where to break lines, lay out portions of text, and so on. An NSTextContainer object normally defines rectangular regions, but you can define exclusion paths inside the text container to create regions where text does not flow. You can also subclass to create text containers with nonrectangular regions, such as circular regions, regions with holes in them, or regions that flow alongside graphics."];
    [storage addLayoutManager:layoutManager];
    UITextView * textView = [[UITextView alloc]initWithFrame:self.view.frame textContainer:container];
    [self.view addSubview:textView];
```

效果如下图：

![](http://static.oschina.net/uploads/space/2016/0907/181210_GVd0_2340880.png)

### 五、关于NSLayoutManager

        顾名思义，NSLayoutManager专门负责对文本的布局渲染，简单理解，其从NSTextStorage从拿去展示的内容，将去处理后布局到NSTextContainer中。

        NSLayoutManager与NSTextContainer的关系为一对多，放入NSLayoutManager中的NSTextContainer会以有序数组的形式进行管理，在内容布局时，超出第一个NSTextContainer的内容会被布局到后一个NSTextContainer中。

        NSLayoutManager中有关NSTextContainer操作的方法如下：

```objectivec
//container数组
@property(readonly, NS_NONATOMIC_IOSONLY) NSArray<NSTextContainer *> *textContainers;
//添加一个container
- (void)addTextContainer:(NSTextContainer *)container;
//在指定位置插入一个container
- (void)insertTextContainer:(NSTextContainer *)container atIndex:(NSUInteger)index;
//删除一个指定的container
- (void)removeTextContainerAtIndex:(NSUInteger)index;
//注意 这个方法不需要显式的调用 当布局Container发生变化时 系统会自动调用
- (void)textContainerChangedGeometry:(NSTextContainer *)container;
```

与布局管理相关的属性与方法如下：

```objectivec
//是否显示隐形的符号
/*
默认为NO，如果设置为YES，则会将空格等隐形字符显示出来
*/
@property(NS_NONATOMIC_IOSONLY) BOOL showsInvisibleCharacters;
//是否显示某些布局控制字符
@property(NS_NONATOMIC_IOSONLY) BOOL showsControlCharacters;
//这个属性可以用于设置断字
/*
这个属性的取值为0到1之间 默认为0 即单词换行时从来不会中断 越接近1 则使用连字符进行单词换行中断的概率越大
*/
@property(NS_NONATOMIC_IOSONLY) CGFloat hyphenationFactor;
//是否使用字体定义的行距
/*
默认使用字体所定义的行距信息 通过设置这个属性为NO可以关闭此功能
*/
@property(NS_NONATOMIC_IOSONLY) BOOL usesFontLeading;
//这个属性设置是否允许对相邻位置的内容进行布局 默认为YES，设置为NO后将可以提供大文本布局的效率
@property(NS_NONATOMIC_IOSONLY) BOOL allowsNonContiguousLayout;

//下面这几个方法用于移除某一范围内的布局
- (void)invalidateGlyphsForCharacterRange:(NSRange)charRange changeInLength:(NSInteger)delta actualCharacterRange:(nullable NSRangePointer)actualCharRange;
- (void)invalidateLayoutForCharacterRange:(NSRange)charRange actualCharacterRange:(nullable NSRangePointer)actualCharRange NS_AVAILABLE(10_5, 7_0);
- (void)invalidateDisplayForCharacterRange:(NSRange)charRange;
- (void)invalidateDisplayForGlyphRange:(NSRange)glyphRange;


```

### 六、文本内容类NSTextStorage

        NSTextStorage实际上是继承自NSMutableAttributedString。NSAttributedString是一种自带属性的字符串类，关于NSAttributedString的基本用法，如下博客中有介绍：

[http://my.oschina.net/u/2340880/blog/397500](http://my.oschina.net/u/2340880/blog/397500)。

        TextKit框架中在对文本进行布局时，主要关注于3个方面：

1.字符的属性，例如颜色，字体等。

2.行与段落的属性，如缩进，行间距等。

3.文档属性，包括四周边距、文档尺寸等。

这些都由NSAttributedString来进行定义。

        如上所介绍的是TextKit框架的主要工作原理，文字渲染，图文混排的更多内容，后面博客会继续探讨。有疏漏之处，共同讨论进步。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: iOS文本布局探讨之三——使用TextKit框架进行富文本布局
date: 2016-09-10
categories: iOS逻辑初窥
tags: []
---
## iOS文本布局探讨之三——使用TextKit框架进行富文本布局

### 一、引言

        关于图文混排，其实以前的博客已经讨论很多，在实际开发中，经常使用第三方的框架来完成排版的需求，其中RCLabel和RTLabel是两个比较好用的第三方库，他们的实现都是基于UIView的，通过更底层的CoreText相关API来进行图文处理。相关介绍博客地址如下：

iOS中支持HTML标签渲染的MDHTMLLaebl：[http://my.oschina.net/u/2340880/blog/703254](http://my.oschina.net/u/2340880/blog/703254)。

扩展于RCLabel的支持异步加载网络图片的富文本引擎的设计：[http://my.oschina.net/u/2340880/blog/499311](http://my.oschina.net/u/2340880/blog/499311)。

iOS开发封装一个可以响应超链接的label——基于RCLabel的交互扩展：[http://my.oschina.net/u/2340880/blog/550194](http://my.oschina.net/u/2340880/blog/550194)。

### 二、原生UILabel真的只能渲染文字么？

        CoreText是一个比较底层且十分强大的文本渲染框架，但是其使用起来并不是十分方便。在较低版本的iOS系统中，要进行富文本排版十分困难。在iOS6中，系统为UILabel，UITextView等这类文本渲染控件引入了NSAttributedString属性，有了NSAttributedString这个类，创建灵活多彩的文本控件变得十分轻松，开发者只需要配置NSAttributedString属性字符串即可。但是要进行图文混排，依然比较困难。iOS7之后引入TextKit框架，就完美的解决了图文混排这样的问题。

        首先，iOS7中新添加了一类NSTextAttachment，从类名理解它是一个文本附件，其实也正是如此，NSTextAttachment类可以向文本中添加一些附件，这有些向邮件系统，寄信者可以向邮件中添加附件一同发送出去。NSTextAttachment类并不直接参与富文本的渲染与布局，渲染和布局依然由NSAttributedString类来完成，NSAttributedString类中提供了方法将NSTextAttachment所描述的内容转换为NSAttributedString示例。以一个简单的图文混排为例：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    //进行NSTextAttachment的创建
    NSTextAttachment * attach = [[NSTextAttachment  alloc]init];
    //设置显示的图片
    attach.image =[UIImage imageNamed:@"image"];
    //设置尺寸
    attach.bounds = CGRectMake(0, 0, 120, 60);
    NSTextAttachment * attach2 = [[NSTextAttachment  alloc]init];
    attach2.image =[UIImage imageNamed:@"image2"];
    attach2.bounds = CGRectMake(0, 0, 100, 90);
    //创建文本NSAttributedString对象
    NSMutableAttributedString * attri = [[NSMutableAttributedString alloc]initWithString:@"Describes a dictionary that fully specifies a font.... UIFontDescriptorInherits From NSObject UIFontDescriptor NSObject UIFontDescriptor Conforms To CVarArgT... 这里是中文"];
    //将NSTextAttachment映射为NSAttributedString对象
    NSMutableAttributedString * att = [[NSMutableAttributedString alloc]initWithAttributedString:[NSAttributedString attributedStringWithAttachment:attach]];
    //将图片插入NSAttributedString中
    [attri insertAttributedString:att atIndex:15];
    [attri insertAttributedString:[NSAttributedString attributedStringWithAttachment:attach2] atIndex:130];
    UILabel * label = [[UILabel alloc]initWithFrame:CGRectMake(20, 20, 280, 540)];
    label.backgroundColor = [UIColor grayColor];
    label.numberOfLines = 0;
    label.attributedText = attri;
    [self.view addSubview:label];
}

```

运行工程后，效果如下图所示，其实只使用UILabel也可以实现复杂的富文本和图文混排：

![](http://static.oschina.net/uploads/space/2016/0910/122032_7QI6_2340880.png)

### 三、为富文本附件添加用户交互能力

        TextKit框架强大到只使用UILabel就可以完成复杂的富文本布局，但是UILabel有一个致命的缺陷，其无法进行用户交互。试想，如果可以向一段文本中添加任意数据类型的文件，当用户点击这个文件时，可以获取到文件数据并进行业务逻辑处理，这将十分酷。这样富文本布局其实就不只局限于图文混排了，我们可以插入音频，插入视频，甚至插入任意自定义格式的数据。结合使用NSTextAttachment与UITextView，这些都能实现。先看NSTextAttachment类中的一些常用属性与方法：

```objectivec
//这个初始化方法用于创建携带任意数据的文本附件
- (instancetype)initWithData:(nullable NSData *)contentData ofType:(nullable NSString *)uti NS_DESIGNATED_INITIALIZER NS_AVAILABLE(10_11, 7_0);
//携带的数据内容
@property(nullable, copy, NS_NONATOMIC_IOSONLY) NSData *contents NS_AVAILABLE(10_11, 7_0);
//数据类型
@property(nullable, copy, NS_NONATOMIC_IOSONLY) NSString *fileType NS_AVAILABLE(10_11, 7_0);

//设置渲染的图片 需要注意 如果设置的这个 附件携带的数据 fileWrapper目录内容将无效
@property(nullable, strong, NS_NONATOMIC_IOSONLY) UIImage *image NS_AVAILABLE(10_11, 7_0);
//设置图片渲染的尺寸
@property(NS_NONATOMIC_IOSONLY) CGRect bounds NS_AVAILABLE(10_11, 7_0);

//设置附件携带的文件目录 需要注意 如果设置了这个属性 image和data将无效
@property(nullable, strong, NS_NONATOMIC_IOSONLY) NSFileWrapper *fileWrapper;
```

结合UITextView可以为NSAttributedString属性字符串添加超链接，在代码回调中监听此超链接的回调可以获取NSTextAttachment携带的附件内容，如此就可以自由的进行业务处理了，示例代码如下：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    //保留一个数组存放附件
    _attArray = [NSMutableArray array];
    //创建附件数据
    NSData * stringData = [NSData dataWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"image3" ofType:@"gif"]];
    NSTextAttachment * attach = [[NSTextAttachment  alloc]initWithData:stringData ofType:@"gif"];
    [_attArray addObject:attach];
    attach.bounds = CGRectMake(0, 0, 30, 40);
    NSMutableAttributedString * attri = [[NSMutableAttributedString alloc]initWithString:@"Describes a dictionary that fully specifies a font.... UIFontDescriptorInherits From NSObject UIFontDescriptor NSObject UIFontDescriptor Conforms To CVarArgT... 这里是中文"];
    NSMutableAttributedString * att = [[NSMutableAttributedString alloc]initWithAttributedString:[NSAttributedString attributedStringWithAttachment:attach]];
    //为NSTextAttachment转换为的NSAttributedString添加超链接
    [att addAttributes:@{NSLinkAttributeName:@"url..."} range:NSMakeRange(0, att.string.length)];
    [attri insertAttributedString:att atIndex:15];
    UITextView * textView = [[UITextView alloc]initWithFrame:CGRectMake(20, 20, 280, 540)];
    textView.backgroundColor = [UIColor grayColor];
    textView.dataDetectorTypes = UIDataDetectorTypeLink;
    textView.delegate =self;
    textView.attributedText = attri;
    textView.editable =    NO;
    [self.view addSubview:textView];
}
```

实现如下的TextView代理方法：

```objectivec
-(BOOL)textView:(UITextView *)textView shouldInteractWithURL:(NSURL *)URL inRange:(NSRange)characterRange{
    //可以获取到url 进行匹配
    NSLog(@"%@",URL);
    //取出NSTextAttachment附件
    NSTextAttachment * attach =_attArray.firstObject;
    NSLog(@"%@--",attach.contents);
    return YES;
}
```

向文本中添加任意数据的NSTextAttachment会展现一个文件的图标，如下图所示：

![](http://static.oschina.net/uploads/space/2016/0910/125439_hZLn_2340880.png)

当用户点击文件图标时，会将携带的gif文件数据进行打印。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

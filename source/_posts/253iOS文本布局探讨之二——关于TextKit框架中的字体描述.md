---
title: iOS文本布局探讨之二——关于TextKit框架中的字体描述
date: 2016-09-09
categories: iOS逻辑初窥
tags: []
---
## iOS文本布局探讨之二——关于TextKit框架中的字体描述

### 一、引言

        UIFont是iOS开发中处理文本字体的类，关于UIFont的相关内容，以前的一篇博客有详细介绍，本片博客主要介绍关于动态字体的应用与字体描述类NSFontDescriptor的应用。

UIFont应用介绍：[http://my.oschina.net/u/2340880/blog/397115](http://my.oschina.net/u/2340880/blog/397115)。

### 二、iOS系统中的动态字体

        所谓动态字体，是指在应用使用中，用户可以动态调整字体的风格字号等。在iOS7及之后的iOS系统版本，TextKit框架中提供了一个新的类UIFontDescriptor。简单理解，UIFontDescriptor类是专门用来描述字体的，其中提供了许多方法可以直接创建出某种字体，也可以对字体进行设置和调整。动态字体也由这个类来创建。

        在iOS7之后，系统增加了动态字体的功能，当用户在系统设置中修改字体的属性或者字号时，不仅会影响系统应用的字体，第三方应用的字体也可以进行相应调整。系统设置字体界面如下：

![](http://static.oschina.net/uploads/space/2016/0909/110629_t9jn_2340880.png)

使用UIFontDescriptor类中的如下方法可以创建动态字体：

```objectivec
//创建动态字体的字体描述类实例
+ (UIFontDescriptor *)preferredFontDescriptorWithTextStyle:(NSString *)style;

```

UIFont类中的如下方法可以将字体描述类转换成UIFont字体：

```objectivec
+ (UIFont *)fontWithDescriptor:(UIFontDescriptor *)descriptor size:(CGFloat)pointSize NS_AVAILABLE_IOS(7_0);
```

系统定义了一组动态字体的风格字符创常量，开发者可以根据需求选用：

```objectivec
//标题1
UIKIT_EXTERN NSString *const UIFontTextStyleTitle1 NS_AVAILABLE_IOS(9_0);
//标题2
UIKIT_EXTERN NSString *const UIFontTextStyleTitle2 NS_AVAILABLE_IOS(9_0);
//标题3
UIKIT_EXTERN NSString *const UIFontTextStyleTitle3 NS_AVAILABLE_IOS(9_0);
//大标题
UIKIT_EXTERN NSString *const UIFontTextStyleHeadline NS_AVAILABLE_IOS(7_0);
//子标题
UIKIT_EXTERN NSString *const UIFontTextStyleSubheadline NS_AVAILABLE_IOS(7_0);
//内容
UIKIT_EXTERN NSString *const UIFontTextStyleBody NS_AVAILABLE_IOS(7_0);
//标注
UIKIT_EXTERN NSString *const UIFontTextStyleCallout NS_AVAILABLE_IOS(9_0);
//注脚
UIKIT_EXTERN NSString *const UIFontTextStyleFootnote NS_AVAILABLE_IOS(7_0);
//字幕
UIKIT_EXTERN NSString *const UIFontTextStyleCaption1 NS_AVAILABLE_IOS(7_0);
//字幕2
UIKIT_EXTERN NSString *const UIFontTextStyleCaption2 NS_AVAILABLE_IOS(7_0);
```

### 三、关于UIFontDescriptor类

        UIFontDescriptor类可以直接通过字体名称来进行创建：

```objectivec
//通过字体名称和字号尺寸来进行UIFontDescriptor对象的创建
+ (UIFontDescriptor *)fontDescriptorWithName:(NSString *)fontName size:(CGFloat)size;
//通过字体名称创建UIFontDescriptor对象，并且设置变换参数
+ (UIFontDescriptor *)fontDescriptorWithName:(NSString *)fontName matrix:(CGAffineTransform)matrix;
```

CGAffineTransform是一个结构体，其用于文本的控件变换十分强大，在CoreAnimation框架中有CATransform3D这个结构体，CGAffineTransform与其用法十分相似，使其它可以完成文字的形变，旋转等。示例如下：

```objectivec
    //进行旋转
    CGAffineTransform transfom = CGAffineTransformRotate(CGAffineTransformIdentity, 0.1);
    //进行纵向拉伸
    transfom = CGAffineTransformScale(transfom, 1, 3);
    UIFontDescriptor * fontDes = [UIFontDescriptor fontDescriptorWithName:[UIFont systemFontOfSize:14].fontName matrix:transfom];
    UIFont * font = [UIFont fontWithDescriptor:fontDes size:14];
    UILabel * label = [[UILabel alloc]initWithFrame:CGRectMake(20, 100, 280, 400)];
    label.backgroundColor = [UIColor grayColor];
    label.font = font;
    label.numberOfLines = 0;
    label.text = @"Describes a dictionary that fully specifies a font.... UIFontDescriptorInherits From NSObject UIFontDescriptor NSObject UIFontDescriptor Conforms To CVarArgT... 这里是中文";
    [self.view addSubview:label];
```

效果如下：

![](http://static.oschina.net/uploads/space/2016/0909/124432_TqrF_2340880.png)

开发者也可以通过配置地点的方式来创建UIFontDescriptor对象：

```objectivec
- (instancetype)initWithFontAttributes:(NSDictionary<NSString *, id> *)attributes;
```

字典中可以配置的键值如下：

```objectivec
//需要配置为NSValue值 CGAffineTransform
UIKIT_EXTERN NSString *const UIFontDescriptorMatrixAttribute;
//需要配置为一个集合set 包含所有字体字符
UIKIT_EXTERN NSString *const UIFontDescriptorCharacterSetAttribute;
//需要配置为一个数组 数组中为字体描述对象
UIKIT_EXTERN NSString *const UIFontDescriptorCascadeListAttribute;
//需要配置为一个字典 其中进行字体特征的描述 后面会介绍
UIKIT_EXTERN NSString *const UIFontDescriptorTraitsAttribute;
//需要配置为NSNumber类型的 浮点数 其会影响到字体排版时的字符间距
UIKIT_EXTERN NSString *const UIFontDescriptorFixedAdvanceAttribute;
//需要配置为一个数组 数组中为字典 字典中对字型进行配置
/*
//字典中需要配置这两个键
UIKIT_EXTERN NSString *const UIFontFeatureTypeIdentifierKey NS_AVAILABLE_IOS(7_0);
UIKIT_EXTERN NSString *const UIFontFeatureSelectorIdentifierKey NS_AVAILABLE_IOS(7_0);
*/
UIKIT_EXTERN NSString *const UIFontDescriptorFeatureSettingsAttribute;
//配置字体风格 可用的在前面列举过
UIKIT_EXTERN NSString *const UIFontDescriptorTextStyleAttribute;
```

关于字体的特征藐视，即上面UIFontDescriptorTraitsAttribute键值所配置的字典，这个字典中可以设置的键值如下：

```objectivec
//这个键值需要配置为一个NSNumber值，设置文字的渲染特征 后面会介绍
UIKIT_EXTERN NSString *const UIFontSymbolicTrait;
//设置字体的粗细属性 
/*
这个键可以设置的值如下
UIKIT_EXTERN const CGFloat UIFontWeightUltraLight NS_AVAILABLE_IOS(8_2);
UIKIT_EXTERN const CGFloat UIFontWeightThin NS_AVAILABLE_IOS(8_2);
UIKIT_EXTERN const CGFloat UIFontWeightLight NS_AVAILABLE_IOS(8_2);
UIKIT_EXTERN const CGFloat UIFontWeightRegular NS_AVAILABLE_IOS(8_2);
UIKIT_EXTERN const CGFloat UIFontWeightMedium NS_AVAILABLE_IOS(8_2);
UIKIT_EXTERN const CGFloat UIFontWeightSemibold NS_AVAILABLE_IOS(8_2);
UIKIT_EXTERN const CGFloat UIFontWeightBold NS_AVAILABLE_IOS(8_2);
UIKIT_EXTERN const CGFloat UIFontWeightHeavy NS_AVAILABLE_IOS(8_2);
UIKIT_EXTERN const CGFloat UIFontWeightBlack NS_AVAILABLE_IOS(8_2);
*/
UIKIT_EXTERN NSString *const UIFontWeightTrait;
//设置字体宽度
UIKIT_EXTERN NSString *const UIFontWidthTrait;
//设置字体倾斜
UIKIT_EXTERN NSString *const UIFontSlantTrait;
```

关于上面UIFontSymbolicTrait键值，定义在UIFontDescriptorSymbolicTraits枚举中，如下：

```objectivec
typedef NS_OPTIONS(uint32_t, UIFontDescriptorSymbolicTraits) {
    UIFontDescriptorTraitItalic = 1u << 0,
    UIFontDescriptorTraitBold = 1u << 1,
    UIFontDescriptorTraitExpanded = 1u << 5,
    UIFontDescriptorTraitCondensed = 1u << 6,
    UIFontDescriptorTraitMonoSpace = 1u << 10, 
    UIFontDescriptorTraitVertical = 1u << 11,
    UIFontDescriptorTraitUIOptimized = 1u << 12, 
    UIFontDescriptorTraitTightLeading = 1u << 15,
    UIFontDescriptorTraitLooseLeading = 1u << 16,
   
    UIFontDescriptorClassMask = 0xF0000000,
    
    UIFontDescriptorClassUnknown = 0u << 28,
    UIFontDescriptorClassOldStyleSerifs = 1u << 28,
    UIFontDescriptorClassTransitionalSerifs = 2u << 28,
    UIFontDescriptorClassModernSerifs = 3u << 28,
    UIFontDescriptorClassClarendonSerifs = 4u << 28,
    UIFontDescriptorClassSlabSerifs = 5u << 28,
    UIFontDescriptorClassFreeformSerifs = 7u << 28,
    UIFontDescriptorClassSansSerif = 8u << 28,
    UIFontDescriptorClassOrnamentals = 9u << 28,
    UIFontDescriptorClassScripts = 10u << 28,
    UIFontDescriptorClassSymbolic = 12u << 28
} NS_ENUM_AVAILABLE_IOS(7_0);
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

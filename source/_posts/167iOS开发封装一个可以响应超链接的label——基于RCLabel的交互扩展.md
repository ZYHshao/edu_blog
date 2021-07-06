---
title: iOS开发封装一个可以响应超链接的label——基于RCLabel的交互扩展
date: 2015-12-23
categories: 代码灵魂
tags: []
---
## iOS开发封装一个可以响应超链接的label——基于RCLabel的交互扩展

### 一、引言

        iOS系统是一个十分注重用户体验的系统，在iOS系统中，用户交互的方案也十分多，然而要在label中的某部分字体中添加交互行为确实不容易的，如果使用其他类似Button的控件来模拟，文字的排版又将是一个解决十分困难的问题。这个问题的由来是项目中的一个界面中有一些广告位标签，而这些广告位的标签却是嵌在文本中的，当用户点击文字标签的位置时，会跳转到响应的广告页。

        CoreText框架和一些第三方库可以解决这个问题，但直接使用CoreText十分复杂，第三方库多注重于富文本的排版，对类似文字超链接的支持亦不是特别简洁，我们可以借助一些第三方的东西进行针对性更强，更易用的封装。

        RCLabel是一个第三方的将html字符串进行文本布局的工具，代码十分轻巧，并且其是基于CoreText框架的，其原生性和扩展性十分强。在以前的一篇博客中，我将RCLabel进行了一些改进，使其支持异步加载远程图片，并且提供了更加简洁的面向应用的方法，博客地址如下：

扩展于RCLabel的支持异步加载网络图片的富文本引擎的设计:[http://my.oschina.net/u/2340880/blog/499311](http://my.oschina.net/u/2340880/blog/499311) 。

        本篇博文，将在其基础上，完成设计一个可以支持文本超链接的文字视图。

### 二、视图类与模型类的设计

        RCLabel的核心之处在于将HTML文本转换为富文本布局视图，因此我们可以将要显示的文本编程html字符串，将其可以进行用户交互的部分进行html超链接关联，RCLabel就检测到我们点击的区域进行响应逻辑的回调。设计类如下：

.h文件

```
//文本与超链接地址关联的model类 后面会说
@class YHBaseLinkingLabelModel;

@protocol YHBaseLinkingLabelProtocol <NSObject>

@optional
/**
 *点击超链接后出发的代理方法 model中有链接地址和文字
 */
-(void)YHBaseLinkingLabelClickLinking:(YHBaseLinkingLabelModel *)model;
/**
 *尺寸改变后出发的方法
 */
-(void)YHBaseLinkingLabelSizeChange:(CGSize)size;
@end

@interface YHBaseLinkingLabel : YHBaseView
/**
 *文字数组 里面存放这文字对应的超链接对象
 */
@property(nonatomic,strong)NSArray<YHBaseLinkingLabelModel *> * textArray;
@property(nonatomic,weak)id<YHBaseLinkingLabelProtocol>delegate;
/**
 *设置文字颜色
 */
@property(nonatomic,strong)UIColor * textColor;
/**
 *设置超链接文字颜色
 */
@property(nonatomic,strong)UIColor * linkColor;
/**
 *设置字体大小
 */
@property(nonatomic,assign)NSUInteger fontSize;
/**
 *设置超链接字体大小
 */
@property(nonatomic,assign)int linkingFontSize;
/**
 *设置是否显示下划线
 */
@property(nonatomic,assign)BOOL isShowUnderLine;
@end
```

.m文件

```
@interface YHBaseLinkingLabel()<YHBaseHtmlViewProcotop>
@end

@implementation YHBaseLinkingLabel
{
    //以前博客中 封装的显示HTML字符串富文本的视图
    YHBaseHtmlView * _label;
}
/*
// 重载一些初始化方法
- (instancetype)init
{
    self = [super init];
    if (self) {
        _label = [[YHBaseHtmlView alloc]init];
        [self addSubview:_label];
        [_label mas_makeConstraints:^(MASConstraintMaker *make) {
            make.leading.equalTo(@0);
            make.trailing.equalTo(@0);
            make.top.equalTo(@0);
            make.bottom.equalTo(@0);
        }];
         _label.delegate=self;
    }
    return self;
}
- (instancetype)initWithCoder:(NSCoder *)coder
{
    self = [super initWithCoder:coder];
    if (self) {
        _label = [[YHBaseHtmlView alloc]init];
        [self addSubview:_label];
        [_label mas_makeConstraints:^(MASConstraintMaker *make) {
            make.leading.equalTo(@0);
            make.trailing.equalTo(@0);
            make.top.equalTo(@0);
            make.bottom.equalTo(@0);
        }];
         _label.delegate=self;
    }
    return self;
}
- (instancetype)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        _label = [[YHBaseHtmlView alloc]init];
        [self addSubview:_label];
        [_label mas_makeConstraints:^(MASConstraintMaker *make) {
            make.leading.equalTo(@0);
            make.trailing.equalTo(@0);
            make.top.equalTo(@0);
            make.bottom.equalTo(@0);
        }];
        _label.delegate=self;
    }
    return self;
}
//设置文本数组
-(void)setTextArray:(NSArray<YHBaseLinkingLabelModel *> *)textArray{
    _textArray = textArray;
    //进行html转换
    NSString * htmlString = [self transLinkingDataToHtmlStr:textArray];
    //进行布局
    [_label reSetHtmlStr:htmlString];
    
}
-(void)setTextColor:(UIColor *)textColor{
    _textColor = textColor;
    _label.fontColor = textColor;
}
-(void)setLinkColor:(UIColor *)linkColor{
    _linkColor = linkColor;
    _label.linkingColor = linkColor;
}
-(void)setFontSize:(NSUInteger)fontSize{
    _fontSize = fontSize;
    [_label setFontSize:(int)fontSize];
}
-(void)setLinkingFontSize:(int)linkingFontSize{
    _linkingFontSize = linkingFontSize;
    [_label setLinkingSize:linkingFontSize];
}
-(void)setIsShowUnderLine:(BOOL)isShowUnderLine{
    _isShowUnderLine = isShowUnderLine;
    [_label setShowUnderLine:isShowUnderLine];
}
-(NSString *)transLinkingDataToHtmlStr:(NSArray<YHBaseLinkingLabelModel *> *)data{
    NSMutableString * mutStr = [[NSMutableString alloc]init];
    for (int i=0; i<data.count; i++) {
    //这个model中存放的是超链接部分的文字和对应的url
        YHBaseLinkingLabelModel * model = data[i];
        if (!model.linking) {
            [mutStr appendString:model.text];
        }else {
            [mutStr appendString:@"<a href="];
            [mutStr appendString:model.linking];
            [mutStr appendString:@">"];
            [mutStr appendString:model.text];
            [mutStr appendString:@"</a>"];
        }
    }
    return mutStr;
}

#pragma mark delegate
//点击的回调
-(void)YHBaseHtmlView:(YHBaseHtmlView *)htmlView ClickLink:(NSString *)url{
    for (YHBaseLinkingLabelModel * model in _textArray) {
        if ([model.linking isEqualToString:url]) {
            if ([self.delegate respondsToSelector:@selector(YHBaseLinkingLabelClickLinking:)]) {
                [self.delegate YHBaseLinkingLabelClickLinking:model];
                return;
            }
        }
    }
}
//布局尺寸改变的回调
-(void)YHBaseHtmlView:(YHBaseHtmlView *)htmlView SizeChanged:(CGSize)size{
    if ([self.delegate respondsToSelector:@selector(YHBaseLinkingLabelSizeChange:)]) {
        [self.delegate YHBaseLinkingLabelSizeChange:size];
    }
}
@end
```

上面我们有用到一个YHBaseLinkingLabelModel类，这个类进行了链接与字符的映射，设计如下：

```
@interface YHBaseLinkingLabelModel : YHBaseModel
/**
 *文字内容
 */
@property(nonatomic,strong)NSString * text;
/**
 *超链接地址 nil则为无
 */
@property(nonatomic,strong)NSString * linking;

@end
```

        YHBaseHtmlView类是对RCLabel的一层封装，其中也对RCLabel进行了一些优化和改动，代码较多且在上篇博客中有介绍，这里不再多做解释了。

        在ViewController中写如下代码进行使用：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
   YHBaseLinkingLabel * label = [[YHBaseLinkingLabel alloc]initWithFrame:CGRectMake(100, 100, 200, 100)];
    NSMutableArray * array = [[NSMutableArray alloc]init];
    for (int i=0; i<6; i++) {
        YHBaseLinkingLabelModel * model = [[YHBaseLinkingLabelModel alloc]init];
        if (!(i%2)) {
            model.text =[NSString stringWithFormat:@"第%d个标签",i];
            model.linking = [NSString stringWithFormat:@"第%d个标签",i];
        }else{
            model.text = @",不能点得文字,";
        }
        [array addObject:model];
    }
    label.textColor = [UIColor blackColor];
    label.linkColor = [UIColor purpleColor];
    label.fontSize = 15;
    label.linkingFontSize = 17;
    label.isShowUnderLine=YES;
    label.delegate=self;
    label.textArray = array;
    [self.view addSubview:label];
   
}
-(void)YHBaseLinkingLabelClickLinking:(YHBaseLinkingLabelModel *)model{
    NSLog(@"%@",model.linking);
}
```

运行效果如下：

![](http://static.oschina.net/uploads/space/2015/1223/224957_KSL8_2340880.png)

效果不错，并且十分简单易用，对吧。

        我将这部分的相关代码集成进了以前写的一个项目开发框架中，git地址是：[https://github.com/ZYHshao/YHBaseFoundationTest](https://github.com/ZYHshao/YHBaseFoundationTest) 。总体看来，这个框架并不是干货，只是我开发中的一些积累，如果可以帮到你，择优而用，如果需要和我交流，QQ316045346，对视欢迎。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

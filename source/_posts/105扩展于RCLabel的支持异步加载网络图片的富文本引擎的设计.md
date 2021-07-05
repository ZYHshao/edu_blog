---
title: 扩展于RCLabel的支持异步加载网络图片的富文本引擎的设计
date: 2015-08-30
categories: 代码灵魂
tags: []
---
## 扩展于RCLabel的支持异步加载网络图片的富文本引擎的设计

        在iOS开发中，图文混排一直都是UI编程的一个核心点，也有许多优秀的第三方引擎，其中很有名的一套图文混排的框架叫做DTCoreText。但是在前些日的做的一个项目中，我并没有采用这套框架，原因有二，一是这套框架体积非常大，而项目的需求其实并不太高；二是要在这套框架中修改一些东西，难度也非常大，我最终采用的是一个叫做RCLabel的第三方控件，经过一些简单的优化和完善，达到了项目的要求。

        先来介绍一下我项目中的图文混排的需求：首先我从服务器中取到的数据是字符串，但是其中穿插图片的位置是一个HTML的图片标签，标签里的资源路径就是图片的请求地址。需要达到的要求是这些数据显示出来后，图片的位置要空出来，然后通过异步的网络请求获取图片的数据，再将图片插入文字中。

        要自己实现一套这样的引擎确实会比较麻烦，幸运的是RCLabel可以完美的帮我们解析带有HTML标签的数据，进行图文混排，我们先来看一下这个东西怎么用，下面是我封装的一个展示html数据的view：

```
@interface YHBaseHtmlView()<YHRTLabelImageDelegate>
{
    //RCLabel对象
    RCLabel * _rcLabel;
    //保存属性 用于异步加载完成后刷新
    RTLabelComponentsStructure * _origenComponent;
    //含html标签的数据字符串
    NSString * _srt;
}

@end

@implementation YHBaseHtmlView

/*
// Only override drawRect: if you perform custom drawing.
// An empty implementation adversely affects performance during animation.
- (void)drawRect:(CGRect)rect {
    // Drawing code
}
*/
- (instancetype)initWithCoder:(NSCoder *)coder
{
    self = [super initWithCoder:coder];
    if (self) {
    //将rclabel初始化
        _rcLabel = [[RCLabel alloc]init];
        [self addSubview:_rcLabel];

    }
    return self;
}

- (instancetype)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        _rcLabel = [[RCLabel alloc]initWithFrame:frame];
        [self addSubview:_rcLabel];
    }
    return self;
}
-(void)reSetHtmlStr:(NSString *)htmlStr{
    _srt = htmlStr;
    //这个代理是我额外添加的 后面解释
    _rcLabel.imageDelegate=self;
    //设置frame
    _rcLabel.frame=CGRectMake(0, 0, self.frame.size.width, 0);
    //设置属性
    _origenComponent = [RCLabel extractTextStyle:htmlStr IsLocation:NO withRCLabel:_rcLabel];
    _rcLabel.componentsAndPlainText = _origenComponent;
   //获取排版后的size
    CGSize size = [_rcLabel optimumSize];
    //重新设置frame
    _rcLabel.frame=CGRectMake(0, 0, _rcLabel.frame.size.width, size.height);
    self.frame=CGRectMake(self.frame.origin.x, self.frame.origin.y, _rcLabel.frame.size.width, size.height);
}
//这是我额外添加的代理方法的实现
-(void)YHRTLabelImageSuccess:(RCLabel *)label{
    _origenComponent = [RCLabel extractTextStyle:_srt IsLocation:NO withRCLabel:_rcLabel];
    _rcLabel.componentsAndPlainText = _origenComponent;
    
    CGSize size = [_rcLabel optimumSize];
    _rcLabel.frame=CGRectMake(0, 0, _rcLabel.frame.size.width, size.height);
    self.frame=_rcLabel.frame;
    if ([self.delegate respondsToSelector:@selector(YHBaseHtmlView:SizeChanged:)]) {
        [self.delegate YHBaseHtmlView:self SizeChanged:self.frame.size];
    }
}
```

RCLabel的用法很简单，总结来说只有三步：

1.初始化并设置frame

2.通过带html标签的数据进行属性的初始化

3.将属性进行set设置并重设视图frame

RCLabel是很强大，并且代码很简练，但是其中处理图片的部分必须是本地的图片，即图片html标签中的路径必须是本地图片的名字，其内部是通过\[UIImage ImageNamed:\]这个方法进行图片的渲染的，所以要达到我们的需要，我们需要对其进行一些简单的扩展：

1、在属性设置方法中添加一个参数，来区分本地图片与网络图片：

```
//我在这个方法中添加了location这个bool值，实际上rclabel这个参数也是我添加的，是为了后面代理使用的
+ (RTLabelComponentsStructure*)extractTextStyle:(NSString*)dataimage IsLocation:(BOOL)location withRCLabel:(RCLabel *)rcLabel;
```

2、在实现方法中添加如下代码，因为原文件有1900多行，在其中弄清楚逻辑关系也确实费了我不小的力气，我这里只将我添加的代码贴过来

```
#warning 这里进行了兼容性处理
                if (location) {
                //本地图片的渲染
                    if (tempURL) {
                        UIImage  *tempImg = [UIImage imageNamed:tempURL];
                        component.img = tempImg;
                        
                    }
                }else{//这里做远程图片数据的处理
                //这里我进行了缓存的操作，这个缓存中心是我封装的框架中的另一套东西，这里可以不用在意
                    //先读缓存
                    NSData * ceche = [[YHBaseCecheCenter sharedTheSingletion] readCecheFile:tempURL fromPath:YHBaseCecheImage];
                    if (ceche) {
                        UIImage * tempImg = [UIImage imageWithData:ceche];
                        component.img=tempImg;
                    }else{
                    //在分线程中进行图片数据的获取
                        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
                            if (tempURL) {
                                NSData * data = [YHBaseData getDataWithUrl:tempURL];
                                if (data) {
                                //获取完成后村缓存
                                    //做缓存
                                    [[YHBaseCecheCenter sharedTheSingletion]writeCecheFile:data withFileID:tempURL toPath:YHBaseCecheImage];
                                    //赋值 回调代理
                                    UIImage * tempImg = [UIImage imageWithData:data];
                                    component.img=tempImg;
                                    //这里代理是我添加的，当图片下载完成后 通知视图重新排版
                                    if ([[rcLabel imageDelegate]respondsToSelector:@selector(YHRTLabelImageSuccess:)]) {
                                        //在主线程中执行回调
                                        //这个地方要在主线程中执行，否则刷新会有延时
                                        dispatch_async(dispatch_get_main_queue(), ^{
                                             [[rcLabel imageDelegate] YHRTLabelImageSuccess:rcLabel];
                                        });
                            
                                    }
                                   
                                }
                                
                            };
                            
                        });
                    }
                   
                    
                }
```

通过如上简单的扩展，基本达到了项目中的需求，这里把我的一些想法和思路分享给大家，有更好的解决方案，或者同是开发爱好者，欢迎指点与交流，我的QQ是316045346。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

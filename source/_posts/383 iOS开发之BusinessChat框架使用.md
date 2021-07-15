---
title: iOS开发之BusinessChat框架使用
date: 2018-10-12
categories: iOS逻辑初窥
tags: []
---
## iOS开发之BusinessChat框架使用

      BusinessChat是iOS11.3后引入的新框架，这个框架配合iMessage应用将商家与用户更加紧密的结合起来，并且为商家提供了另外一种非常方便的客服系统。

      我们知道，在iOS10中新引入了iMessage扩展，iMessage扩展除了丰富了表情包外，开发者也可以开发一些功能独立的iMessage应用，关于iMessage扩展的相关应用，如下博客中有着完整的介绍。

[https://my.oschina.net/u/2340880/blog/749331](https://my.oschina.net/u/2340880/blog/749331)

     随着iMessage扩展使得iMessage功能的越来越强大，其为用户提供能力和与第三方APP交互能力也越来越强，BusinessChat框架是提供给应用程序调用iMessage来与商家的客服系统联系的功能框架。

    许多服务类的应用程序都有客服系统，例如当用户使用电商类应用程序时通常会需要联系商家。要使用BusinessChat相关功能，首先需要注册成为Apple商家，在如下网站进行商家注册：

[https://register.apple.com](https://register.apple.com)

界面如下：

![](https://oscimg.oschina.net/oscnet/5c192fbbb955071e0ec88c685e2b8d8bc3e.jpg)

使用AppleID登录后，填写必要的商家信息和成员信息，即可进行申请，提交申请后，需要Apple进行审核，如果审核通过会分配商户ID给我们，我们需要使用这个商户ID来进行我们的开发。

    BusinessChat框架中有两个类：BCChatButton类和BCChatAction类，BCChatButton类是单纯的UI支持类，它提供了同意的联系客服按钮样式，BCChatAction类用来处理行为逻辑。示例代码如下：

```objectivec
@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    BCChatButton * btn = [[BCChatButton alloc]initWithStyle:BCChatButtonStyleDark];
    btn.frame = CGRectMake(50, 100, 200,100);
    [btn addTarget:self action:@selector(click) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:btn];
}

-(void)click{
    NSLog(@"message");
    [BCChatAction openTranscript:@"8d7f4b79-bf77-45ab-86b5-b74f56d47737" intentParameters:@{BCParameterNameIntent:@"buy",BCParameterNameGroup:@"custom",BCParameterNameBody:@"Hello World"}];
}


@end
```

运行代码，按钮样式如下图：

![](https://oscimg.oschina.net/oscnet/b91363b71d46c32bde9a0192e130932786b.jpg)

点击按钮后，会调起iMessage应用，用户可以直接与商户进客服行联系。

      BCChatButton是一个纯UI的按钮类，其继承自UIControl，使用方式和正常的UIButton一样，需要注意，其中并没有封装交互逻辑，按钮的触发事件需要开发者自己定义。BCChatAction来进行交互逻辑的处理，这个类中只有一个方法，如下：

```objectivec
/*
businessIdentifier为商户ID
intentParameters为意图参数字典，其中可定义键值如下：
BCParameterNameIntent 定义意图 用户发送消息时可以让商户更清楚用户的问题领域
BCParameterNameGroup 定义组 帮助商户将问题分发明确的组 
BCParameterNameBody 信息内容
*/
+ (void)openTranscript:(NSString *)businessIdentifier
      intentParameters:(NSDictionary<BCParameterName, NSString *> *)intentParameters;
```

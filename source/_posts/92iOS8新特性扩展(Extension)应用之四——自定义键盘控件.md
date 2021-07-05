---
title: iOS8新特性扩展(Extension)应用之四——自定义键盘控件
date: 2015-07-31
categories: iOS逻辑初窥
tags: []
---
## iOS8新特性扩展(Extension)应用之四——自定义键盘控件

        iOS8系统的开放第三方键盘，使得用户在输入法的选择上更加自主灵活，也更加贴近不同语言的输入风格。这篇博客，将介绍如何开发一个第三方的键盘控件。

### 一、了解UIInputViewController类

        UIInputViewController是系统扩展支持键盘扩展的一个类，通过这个类，我们可以自定义一款我们自己的键盘提供给系统使用。

        首先，我们先来看一下这个类中的一些属性和方法：

[@property](http://my.oschina.net/property) (nonatomic, retain) UIInputView *inputView;

键盘的输入视图，我们可以自定义这个视图。

[@property](http://my.oschina.net/property) (nonatomic, readonly) NSObject <UITextDocumentProxy> *textDocumentProxy;

实现了UITextDocumentProxy协议的一个对象，后面会介绍这个协议。

[@property](http://my.oschina.net/property) (nonatomic, copy) NSString *primaryLanguage;

系统为我们准备了一些本地化的语言字符串

\- (void)dismissKeyboard;

收键盘的方法

\- (void)advanceToNextInputMode;

切换到下一输入法的方法

UITextDocumentProxy协议内容如下：

```
@protocol UITextDocumentProxy <UIKeyInput>
//输入的上一个字符
@property (nonatomic, readonly) NSString *documentContextBeforeInput;
//即将输入的一个字符
@property (nonatomic, readonly) NSString *documentContextAfterInput;
//将输入的字符移动到某一位置
- (void)adjustTextPositionByCharacterOffset:(NSInteger)offset;

@end
```

而UITextDocumentProxy这个协议继承与UIKeyInput协议，UIKeyInput协议中提供的两个方法用于输入字符和删除字符：

\- (void)insertText:(NSString *)text;  
\- (void)deleteBackward;

### 二、创建一款最简单的数字输入键盘

        创建一个项目，作为宿主APP，接着我们File->new->target->customKeyBoard:

![](http://static.oschina.net/uploads/space/2015/0731/105006_AFk6_2340880.png)

系统要求我们对键盘的布局要使用autolayout，并且只可以采用代码布局的方式，我们这里为了简单演示，将坐标写死：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 设置数字键盘的UI
    //数字按钮布局
    for (int i=0; i<10; i++) {
        UIButton * btn = [UIButton buttonWithType:UIButtonTypeSystem];
        btn.frame=CGRectMake(20+45*(i%3), 20+45*(i/3), 40, 40);
        btn.backgroundColor=[UIColor greenColor];
        [btn setTitle:[NSString stringWithFormat:@"%d",i] forState:UIControlStateNormal];
        btn.tag=101+i;
        [btn addTarget:self action:@selector(click:) forControlEvents:UIControlEventTouchUpInside];
        [self.view addSubview:btn];
    }
    //创建切换键盘按钮
    UIButton * change = [UIButton buttonWithType:UIButtonTypeSystem];
    change.frame=CGRectMake(200,20, 80, 40) ;
    NSLog(@"%f,%f",self.view.frame.size.height,self.view.frame.size.width);
    [change setBackgroundColor:[UIColor blueColor]];
    [change setTitle:@"切换键盘" forState:UIControlStateNormal];
    [change addTarget:self action:@selector(change) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:change];
    //创建删除按钮
    UIButton * delete = [UIButton buttonWithType:UIButtonTypeSystem];
    delete.frame=CGRectMake(200, 120, 80, 40);
    [delete setTitle:@"delete" forState:UIControlStateNormal];
    [delete setBackgroundColor:[UIColor redColor]];
    [delete addTarget:self action:@selector(delete) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:delete];
}
//删除方法
-(void)delete{
    if (self.textDocumentProxy.documentContextBeforeInput) {
        [self.textDocumentProxy deleteBackward];
    }
}
//切换键盘方法
-(void)change{
    [self advanceToNextInputMode];
}
//点击数字按钮 将相应数字输入
-(void)click:(UIButton *)btn{
    [self.textDocumentProxy insertText:[NSString stringWithFormat:@"%ld",btn.tag-101]];
}
```

运行后，在使用之前，我们需要先加入这个键盘：在模拟器系统设置中general->keyboard->keyboards->addNowKeyboard

选中我们自定义的键盘，之后运行浏览器，切换到我们的键盘，效果如下：

![](http://static.oschina.net/uploads/space/2015/0731/105628_dKoe_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

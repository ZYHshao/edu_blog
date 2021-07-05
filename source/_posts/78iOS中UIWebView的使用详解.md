---
title: iOS中UIWebView的使用详解
date: 2015-06-23
categories: iOS之UI控件
tags: []
---
## iOS中UIWebView的使用详解

### 一、初始化与三种加载方式

     UIWebView继承与UIView，因此，其初始化方法和一般的view一样，通过alloc和init进行初始化，其加载数据的方式有三种：

第一种：

\- (void)loadRequest:(NSURLRequest *)request;

这是加载网页最常用的一种方式，通过一个网页URL来进行加载，这个URL可以是远程的也可以是本地的，例如我加载百度的主页：

```
    UIWebView * view = [[UIWebView alloc]initWithFrame:self.view.frame];
    [view loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.baidu.com"]]];
    [self.view addSubview:view];
```

会得到如下的效果：

![](http://static.oschina.net/uploads/space/2015/0623/212246_MjT6_2340880.png)

第二种：

\- (void)loadHTMLString:(NSString *)string baseURL:(NSURL *)baseURL;

这个方法需要将httml文件读取为字符串，其中baseURL是我们自己设置的一个路径，用于寻找html文件中引用的图片等素材。

第三种：

\- (void)loadData:(NSData *)data MIMEType:(NSString *)MIMEType textEncodingName:(NSString *)textEncodingName baseURL:(NSURL *)baseURL;

这个方式使用的比较少，但也更加自由，其中data是文件数据，MIMEType是文件类型，textEncodingName是编码类型，baseURL是素材资源路径。

### 二、一些常用的属性和变量

[@property](http://my.oschina.net/property) (nonatomic, assign) id <UIWebViewDelegate\> delegate;

设置webView的代理

[@property](http://my.oschina.net/property) (nonatomic, readonly, retain) UIScrollView *scrollView;

内置的scrollView

[@property](http://my.oschina.net/property) (nonatomic, readonly, retain) NSURLRequest *request;

URL请求

\- (void)reload;

重新加载数据

\- (void)stopLoading;

停止加载数据

\- (void)goBack;

返回上一级

\- (void)goForward;

跳转下一级

[@property](http://my.oschina.net/property) (nonatomic, readonly, getter=canGoBack) BOOL canGoBack;

获取能否返回上一级

[@property](http://my.oschina.net/property) (nonatomic, readonly, getter=canGoForward) BOOL canGoForward;

获取能否跳转下一级

@property (nonatomic, readonly, getter=isLoading) BOOL loading;

获取是否正在加载数据

\- (NSString *)stringByEvaluatingJavaScriptFromString:(NSString *)script;

通过javaScript操作web数据

@property (nonatomic) BOOL scalesPageToFit;

设置是否缩放到适合屏幕大小

@property (nonatomic) UIDataDetectorTypes dataDetectorTypes NS\_AVAILABLE\_IOS(3_0);

设置某些数据变为链接形式，这个枚举可以设置如电话号，地址，邮箱等转化为链接

@property (nonatomic) BOOL allowsInlineMediaPlayback NS\_AVAILABLE\_IOS(4_0);

设置是否使用内联播放器播放视频

@property (nonatomic) BOOL mediaPlaybackRequiresUserAction NS\_AVAILABLE\_IOS(4_0);

设置视频是否自动播放

@property (nonatomic) BOOL mediaPlaybackAllowsAirPlay NS\_AVAILABLE\_IOS(5_0);

设置音频播放是否支持ari play功能

@property (nonatomic) BOOL suppressesIncrementalRendering NS\_AVAILABLE\_IOS(6_0);

设置是否将数据加载如内存后渲染界面

@property (nonatomic) BOOL keyboardDisplayRequiresUserAction NS\_AVAILABLE\_IOS(6_0);

设置用户交互模式

### 三、iOS7中的一些新特性

下面这些属性是iOS7之后才有的，通过他们可以设置更加有趣的web体验

@property (nonatomic) UIWebPaginationMode paginationMode NS\_AVAILABLE\_IOS(7_0);

这个属性用来设置一种模式，当网页的大小超出view时，将网页以翻页的效果展示，枚举如下：

```
typedef NS_ENUM(NSInteger, UIWebPaginationMode) {
    UIWebPaginationModeUnpaginated,//不使用翻页效果
    UIWebPaginationModeLeftToRight,//将网页超出部分分页，从左向右进行翻页
    UIWebPaginationModeTopToBottom,//将网页超出部分分页，从上向下进行翻页
    UIWebPaginationModeBottomToTop,//将网页超出部分分页，从下向上进行翻页
    UIWebPaginationModeRightToLeft//将网页超出部分分页，从右向左进行翻页
};
```

@property (nonatomic) CGFloat pageLength NS\_AVAILABLE\_IOS(7_0);

设置每一页的长度

@property (nonatomic) CGFloat gapBetweenPages NS\_AVAILABLE\_IOS(7_0);

设置每一页的间距

@property (nonatomic, readonly) NSUInteger pageCount NS\_AVAILABLE\_IOS(7_0);

获取分页数

### 四、webView协议中的方法

\- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType;

准备加载内容时调用的方法，通过返回值来进行是否加载的设置

\- (void)webViewDidStartLoad:(UIWebView *)webView;

开始加载时调用的方法

\- (void)webViewDidFinishLoad:(UIWebView *)webView;

结束加载时调用的方法

\- (void)webView:(UIWebView *)webView didFailLoadWithError:(NSError *)error;

加载失败时调用的方法

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

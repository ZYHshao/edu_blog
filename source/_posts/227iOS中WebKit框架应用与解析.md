---
title: iOS中WebKit框架应用与解析
date: 2016-06-24
categories: iOS逻辑初窥
tags: []
---
## iOS中WebKit框架应用与解析

### 一、引言

        在iOS8之前，在应用中嵌入网页通常需要使用UIWebView这样一个类，这个类通过URL或者HTML文件来加载网页视图，功能十分有限，只能作为辅助嵌入原生应用程序中。虽然UIWebView也可以做原生与JavaScript交互的相关处理，然而也有很大的局限性，JavaScript要调用原生方法通常需要约定好协议之后通过Request来传递。WebKit框架中添加了一些原生与JavaScript交互的方法，增强了网页视图与原生的交互能力。并且WebKit框架中采用导航堆栈的模型来管理网页的跳转，开发者也可以更加容易的控制和管理网页的渲染。关于UIWebView的相关使用，在前面的博客中有详细介绍，地址如下。

UIWebView的使用详解：[http://my.oschina.net/u/2340880/blog/469916](http://my.oschina.net/u/2340880/blog/469916)。

### 二、WebKit框架概览

        WebKit框架中涉及的类很多，框架的设计十分面向对象和模块化，开发者在使用时可以轻松的写出结构清晰的代码。在进行使用前，我们首先应该清楚整个框架的结构和开发思路，下面一张脑图中基本列出了WebKit框架中所涉及到的所有重要的类以及他们之间的相互关系：

![](http://static.oschina.net/uploads/space/2016/0624/104532_WAVb_2340880.png)如上图所示，WebKit框架中最核心的类应该属于WKWebView了，这个类专门用来渲染网页视图，其他类和协议都将基于它和服务于它。

WKWebView：网页的渲染与展示，通过WKWebViewConfiguration可以进行配置。

WKWebViewConfiguration：这个类专门用来配置WKWebView。

WKPreference:这个类用来进行M相关设置。

WKProcessPool：这个类用来配置进程池，与网页视图的资源共享有关。

WKUserContentController：这个类主要用来做native与JavaScript的交互管理。

WKUserScript：用于进行JavaScript注入。

WKScriptMessageHandler：这个类专门用来处理JavaScript调用native的方法。

WKNavigationDelegate：网页跳转间的导航管理协议，这个协议可以监听网页的活动。

WKNavigationAction：网页某个活动的示例化对象。

WKUIDelegate：用于交互处理JavaScript中的一些弹出框。

WKBackForwardList：堆栈管理的网页列表。

WKBackForwardListItem：每个网页节点对象。

### 三、使用WKWebViewConfiguration对WebView进行配置

        使用下面的代码可以创建一个WKWebView视图，创建WebView视图时，需要使用WKWebViewConfiguration来进行配置：

```objectivec
    WKWebView * WK;
    WKWebViewConfiguration * config = [[WKWebViewConfiguration alloc]init];
    WK = [[WKWebView alloc]initWithFrame:CGRectMake(0, 0, self.view.frame.size.width, self.view.frame.size.height-40) configuration:config];
    [WK loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.baidu.com"]]];
```

WKWebViewConfiguration中可以进行配置的方法和属性如下：

```objectivec
    //设置进程池
    WKProcessPool * pool = [[WKProcessPool alloc]init];
    config.processPool = pool;
```

WKProcessPool类中没有暴露任何属性和方法，配置为同一个进程池的WebView会共享数据，例如Cookie、用户凭证等，开发者可以通过编写管理类来分配不同维度的WebView在不同进程池中。

```
    //进行偏好设置
    WKPreferences * preference = [[WKPreferences alloc]init];
    //最小字体大小 当将javaScriptEnabled属性设置为NO时，可以看到明显的效果
    preference.minimumFontSize = 0;
    //设置是否支持javaScript 默认是支持的
    preference.javaScriptEnabled = YES;
    //设置是否允许不经过用户交互由javaScript自动打开窗口
    preference.javaScriptCanOpenWindowsAutomatically = YES;
    config.preferences = preference;
```

WKPerference实例为WebView提供一个偏好设置。

```objectivec
    //设置内容交互控制器 用于处理JavaScript与native交互
    WKUserContentController * userController = [[WKUserContentController alloc]init];
    //设置处理代理并且注册要被js调用的方法名称
    [userController addScriptMessageHandler:self name:@"name"];
    //js注入，注入一个测试方法。
    NSString *javaScriptSource = @"function userFunc(){window.webkit.messageHandlers.name.postMessage( {\"name\":\"HS\"})}";
    WKUserScript *userScript = [[WKUserScript alloc] initWithSource:javaScriptSource injectionTime:WKUserScriptInjectionTimeAtDocumentStart forMainFrameOnly:YES];// forMainFrameOnly:NO(全局窗口)，yes（只限主窗口）
    [userController addUserScript:userScript];
    config.userContentController = userController;

```

WKUserContentController专门用来管理native与JavaScript的交互行为，addScriptMessageHandler:name:方法来注册要被js调用的方法名称，之后再JavaScript中使用window.webkit.messageHandlers.name.postMessage()方法来像native发送消息，支持OC中字典，数组，NSNumber等原生数据类型，JavaScript代码中的name要和上面注册的相同。在native代理的回调方法中，会获取到JavaScript传递进来的消息，如下：

```objectivec
-(void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message{
    //这里可以获取到JavaScript传递进来的消息
}
```

WKScriptMessage类是JavaScript传递的对象实例，其中属性如下：

```objectivec
//传递的消息主体
@property (nonatomic, readonly, copy) id body;
//传递消息的WebView
@property (nullable, nonatomic, readonly, weak) WKWebView *webView;
//传递消息的WebView当前页面对象
@property (nonatomic, readonly, copy) WKFrameInfo *frameInfo;
//消息名称
@property (nonatomic, readonly, copy) NSString *name;
```

WKUserContentController实例的addUserScript:用于注入JavaScript代码，后面会专门介绍。

```objectivec
    //设置数据存储store
    config.websiteDataStore = [WKWebsiteDataStore defaultDataStore];
```

WebKit框架采用其本身的缓存框架，WKWebsiteDataStore类用来处理数据的存储，其中属性和方法如下：

```objectivec
@interface WKWebsiteDataStore : NSObject
//获取默认的存储器 此存储器为持久性的会被写入磁盘
+ (WKWebsiteDataStore *)defaultDataStore;
//获取一个临时的存储器
+ (WKWebsiteDataStore *)nonPersistentDataStore;
//存储器是否是临时的
@property (nonatomic, readonly, getter=isPersistent) BOOL persistent;
//所有可以存储的类型
+ (NSSet<NSString *> *)allWebsiteDataTypes;
@end
```

```objectivec
    //设置是否将网页内容全部加载到内存后再渲染
    config.suppressesIncrementalRendering = NO;
    //设置HTML5视频是否允许网页播放 设置为NO则会使用本地播放器
    config.allowsInlineMediaPlayback =  YES;
    //设置是否允许ariPlay播放
    config.allowsAirPlayForMediaPlayback = YES;
    //设置视频是否需要用户手动播放  设置为NO则会允许自动播放
    config.requiresUserActionForMediaPlayback = NO;
    //设置是否允许画中画技术 在特定设备上有效
    config.allowsPictureInPictureMediaPlayback = YES;
    //设置选择模式 是按字符选择 还是按模块选择
    /*
    typedef NS_ENUM(NSInteger, WKSelectionGranularity) {
        //按模块选择
        WKSelectionGranularityDynamic,
        //按字符选择
        WKSelectionGranularityCharacter,
    } NS_ENUM_AVAILABLE_IOS(8_0);
    */
    config.selectionGranularity = WKSelectionGranularityCharacter;
    //设置请求的User-Agent信息中应用程序名称 iOS9后可用
    config.applicationNameForUserAgent = @"HS";
```

### 四、WKWebView中的属性和方法解析

        下面列举了WKWebView中常用的属性和方法。

```
//设置导航代理
@property (nullable, nonatomic, weak) id <WKNavigationDelegate> navigationDelegate;
//设置UI代理
@property (nullable, nonatomic, weak) id <WKUIDelegate> UIDelegate;
//导航列表
@property (nonatomic, readonly, strong) WKBackForwardList *backForwardList;
//通过url加载网页视图
- (nullable WKNavigation *)loadRequest:(NSURLRequest *)request;
//通过文件加载网页视图
- (nullable WKNavigation *)loadFileURL:(NSURL *)URL allowingReadAccessToURL:(NSURL *)readAccessURL NS_AVAILABLE(10_11, 9_0);
//通过HTML字符串加载网页视图
- (nullable WKNavigation *)loadHTMLString:(NSString *)string baseURL:(nullable NSURL *)baseURL;
//通过data数据加载网页视图
- (nullable WKNavigation *)loadData:(NSData *)data MIMEType:(NSString *)MIMEType characterEncodingName:(NSString *)characterEncodingName baseURL:(NSURL *)baseURL NS_AVAILABLE(10_11, 9_0);
//渲染导航列表中的某个网页节点
- (nullable WKNavigation *)goToBackForwardListItem:(WKBackForwardListItem *)item;
//网页标题
@property (nullable, nonatomic, readonly, copy) NSString *title;
//网页的url
@property (nullable, nonatomic, readonly, copy) NSURL *URL;
//网页是否正在加载中
@property (nonatomic, readonly, getter=isLoading) BOOL loading;
//加载进度 可以监听这个属性的值配合UIProgressView来设计进度条
@property (nonatomic, readonly) double estimatedProgress;
//是否全部是安全连接
@property (nonatomic, readonly) BOOL hasOnlySecureContent;
//证书列表
@property (nonatomic, readonly, copy) NSArray *certificateChain;
//是否可以回退
@property (nonatomic, readonly) BOOL canGoBack;
//是否可以前进
@property (nonatomic, readonly) BOOL canGoForward;
//回退网页
- (nullable WKNavigation *)goBack;
//前进网页
- (nullable WKNavigation *)goForward;
//刷新网页
- (nullable WKNavigation *)reload;
//忽略缓存的刷新
- (nullable WKNavigation *)reloadFromOrigin;
//停止加载
- (void)stopLoading;
//执行JavaScript代码
- (void)evaluateJavaScript:(NSString *)javaScriptString completionHandler:(void (^ __nullable)(__nullable id, NSError * __nullable error))completionHandler;
//是否允许右滑返回手势
@property (nonatomic) BOOL allowsBackForwardNavigationGestures;
```

WKBackForwardList类为导航管理的网页列表类，其中属性方法意义如下：

```objectivec
@interface WKBackForwardList : NSObject
//当前所在的网页节点
@property (nullable, nonatomic, readonly, strong) WKBackForwardListItem *currentItem;
//前进的一个网页节点
@property (nullable, nonatomic, readonly, strong) WKBackForwardListItem *forwardItem;
//回退的一个网页节点
@property (nullable, nonatomic, readonly, strong) WKBackForwardListItem *backItem;
//获取某个index的网页节点
- (nullable WKBackForwardListItem *)itemAtIndex:(NSInteger)index;
//获取回退的节点数组
@property (nonatomic, readonly, copy) NSArray<WKBackForwardListItem *> *backList;
//获取前进的节点数组
@property (nonatomic, readonly, copy) NSArray<WKBackForwardListItem *> *forwardList;
@end
```

在WebKit中，网页节点被抽象成为了WKBackForwardListItem类，这个类中封装的属性如下：

```objectivec
@interface WKBackForwardListItem : NSObject
//当前节点的URL
@property (readonly, copy) NSURL *URL;
//当前节点的标题
@property (nullable, readonly, copy) NSString *title;
//创建此WebView的初始URL
@property (readonly, copy) NSURL *initialURL;
```

### 五、关于native与JavaScript交互

        WebKit中的native与JavaScript的交互主要有4类。

#### 1.JavaScript调用native方法

        这种方式是由WKUserContentController注册，并在代理方法中实现的。

#### 2.native调用JavaScript方法

        这种方式通过WKWebView直接调用evaluteJavaScript:completionHandler:方法来实现。

#### 3.将JavaScript代码注入

        这种方式可以在网页中注入一些自定义的JavaScript代码，也可以注入自定义的方法，再使用evaluteJavaScript:completionHandler:来调用方法。JavaScript代码的注入也是通过WKUserContentController来完成的，使用addUserScript:方法来注入JavaScript，其中需要通过WKUserScript类来生成要注入的对象，这个类使用如下方法来进行实例化：

```objectivec
/*
source为要注入的js代码
WKUserScriptInjectionTime设置注入的时机
forMainFrameOnly参数设置是否只在主页面注入
typedef NS_ENUM(NSInteger, WKUserScriptInjectionTime) {
    //原js代码运行前注入
    WKUserScriptInjectionTimeAtDocumentStart,
    //原js代码运行后注入
    WKUserScriptInjectionTimeAtDocumentEnd
} NS_ENUM_AVAILABLE(10_10, 8_0);

*/
- (instancetype)initWithSource:(NSString *)source injectionTime:(WKUserScriptInjectionTime)injectionTime forMainFrameOnly:(BOOL)forMainFrameOnly;
```

#### 4.通过WKUIDelegate来交互

        这种方式主要用于相应JavaScript中的弹出框，后面会详细介绍这个协议。

### 六、WKNavagationDelegate中方法解析

        WKNavagationDelegate协议重要有两个作用，监听页面渲染流程与控制页面跳转，其中方法如下：

```objectivec
/*
决定是否响应网页的某个动作，例如加载，回退，前进，刷新等，在这个方法中，必须执行decisionHandler()代码块，并将是否允许这个活动执行在block中进行传入
*/
/*
WKNavigationAction是网页动作的抽象化，其中封装了许多行为信息，后面会介绍
WKNavigationActionPolicy为开发者回执，枚举如下：
typedef NS_ENUM(NSInteger, WKNavigationActionPolicy) {
    //取消此次行为
    WKNavigationActionPolicyCancel,
    //允许此次行为
    WKNavigationActionPolicyAllow,
} NS_ENUM_AVAILABLE(10_10, 8_0);
*/
-(void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler{
    decisionHandler(WKNavigationActionPolicyAllow);
}
//需要响应身份验证时调用 同样在block中需要传入用户身份凭证
-(void)webView:(WKWebView *)webView didReceiveAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition, NSURLCredential * _Nullable))completionHandler{
    //用户身份信息
    NSURLCredential *newCred = [NSURLCredential credentialWithUser:@""
                                                          password:@""
                                                       persistence:NSURLCredentialPersistenceNone];
    // 为 challenge 的发送方提供 credential
    [[challenge sender] useCredential:newCred
           forAuthenticationChallenge:challenge];
    completionHandler(NSURLSessionAuthChallengeUseCredential,newCred);
}
//接收到数据后是否允许执行渲染
/*
其中，WKNavigationResponse为请求回执信息
WKNavigationResponsePokicy为开发者回执，枚举如下：
typedef NS_ENUM(NSInteger, WKNavigationResponsePolicy) {
    //取消渲染
    WKNavigationResponsePolicyCancel,
    //允许渲染
    WKNavigationResponsePolicyAllow,
} NS_ENUM_AVAILABLE(10_10, 8_0);
*/
-(void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler{
    decisionHandler(WKNavigationResponsePolicyAllow);
}
//=====================下面这个协议方法用于监听流程=========================================
//页面加载启动时调用
-(void)webView:(WKWebView *)webView didStartProvisionalNavigation:(WKNavigation *)navigation{

}
//当主机接收到的服务重定向时调用
-(void)webView:(WKWebView *)webView didReceiveServerRedirectForProvisionalNavigation:(WKNavigation *)navigation{

}
//内容到达主机时调用
-(void)webView:(WKWebView *)webView didCommitNavigation:(WKNavigation *)navigation{

}
//主页加载完成时调用
-(void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation{

}
//提交发生错误时调用
-(void)webView:(WKWebView *)webView didFailNavigation:(WKNavigation *)navigation withError:(NSError *)error{

}
//主页数据加载发生错误时调用
-(void)webView:(WKWebView *)webView didFailProvisionalNavigation:(null_unspecified WKNavigation *)navigation withError:(nonnull NSError *)error{

}
//进程被终止时调用
-(void)webViewWebContentProcessDidTerminate:(WKWebView *)webView{

}
```

### 七、WKUIDelegate协议中方法解析

```objectivec
//创建新的webView时调用的方法
-(WKWebView *)webView:(WKWebView *)webView createWebViewWithConfiguration:(WKWebViewConfiguration *)configuration forNavigationAction:(WKNavigationAction *)navigationAction windowFeatures:(WKWindowFeatures *)windowFeatures{
    return webView;
}
//关闭webView时调用的方法
-(void)webViewDidClose:(WKWebView *)webView{

}
//下面这些方法是交互JavaScript的方法
//JavaScript调用alert方法后回调的方法 message中为alert提示的信息 必须要在其中调用completionHandler()
-(void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler{
    NSLog(@"%@",message);
    completionHandler();
}
//JavaScript调用confirm方法后回调的方法 confirm是js中的确定框，需要在block中把用户选择的情况传递进去
-(void)webView:(WKWebView *)webView runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(BOOL))completionHandler{
    NSLog(@"%@",message);
    completionHandler(YES);
}
//JavaScript调用prompt方法后回调的方法 prompt是js中的输入框 需要在block中把用户输入的信息传入
-(void)webView:(WKWebView *)webView runJavaScriptTextInputPanelWithPrompt:(NSString *)prompt defaultText:(NSString *)defaultText initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(NSString * _Nullable))completionHandler{
    NSLog(@"%@",prompt);
    completionHandler(@"123");
}

```

### 八、扩展

        首先，在注册要被JavaScript调用的方法时需要设置代理，在不需要时需要将代理移除，WKUserContentController中也提供了移除这个代理的方法，如果不移除，将会造成WebView不能释放。方法如下：

```objectivec
//注册一个监听方法
- (void)addScriptMessageHandler:(id <WKScriptMessageHandler>)scriptMessageHandler name:(NSString *)name;
//移除一个方法的监听
- (void)removeScriptMessageHandlerForName:(NSString *)name;

```

同样与注入JavaScript对应，也可以将注入的代码移除，方法如下：

```objectivec
//注入一个JavaScript抽象对象
- (void)addUserScript:(WKUserScript *)userScript;
//移除所有注入
- (void)removeAllUserScripts;

```

        在上面，经常会见到WKNavagationAction这个类，这个类中封装的是一些页面活动信息，如下：

```objectivec
@interface WKNavigationAction : NSObject
//原页面
@property (nonatomic, readonly, copy) WKFrameInfo *sourceFrame;
//目标页面
@property (nullable, nonatomic, readonly, copy) WKFrameInfo *targetFrame;
//请求URL
@property (nonatomic, readonly, copy) NSURLRequest *request;
//活动类型
/*
typedef NS_ENUM(NSInteger, WKNavigationType) {
    //链接激活
    WKNavigationTypeLinkActivated,
    //提交操作
    WKNavigationTypeFormSubmitted,
    //前进操作
    WKNavigationTypeBackForward,
    //刷新操作
    WKNavigationTypeReload,
    //重提交操作 例如前进 后退 刷新
    WKNavigationTypeFormResubmitted,
    //其他类型
    WKNavigationTypeOther = -1,
} NS_ENUM_AVAILABLE(10_10, 8_0);
*/
@property (nonatomic, readonly) WKNavigationType navigationType;
@end
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

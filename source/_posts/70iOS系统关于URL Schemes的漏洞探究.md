---
title: iOS系统关于URL Schemes的漏洞探究
date: 2015-06-02
categories: iOS逻辑初窥
tags: []
---
## iOS系统关于URL Schemes的漏洞探究

### 一、何为URL Schemes

    我想这个东西的设计的目的是为了方便App之间的相互调用与通讯，你可以在自己的App中使用OpenURL方法来唤起其他的App。比如微信的URL Schemes是wiexin，我们新建一个工程，实现如下代码后运行程序：

```
[[UIApplication sharedApplication]openURL:[NSURL URLWithString:@"weixin://]];
```

这时你会发现，你的应用启动后很快就调起了微信的客户端。

### 二、由URL Schemes引发的漏洞的根源

#### 1、一个小问题引起的漏洞根源

    如上所说，通过URL Schemes可以在应用间相互唤起，而产生漏洞的根源在于这个URL并非是应用唯一的。apple并没有任何限制或者审核这个URL的任何措施，也就是说，如果两个App有着相同的URL Schemes，那么系统唤起的App可能并不是你想唤起的。

### 2、URL Schemes的优先级如何确定

    由于相同的URL Scheme可能同时被多个App使用，再如果这些App都安装在了同一个设备上，那么系统究竟会唤起哪一个呢？这个我也不能十分的确定，只有一点可以肯定：如果有和系统应用的URL Scheme相同，那么系统一定会唤起系统自己的应用，在这里系统的应用有着最高的优先级（苹果这里做的好像很不厚道，将自己的应用保护了起来，而把广大其他开发者的应用放在漏洞前置之不理）。如果没有和系统耦合的，那么系统会唤起哪一个App就看运气了。不过，这也不是无章可循，经过测试，优先级和App的Bundle identifier有关，更准确说和Bundle identifier的字母排序有关，如果精心设计这个id，我们就可以做到截获其他应用的URL。

### 3、这个漏洞会引发什么问题么？

    仅仅通过上面的叙述，你可能还看不出这个漏洞会引发什么样的后果。可是如果你仔细观察，你会发现，各种iPhone上的第三方调用，例如QQ音乐快捷登录，腾讯的各种游戏，甚至包括调用支付宝钱包的支付功能，都是通过这样的原理实现的。如果这些回调的数据被截获，那么就等于说登录信息，用户信息甚至支付订单信息都会暴漏在他人眼下，对于截获者来说，他可以用你的信息进行登录，可以替你完成支付，也可以盗取你登陆后的用户信息。

### 三、利用URL Scheme漏洞进行远程登录

    下面，就用一个实例来演示一下我如何通过一个伪装App登录天天炫斗账号。

    天天炫斗是腾讯的一款十分火爆的格斗游戏，像其他腾讯游戏一样，支持QQ和微信登录，这里我拿微信登录为例。

    首先，我们需要做一个伪装的App来截取用户的登录信息，新建一个项目，在plist文件中添加一个和天天炫斗微信登录相同的URL Scheme：

![](http://static.oschina.net/uploads/space/2015/0602/143732_Wvyl_2340880.png)

这里的wx63124814f356e266就是微信登录天天炫斗的URL Scheme，这里将Bundle id设置为A，使它有比天天炫斗更高的优先级。

在AppDelegate中添加如下代码：

```
-(BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation{
    NSLog(@"%@",url);
    return YES;
}
```

这个函数是在App被通过URL唤起时首先调用的函数，这里传入的URL就是用户的登录验证信息，我们可以在这里将这个信息发送回来。

将伪装好的程序跑一遍后，运行天天炫斗，然后使用微信登录，会发现在微信验证成功后跳转后并没有跳转回天天炫斗应用，而是跳转到了我们伪装的这个Demo。这时xcode调试区会打印出如下的信息：

![](http://static.oschina.net/uploads/space/2015/0602/144313_cppe_2340880.png)

之后，来开始做我们的侵入程序，这个其实更加简单，新建一个工程，只需要添加一行代码：

```
[[UIApplication sharedApplication]openURL:[NSURL URLWithString:@"wx63124814f356e266://oauth?code=0118aa2f2b99d8a9e0e76a7176b2bd4E&state=weixin"]];
```

这里的URL就是我们截获的带参的URL，在另一个装有天天炫斗的手机上跑这个程序（在同一个手机上测试的话要将刚才的伪装App删去，不然它也会将我们的侵入程序一起骗了）。会发现登录天天炫斗成功，角色信息完全一致。

    同样的做法，还可以远程登录QQ音乐，天天飞车等等各种通过微信，QQ，微博快捷登录的应用。

### 四、要战胜你的敌人，必须要了解你的敌人

    不了解apple为什么一直不对URL Scheme做限制，或许需要或许不需要。但是这一点建议总是好的：在你的App使用快捷登录的时候，最好同时将设备号或者某个本地保存的标志绑定，防止恶意的第三方借此获取用户的信息。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

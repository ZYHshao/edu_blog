---
title: iOS应用内评价与购买三方APP组件
date: 2017-08-21
categories: iOS之UI控件
tags: []
---
## iOS应用内评价与购买三方APP组件

    首先来说应用内评价组件，应用内评价组件是iOS10.3中新引入的功能。其封装在StoreKit框架中。用户可以直接在APP内唤起评价组件对应用程序进行评星，示例代码如下：

```objectivec
[SKStoreReviewController requestReview];
```

效果如下图：

![](https://static.oschina.net/uploads/space/2017/0821/222356_b5SD_2340880.png)

在模拟器上，这个Submit按钮是不可点击的，如果在真机上，并且应用程序已经上线，可以直接进行评价。这个方便的评价组件可以避免让用户跳出APP进行评价的不好体验。

    SKStoreReviewController中只有requestReview这一个类方法，需要注意，只有在iOS10.3后才可以使用。但是StoreKit这个框架很早就有了。里面还有一个类可以让用户直接在应用内打开一个第三方应用的AppStore购买页。示例代码如下：

```objectivec
    SKStoreProductViewController * controller = [[SKStoreProductViewController alloc]init];
    [self presentViewController:controller animated:YES completion:nil];
    [controller loadProductWithParameters:@{SKStoreProductParameterITunesItemIdentifier:@(321231)} completionBlock:^(BOOL result, NSError * _Nullable error) {
        
    }];
```

上面代码SKStoreProuctViewController是应用程序购买页视图控制器，其调用loadProductWithParameters方法进行页面的加载，这个方法有两个参数，第1个参数用来设置配置字典，第2个参数回调Block来告诉开发者页面的加载是否成功。关于配置字典，有如下键值对可用：

```objectivec
//设置要加载的APPID NSNumber类型
SKStoreProductParameterITunesItemIdentifier
//广告token
SKStoreProductParameterAdvertisingPartnerToken
//affiliate token
SKStoreProductParameterAffiliateToken
//CampaignToken
SKStoreProductParameterCampaignToken
//ProviderToken
SKStoreProductParameterProviderToken

```

再多说一点，关于appid的获取，可以直接在https://linkmaker.itunes.apple.com/。网站进行搜索，之后可以获取到应用的下载url地址，这个url地址是被编码过的，解码后其中的参数即有appid值。

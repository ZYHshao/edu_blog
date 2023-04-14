---
title: StoreKit：iOS应用内推广其他App
date: 2022-03-24
categories: iOS之逻辑初窥
tags: []
---
# StoreKit：iOS应用内推广其他App

在iOS应用中，要推广其他App有两种途径，一种是直接跳转到AppStore软件的对应App商品页，还有一种是在当前应用内内嵌一个App商品页。相比第一种方式，第二种方式的体验更好，并且不会打断用户对当前应用的使用。

本篇文章，我们主要介绍StoreKit框架中的相关接口，使用StoreKit可以轻松的在当前应用内推广其他App。

## · 在应用内打开其他App的商品页

StoreKit框架中提供了一个名为SKStoreProductViewController的类，此类事继承自UIViewController的，因此我们可以像使用普通视频控制器一样来使用它。只要我们提供了某个应用的ITunes ID，就可以轻松的在应用中打开其AppStore商品页。

例如下面的代码：

```swift
// 创建视图控制器
let appStoreController = SKStoreProductViewController()
// 设置代理
appStoreController.delegate = self
// 定义参数
let params = [SKStoreProductParameterITunesItemIdentifier: "387682726"]
// 加载应用信息
appStoreController.loadProduct(withParameters: params)
// 将页面弹出
self.present(appStoreController, animated: true)
```

上面代码中使用了淘宝应用的ITunes ID，代码执行效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-34a7598fbde2a3c18a283b104507c952025.jpg)

可以看到，我们直接在应用内就弹起了”淘宝“的详情页，可以直接进行下载/更新操作。

需要注意，上面代码只能在真机上进行测试，且默认页面的弹出方式为浮层样式。

下面我们再来详细看下SKStoreProductViewController这个类的用法，SKStoreProductViewController本身比较简单，创建出实例后，只需要使用loadProduct来加载指定的应用即可，其所传的参数字典中，可配置的选项如下：

```
// 应用的iTunes ID 
@available(iOS 6.0, *)
public let SKStoreProductParameterITunesItemIdentifier: String

// 内购商品的SKU码，如果配置了，则会显示内购商品信息 
@available(iOS 11.0, *)
public let SKStoreProductParameterProductIdentifier: String

// 自定义商品页的ID
@available(iOS 15.0, *)
public let SKStoreProductParameterCustomProductPageIdentifier: String

// 机构token
@available(iOS 8.0, *)
public let SKStoreProductParameterAffiliateToken: String
@available(iOS 8.0, *)
public let SKStoreProductParameterCampaignToken: String

// 用来分析提供者的token
@available(iOS 8.3, *)
public let SKStoreProductParameterProviderToken: String

// 广告伙伴token
@available(iOS 9.3, *)
public let SKStoreProductParameterAdvertisingPartnerToken: String

```

其中，SKStoreProductParameterITunesItemIdentifier键是必传的。

SKStoreProductViewController中也定义了一个delegate属性，设置代理可以对商品页的关闭行为进行监听，如下：

```swift
extension ViewController: SKStoreProductViewControllerDelegate {
    func productViewControllerDidFinish(_ viewController: SKStoreProductViewController) {
        print("商品页关闭")
    }
}
```

此代理方法是可选实现的。

## · 一些小技巧

**如何获取公开应用的ITunes ID？**

现在，我们以及知道了如何在应用内打开其他App的详情页，如何获取ITunes参数呢，其实是有官方的渠道可查的。地址如下：

[https://tools.applemediaservices.com/](https://tools.applemediaservices.com/)

在其中输入我们要查询的应用名称，即可获取到与此应用相关的推广信息，如下图所示：

![](https://oscimg.oschina.net/oscnet/up-129895be3727732e459adaec12ea8d79b2e.png)

可以看到，图中有一段Content Link，这其中就包含了应用的ITunes ID信息，只是其是被URL encode后的，将其复制出来，可以在如下网站进行URL Decode，即可得到原始的ITunes ID。

[http://www.jsons.cn/urlencode/](http://www.jsons.cn/urlencode/)

**对内嵌商品页弹起的高度，文本风格颜色进行配置？**

虽然SKStoreProductViewController提供的接口很少，但我们依然有办法对其做一定程度上的定制，比如其中按钮的风格颜色，浮层的弹起高度。

新建一个继承于SKStoreProductViewController的类，实现如下：

```swift
import UIKit
import StoreKit

class MyStoreProductController: SKStoreProductViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // 设置页面内部分元素的风格颜色
        view.tintColor = .red
    }

    override func viewWillLayoutSubviews() {
        super.viewWillLayoutSubviews()
        // 这里可以控制页面弹起的高度
        view.frame = CGRect(x: 0, y: 400, width: UIScreen.main.bounds.width, height: UIScreen.main.bounds.height - 400)
    }
}

```

运行效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-41f9f1cf89d98ac4f55c6222ee27a2be7ee.jpg)

## · 使用应用挂件

SKStoreProductViewController打开的是一个完整的产品详情页，有时候，我们更期望要推广的应用只是占据一个挂件的位置，在iOS 14及之后的版本中，StoreKit框架中提供了SKOverlay类来实现应用挂件。

示例代码如下：

```swift
// 创建配置，传入要渲染的应用的ITunes ID
let config = SKOverlay.AppConfiguration(appIdentifier: "387682726", position: .bottom)
let overlay = SKOverlay(configuration: config)
// 指定展示在Scene上
if let scene = UIApplication.shared.windows.first?.windowScene {
    overlay.present(in: scene)
}
```

效果如下图所示：

![](https://oscimg.oschina.net/oscnet/up-337616d5e86d2173af67ef93356d290765f.jpg)

可以看到，在窗口底部会出现一个应用挂件。

AppConfiguration实例可配置的属性不多，列举如下：

```swift
@available(iOS 14.0, *)
public class AppConfiguration : SKOverlay.Configuration {
    // 初始化方法
    public init(appIdentifier: String, position: SKOverlay.Position)

    // ITunes ID
    open var appIdentifier: String

    // 一些额外的Token
    open var campaignToken: String?
    open var providerToken: String?

    // 自定义页面的ID
    @available(iOS 15.0, *)
    open var customProductPageIdentifier: String?
  
    // 设置要展示最近版本
    @available(iOS 15.0, *)
    open var latestReleaseID: String?

    // 挂件的位置，可枚举bottom和bottomRaised 差别不大
    open var position: SKOverlay.Position

    // 是否允许用户关闭
    open var userDismissible: Bool

    // 启动附加数据    
    open func setAdditionalValue(_ value: Any?, forKey key: String)
    open func additionalValue(forKey key: String) -> Any?

    // 广告体验配置
    @available(iOS 16.0, *)
    open func setAdImpression(_ impression: SKAdImpression)
}
```

整体来说，SKOverlay不太灵活，对其出现的位置并不能精准的进行控制，SKOverlayDelegate定义了一些方法来监听其行为，如下：

```swift
public protocol SKOverlayDelegate : NSObjectProtocol {
    // 产品加载失败的回调    
    optional func storeOverlayDidFailToLoad(_ overlay: SKOverlay, error: Error)
    // 挂件将要开始弹出的回调
    optional func storeOverlayWillStartPresentation(_ overlay: SKOverlay, transitionContext: SKOverlay.TransitionContext)
    // 挂件以及弹出的回调
    optional func storeOverlayDidFinishPresentation(_ overlay: SKOverlay, transitionContext: SKOverlay.TransitionContext)
    // 挂件将要消失的回调
    optional func storeOverlayWillStartDismissal(_ overlay: SKOverlay, transitionContext: SKOverlay.TransitionContext)
    // 挂件已经消失的回调
    optional func storeOverlayDidFinishDismissal(_ overlay: SKOverlay, transitionContext: SKOverlay.TransitionContext)
}
```

> 专注技术，懂的热爱，愿意分享，做个朋友
> 
> QQ：316045346
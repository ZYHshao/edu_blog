---
title: iOS14新特性探索之一：App Clips轻应用
date: 2020-07-02
categories: iOS14专题
tags: []
---
# iOS14新特性探索之一：App Clips轻应用

        App Clips是苹果WWDC 2020所发布的iOS 14新特性中最具焦点的一项功能。一经曝光，就引发了互联网上针对其特性的各种讨论。有人说App Clips是苹果模仿微信推出的iOS平台的小程序；有人说它是轻量级的应用程序，为用户提供了简洁版的App体验；同样，对AppClips的评价也是众说纷纭，有人看好也有人看跌，有人觉得是新的平台也有人觉得非常鸡肋。

       无论如何，AppClips都是Apple给iPhone用户提供了一种新的交互方式和新的应用使用体验，作为开发者，我们更需要做的是了解这样一种新的技术的应用，并将其赋能到我们的产品中，为用户提供更好的使用体验，为产品带来更大的价值。

        本篇博客，也是基于这样的想法，将全面的介绍App Clips的应用与开发细节，帮助大家最快的了解与上手这样一种技术。我在编写本篇博客时，使用的依然是iOS14的bate版本，开发工具Xcode的版本也是12.0Bate版本，因此，不能保证后续Apple不会对App Clips的某些特性进行优化修改。如果你在之后很久的某个时间阅读到本篇博客，请有选择的借鉴与吸收。

## 1\. 关于App Clips

        App Clips用中文如何翻译，一直没有找到合适的词汇。可以叫他应用切片，也可以叫它轻应用，更可以称它为小程序，这些称呼好像都合适又好像都插那么一点感觉，我们不如就叫它App Clips好了。

       开发一个App Clips的目的是提供你的App中的部分功能让用户可以快速使用，并且不需要下载完整的App。这句话有两个非常重要的点，首先App Clip提供完整应用程序的一部分功能，这表明你一定要有一个完整功能的App，才可以开发上线App Clips，与iOS开发中其他的Extension类似，App Clip也可以理解为一种Extension，其必须由一个宿主App来承载。

       一个App Clip也可以理解为是某个App的轻量级版本，用来提供一些完成瞬时的任务的功能，例如为一杯咖啡进行支付、使用店铺提供的优惠券、公共的信息查询等等。

      App Clip的启动需要由一个调用方调起，在iOS开发中，更专业点的术语叫**_invocation_**，**_invocation_**可以是多种形态的，例如通过点击基于位置信息的推荐Banner，点击Sari的推荐或者通过扫描二维码或NFC等。App Clip被**_invocation_**调起后，用户可以通过它完成一件专注的任务，当用户不再需要使用它时，它会自动的被iPhone移除，这个过程对用户来说是无感知的，因此App Clip也不会占用用户的桌面空间。

## 2\. 开发App Clip前的准备

        在开发App Clip前，你首先需要明确一个核心原则：

**_        App Clips技术一定要用在帮助用户方便启动并快速完成特定任务。_**

       这个原则非常重要，它是App Clips被提供使用的初衷，如果你提供的App Clip非常庞大且操作复杂，那不仅无法丧失了App Clip具有的“快速”优势，而且也会让用户产生困扰，造成糟糕的用户体验。换句话说，App Clips应该是“随用随走”的，即在用户需要使用时快速启动，再用户使用完成后也立刻消失。

       在开发过程中，开发一款App Clip与开发正常的iOS应用并没有特别大的差异，它与开发普通iOS应用有着相关的Framework支持，例如使用UIKit开发应用的界面。同样，App Clips也可以像完整App一样的使用设备的硬件（当然需要申请对应的权限），例如使用相机、蓝牙等等。还有一点需要注意，一个完整的App只能拥有一个App Clip，并且App Clip提供的功能在主App中也需要被完整的支持才行。

        前面说过，App Clip的启动需要由**_invocation_**来触发，**_invocation_**包括如下5中场景：

-   通过NFC扫描来唤起
-   通过点击Sari提供的基于地理位置的推荐
-   在地图App上点击指定的链接
-   点击网页上的智能推荐横幅
-   通过Messages App分享的链接

唤起一个App Clip的过程如下图所示（来自Apple官方文档）

![](https://oscimg.oschina.net/oscnet/up-9ea55f861a6647b4c36f496a95c54c75c9e.png)

如上图，当某个**_invocation_**触发了App Clip时，系统首先会检查**_invocation_**关联的URL，通过URL获取用来展示预览信息的数据，预览信息包括一个背景图案，描述标题与启动按钮，用户点击启动按钮后会打开App Clip。我们可以在App Clip启动时拿到传递进来的URL，通过URL的参数进行不同的逻辑处理。

        了解了App Clips的启动过程，我们知道实际上在启动App Clip之前，系统会先弹出一个预览卡片，这个卡片上的信息可以由开发者在iTunes Connect上自行定义。

        在着手开发App Clips之前，还有一些事情我们需要考虑。

### A.  提供畅快的用户体验

        App Clips不会像通常的App那样展示一个图标在主屏幕上，用户不需要对App Clips进行管理，不用下载也无需删除，当指定的App Clip一段时间不活跃后，系统会自动对它们进行清除。因此，官方建议，App Clips提供的功能应该是线性的，让用户一次性的完成任务。

### B. App Clips需要足够小巧

      App Clips应该足够的小巧，官方限定不可超过10M大小，只有足够小，在用户需要使用的时候才能以更短的时间加载与展示。

### C. 检查可用的框架

      在开发之前，首先要确认下App Clips可用的框架，大部分主App可用的框架在App Clips中都可以使用，但并不是所有，CallKit, CareKit, CloudKit, HealthKit, HomeKit, ResearchKit, SensorKit, 和 Speech在App Clips中是不被支持的。

### D. 保护用户隐私

        由于App Clips会以推荐或其他广告的方式触发，因此保护用户的隐私非常重要。在App Clips中，隐私保护会一直被启用，例如对后台定位权限的申请，当用户同意后，次日的凌晨4点，这个权限会被重新关闭，如果再次启用了App Clips，需要重新向用户申请。当然，还有一些权限在App Clips中是禁止获取的，其中有：运动和健身数据，音乐和多媒体文件，通讯录/信息/照片等文件。除了其宿主App意外，App Clips也不可以和其他应用共享数据。

### E. 思考主App的哪些功能是适合在App Clip提供的

      App Clips提供瞬时的应用体验，因此更多时候，在开发App Clips之前，你最先应该思考的是主App上的哪些功能是适合在App Clips上提供的。这需要从产品角度深度的思考，并真实的站在用户的角度体验。

## 3\. 创建一个App Clip程序

        前面讲了很多开发App Clips前的准备，现在就让我们上手创建一个App Clip体验一下。创建App Clip非常简单，首先其需要一个宿主App，如果为已经存在的一个完全应用程序添加App Clip，就更加容易，我们只需要新建一个Target，选择App Clip即可，如下图所示：

![](https://oscimg.oschina.net/oscnet/up-23a424ebfc54514fa370610c843b77d9057.png)

新建了App Clip的Target后，Xcode会自动的帮我们创建好一系列必须的文件，并做好配置。这时，直接运行target对应的scheme即可在模拟器或真机上运行App Clip做测试，当前，如果你运行会出现一个空白的页面，这是因为我们还没有编写任何代码。

        前面说过，App Clip的运行需要**_invocation_**进行调用，对于**_invocation_**的调用，如果用户安装了完全的主App，则会唤起主App来处理用户任务，如果用户没有安装主App，则自动调起App Clip。无论通过哪种**_invocation_**来调起App Clip，我们都需要在App Clip的target中配置指定的域权限。在target工程的设置页面，找到Associated Domains选项，在其中添加要调起App Clip的域名，需要找到这样的格式：appclips:xxx.com。这种配置方式与Deep link的逻辑基本一致。

![](https://oscimg.oschina.net/oscnet/up-b403bb7393cb5539d991c027fd1e220f2d3.png)

下面，我们可以尝试向App Clip工程中添加一些代码。观察App Clip的工程目录结构，可以发现，其和正常的App几乎没有什么差异。如下图所示：

![](https://oscimg.oschina.net/oscnet/up-b902e06b5f4e55bab4b7d37a4532dee560d.png)

我们简单的在ViewController中添加一些代码，例如点击屏幕后，随机改变界面的颜色，如下：

```objectivec
#import "ViewController.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    [self changeColor];
}

- (void)changeColor {
    self.view.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
}

@end

```

在开发App Clips时，我们应该尽量的让主App的代码与App Clips需要使用的代码共享，共享代码非常容易，将可以共享的代码保证在静态库或动态库内即可。有时候，你可能需要在共享的代码中区分环境，比如某些代码只在target环境下被编译，某些则只在主App环境下被编译。可以为target添加一个特殊的编译宏来区分环境，Objective-C的工程，这个编译选项需要在Build Settings的Preprocessor Macros选项下进行配置，Swift的工程则需要在Active Compilation Conditions选项下进行配置，例如我们为target工程添加一个CLIP的编译宏，如下图所示：

![](https://oscimg.oschina.net/oscnet/up-2ac60b78f13d041325bfb252a08bae74824.png)

之后，在编写代码时，对于针对target的代码，就可以将其通过条件编译区分：

```objectivec
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
#ifdef CLIP
    self.view.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
#endif
}
```

对于发布App Clips的程序，还有一点需要注意，在系统弹出预览卡片之前，首先会先通过**_invocation_**关联的URL对App Clip进行校验，如果校验不成功，则不会弹出预览卡片也不会打开App Clip程序。要成功进行验证，除了在Xcode工程中配置正确的Associated Domains外，还需要对我们的Web服务做一些修改。需要在服务器添加Apple App Site Association文件，并且添加上类似如下的配置来支持App Clips：

```objectivec
{
    "appclips": {
        "apps": ["ABCED12345.com.example.MyApp.Clip"]
    }
    ...
} 
```

## 4\. 测试App Clips的启动

      虽然App Clips的启动需要由**_invocation_**触发，但是在开发过程中，我们依然有方法来模拟Web发出的**_invocation_**来启动App Clip。编辑在App Clip对应的scheme，在其中添加 _XCAppClipURL这个键值来配置模拟从某个URL调起App Clips，如下图所示：

![](https://oscimg.oschina.net/oscnet/up-0de1604741d53bc7587e144dd3a837054fe.png)

当App Clip被调起后，我们可以通过一些回调方法来拿到URL信息，根据不同的信息，可以将不同的功能线展示给用户进行使用。以UIKit工程为例，当App Clip被调起时，会调用如下方法：

```objectivec
- (void)scene:(UIScene *)scene continueUserActivity:(NSUserActivity *)userActivity {
    NSLog(@"%@", userActivity);
}
```

在UNUserAvtivity对象中可以获取到传递的URL等信息。如果你的App Clip是可以被特殊的地理位置触发的，那么需要在Clip对应target的info.plist中添加 NSAppClip键，并将其中的 NSAppClipRequestLocationConfirmation键设置为YES。

## 5\. 配置预览卡片

        在真正的启动App Clip之前，首先会弹出预览卡片。在提交App Clip时，是跟宿主App一起打包上传到App Store Connect上的，App Clip的预览卡片配置也是在App Store Connect上完成，提供给开发者自由配置的地方并不多，包括3个方面：

1.  配置一个头图
2.  配置副标题并提供描述文案
3.  配置交互按钮

## 6\. 关于数据共享

        App Clips的和宿主App的数据共享并没有什么特殊的地方，其和普通Extension Target与宿主App通信的方式一样，只要创建一个App Group，并将App Clips与宿主App放入同一个App Group中，之后就可以通过NSUserDefault来进行数据的共享。

---
title: iOS中Today扩展插件与宿主APP的交互
date: 2016-07-14
categories: iOS逻辑初窥
tags: []
---
## iOS中Today扩展插件与宿主APP的交互

        扩展是iOS8后系统开发给开发者的新开发思路与接口，每一个扩展都可以理解为一个简单的小应用程序，只是其不是独立存在的，要寄附于某一个主应用上。介绍iOS8扩展与Today插件的专题见如下博客：

iOS8中扩展与Today插件：[http://my.oschina.net/u/2340880/blog/485533](http://my.oschina.net/u/2340880/blog/485533)。

        上述博客中只是简单的介绍扩展的应用场景与创建Today扩展插件的方法，在实际开发中，由于扩展是寄附于某个应用程序之上的，因此其通常需要和宿主APP进行数据交互。创建Today扩展Target后，Xcode模板会自动帮助开发者生成一个ViewController作为主界面，开发者可以向其中添加展示UI或者交互控件，十分强大的是，Today扩展中是支持对UIViewController的切换的。需要注意，扩展与原APP是在不同的目录结构中的，默认情况下，扩展与原APP的数据并不共享，代码也不能复用。例如原APP中可能有网络请求，数据持久化存储等结构框架，扩展中不可以直接使用，扩展需要提供自己的网络请求框架爱，数据持久化结构框架等。

      如果项目是使用Pod进行的管理，则可以通过手动设置，使扩展中可以使用继承的Pod库，步骤如下：

![](http://static.oschina.net/uploads/space/2016/0714/145044_HMle_2340880.png)

![](http://static.oschina.net/uploads/space/2016/0714/145448_dmuE_2340880.png)

完成上面两张图中的步骤，即可在扩展中使用Pod库了。

        Xcode扩展模板创建的ViewController会自动遵守NSWidgetProviding这个协议，这个协议中的方法和意义如下，开发者可以根据需求选择实现：

```objectivec
//数据更新时调用的方法 系统会定期更新扩展
- (void)widgetPerformUpdateWithCompletionHandler:(void (^)(NCUpdateResult result))completionHandler;
//设置扩展UI边距 注意 在使用Storyboard时，若要所见即所得 这个方法中需要返回UIEdgeInsetsZero
- (UIEdgeInsets)widgetMarginInsetsForProposedMarginInsets:(UIEdgeInsets)defaultMarginInsets;
```

注意：Today扩展有其自己的plist配置文件，若需要对扩展进行配置，注意不要与宿主工程的plist文件混淆。

        在Today扩展中打开原宿主APP使用openURL的方式，示例如下：

```objectivec
 [viewController.extensionContext openURL:[NSURL URLWithString:[NSString stringWithFormat:@"MyApp://action=%@",@"action"]] completionHandler:nil];
```

上面打开原宿主APP的代码中，MyApp是宿主APP配置的url Schemes，配置方式如下图：

![](http://static.oschina.net/uploads/space/2016/0714/152000_aTRd_2340880.png)

可以通过为url配置参数的方式来进行Today扩展与原宿主APP的信息交互，当扩展使用openURL的方式打开原宿主APP时，宿主APP会调用AppDelegate中的如下方法：

```objectivec
-(BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString *,id> *)options{
//可以拿到url做相应逻辑处理
    UIAlertView * alert = [[UIAlertView alloc]initWithTitle:url.absoluteString message:nil delegate:nil cancelButtonTitle:@"确定" otherButtonTitles:nil, nil];
    [alert show];
    return YES;
}
```

        上面介绍的openURL的方式只是进行跳转交互，参数传递，并不能完成数据共享的需求，并且通过openURL的方式传递的数据是单向的。实际上，扩展和原宿主APP共享数据的应用场景十分广泛，例如电商类宿主APP中拉取到一批商品信息，Today扩展中也需要这些信息进行展示，如果数据不共享，同样的数据将在宿主APP内部和扩展都都请求一次，十分浪费，难很难同步。系统还提供了另一种方式来使宿主APP和Today扩展可以共享一块存储空间，这需要使用App Group技术来实现。开发者在进行App Group相关功能的测试时，必须与AppID进行关联。

        首先，需要开启宿主APP的App Group，示例图如下：

![](http://static.oschina.net/uploads/space/2016/0714/153442_xagO_2340880.png)

在Today扩展中，选择相同的App Group，如下：

![](http://static.oschina.net/uploads/space/2016/0714/153706_iwtY_2340880.png)

开启了App Group功能后，Xcode会自动生成一套匹配的权限文件，如下：

![](http://static.oschina.net/uploads/space/2016/0714/154357_RXtd_2340880.png)

配置工作完成后，可以通过两种方式共享数据存储空间，示例如下：

```objectivec
    //使用数据共享的NSUserDefaults 这个NSUserDefaults是宿主APP与扩展所共享的
    NSUserDefaults * defaults =[NSUserDefaults alloc]initWithSuiteName:@"开发者设置的AppGroup名称"];

    //使用数据共享的文件目录
    NSFileManager * manager = [NSFileManager defaultManager];
    //共享目录
    NSURL * baseURL = [manager containerURLForSecurityApplicationGroupIdentifier:@"开发者设置的AppGroup名称"];
    //找文件
    NSURL * filePath =  [baseURL URLByAppendingPathComponent:@"file"];
```

注意：还有一点细节需要注意，扩展与原宿主APP素材文件也是互相独立的，要在扩展中使用的素材必须添加进扩展Target。

小提示：使用Xcode调试扩展时，需要运行扩展的Target，开发者有时会发现断点失效，将模拟器上的应用删掉，重新运行扩展即可解决。

Demo地址：[http://pan.baidu.com/s/1bp0ZcYF](http://pan.baidu.com/s/1bp0ZcYF)。

截图：

![](http://static.oschina.net/uploads/space/2016/0714/164000_sN8O_2340880.png)      ![](http://static.oschina.net/uploads/space/2016/0714/164019_yRNT_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

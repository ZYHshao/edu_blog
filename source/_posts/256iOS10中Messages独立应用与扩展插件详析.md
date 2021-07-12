---
title: iOS10中Messages独立应用与扩展插件详析
date: 2016-09-18
categories: iOS10专题
tags: []
---
## iOS10中Messages独立应用与扩展插件详析

### 一、引言

        Messages是iOS系统中原生的信息应用，其既可以通过运营商网络发送短信息，也可以通过互联网进行类似微信类社交软件的即时聊天。但是由于其封闭性与功能的单一，使用其进行即时聊天的用户并不多。随着iOS10系统的推出，或许可以改变这一现状。在iOS10中，Messages的功能被扩展的十分强大，通过Messages，用户可以分享图片，音乐，视频，可以随手涂鸦，使用自定义的表情包，可以进行Apple Pay支付，购物，甚至可以在Messages中玩游戏。并且，上面所提到的这些功能都全面开发出了接口供开发者进行开发与扩展。

        在iOS10中，开发者可以进行与Messages相关的开发有两类：独立的Messages应用与Messages应用扩展。其中，Messages应用扩展需要依附一个宿主App而存在。无论哪种类型的Messages应用，其都又分为两类，StickerPicks（表情包）与iMessage Apps(Messages应用)。

### 二、开发表情包StickerPicks

#### 1.开发独立的表情包

        Sticker Picks可谓是iOS10中一个十分强大的新功能。在iOS10系统的iPhone上，Messages应用中会内嵌一个Message App Store，用户可以直接从里面下载针对于Messages的独立表情包和独立第三方应用。开发者也可以独立开发表情包发布到这个Message App Store中。

        开发Sticker Picks表情包十分简单，开发者可以不用写一句代码，将整理好的表情进行打包提交即可完成。使用Xcode8创建一个新的工程，选择Sticker Pack Application模板，如下图所示：

![](https://static.oschina.net/uploads/space/2016/0919/160153_pr19_2340880.png)

创建出工程后，可以发现模板中没有任何代码文件，只有一个Stickers.xcstickers包。将准备好的表情包图片导入这个Stickers中，其中支持静态图片，也支持动态表情gif图片。关于导入的图片，有如下几条规则：

1.图片文件的格式必须是PNG、APNG、GIF或者JPEG。

2.单个文件的大小不能超过500KB。

3.最优的效果是当图片尺寸在100\*100到206\*206之间。

注意：在提供图片的时候，开发者只需要提供@3倍图即可，即最优尺寸在300\*300到618\*618之间的图片。系统会自动生成@2与@1倍图。

        开发的表情包会显示在Messages应用的工具中，需要注意，在表情列表的排版中，每个表情缩略图只支持3种尺寸的排版，对应的尺寸分别如下：

Small类型：100*100

Medium类型：136*136

Large类型：206*206

在Xcode中，可以对要使用的模板进行选择，如下图：

![](https://static.oschina.net/uploads/space/2016/0919/162302_BW5T_2340880.png)

在模拟器中运行工程，Messages中效果如下图：

![](https://static.oschina.net/uploads/space/2016/0919/162900_c8hi_2340880.png)               ![](https://static.oschina.net/uploads/space/2016/0919/162917_KS9Y_2340880.png)

和普通iOS应用程序一样，将设备选择为Generic iOS Device后直接Archives即可将表情包提交到AppStore，审核通过后，即可在Message App Store中进行下载。

小提示：其实StickerPicks翻译成表情包并不合适，其更有一层贴纸的概念。实际上其也确实有贴纸的功能，在Messages应用中，用户可以通过长按移动手势，来将某个Sticker添加在另一个Sticker上面。如下图：

![](https://static.oschina.net/uploads/space/2016/0920/163007_G0dP_2340880.png)

#### 2.开发寄宿于宿主App的表情包扩展

        扩展表情包与独立表情包最大的不同在于扩展需要寄宿于某个宿主App中，创建扩展target，选择Sticker Pick Extension，如下图，之后和独立表情包开发过程一致。

![](https://static.oschina.net/uploads/space/2016/0919/164112_DIJT_2340880.png)

#### 3.关于表情包的icon图标

        StickerPicks的图标和宿主App并不共用，其需要一套独特尺寸的icon，尺寸如下：

![](https://static.oschina.net/uploads/space/2016/0919/165649_ZhQv_2340880.png)

效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/0919/165952_Vc54_2340880.png)

### 三、开发Messages App应用

#### 1.认识Messages框架

        和StickerPicks表情包一样，Messages App也分为独立应用与扩展两种。其实它们的开发思路和方法完全一致，只是有无宿主App的区别。

        开发Messages App需要使用到iOS中引入的一个新的开发框架Messages。Messages比较简单，其中涉及到的类并不十分多，下图中概述了其中重要的类和之间的关系：

![](https://static.oschina.net/uploads/space/2016/0920/162435_4Jg0_2340880.png)

MSMessageAppViewController：这个类Messages App的基础视图控制器类，其继承自UIViewController，但其中添加了许多Messages App相关的声明周期方法。

MSConversation：描述一个会话实例。

MSSticker：表情贴图实例。

MSMessage：在Messages App之间进行传递的消息实体。

MSMessageLayout：抽象类，其并没有实现任何方法，有子类实现。

MSMessageTemplateLayout：用于对消息实体MSMessage进行布局排版。

MSStickerBorwserViewController：用于创建表情包视图控制器。

MSStickerBorwserView：表情包视图容器，类似CollectionView。

MSStickerView：表情承载视图。

#### 2.实现一个Messages App的列表界面

        使用Xcode新建一个Messages App工程如下：

![](https://static.oschina.net/uploads/space/2016/0920/163705_0aIf_2340880.png)

其会自动生成一个MessagesViewController类，这个类就是此Messages App的主界面视图控制器。需要注意，Messages App的视图控制器都分为两种状态，分别为Compact(紧凑的)和Expanded(扩宽的)。并且在这两种状态进行切换时，视图的底部的工具栏和头部的导航栏也会交替出现，这导致了即使是使用自动布局，依然无法完美的解决Messages App布局的统一性，需要手动进行调整处理，后面会介绍到。

        在MessagesViewController类中添加其他视图控件，大部分iOS App开发中可以使用的UI控件这里都可以使用，但是有一点需要注意，对于可以弹出键盘的UI控件，例如UITextView与UITextField，当Messages App界面处理Compact模式时，键盘是不能弹出的，只有当界面处于Expanded模式时，键盘才被允许弹出。

        为了使Messages App的界面在任何模式下都能保持统一，需要手动对其中视图约束进行修改，示例代码如下：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    self.dataArray = [NSMutableArray array];
    [self.dataArray addObjectsFromArray:@[@"发送文本信息",@"插入表情",@"插入文件",@"插入消息实体",@"跳转第二个界面",@"贴图包"]];
    [self.tableView registerClass:[UITableViewCell class] forCellReuseIdentifier:@"cellId"];
    self.tableView.dataSource = self;
    self.tableView.delegate = self;
    self.session = [[MSSession alloc]init];
}


-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
    return self.dataArray.count;
}

-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    UITableViewCell * cell = [tableView dequeueReusableCellWithIdentifier:@"cellId"];
    cell.textLabel.text = self.dataArray[indexPath.row];
    return cell;
}
//这个方法在Messages App加载完成处于活跃状态时被调用 在其中根据模式设置布局参数
-(void)didBecomeActiveWithConversation:(MSConversation *)conversation {
    // Called when the extension is about to move from the inactive to active state.
    // This will happen when the extension is about to present UI.
    if (self.presentationStyle==MSMessagesAppPresentationStyleCompact) {
        _topMargan.constant = 0;
        _leftMargan.constant = -15;
        _rightMargan.constant = -15;
        _bottomMargan.constant=-44;
    }else{
        _topMargan.constant = -85;
        _leftMargan.constant = -15;
        _rightMargan.constant = -15;
        _bottomMargan.constant=0;
    }
   [self.view layoutIfNeeded]; 
}
//这个方法在视图控制器的模式发生了改变时调用 在其中根据模式修改布局参数
-(void)didTransitionToPresentationStyle:(MSMessagesAppPresentationStyle)presentationStyle {
    // Called after the extension transitions to a new presentation style.
    
    // Use this method to finalize any behaviors associated with the change in presentation style.
    if (presentationStyle==MSMessagesAppPresentationStyleCompact) {
        _topMargan.constant = 0;
        _leftMargan.constant = -15;
        _rightMargan.constant = -15;
        _bottomMargan.constant=-44;
    }else{
        _topMargan.constant = -85;
        _leftMargan.constant = -15;
        _rightMargan.constant = -15;
        _bottomMargan.constant=0;
    }
    [self.view layoutIfNeeded];
}

```

![](https://static.oschina.net/uploads/space/2016/0920/165414_rI7L_2340880.png)                          ![](https://static.oschina.net/uploads/space/2016/0920/165435_uA43_2340880.png)

#### 3.解析MSMessagesAppViewController类

        由于MSMessagesAppViewController类是继承于UIViewController类的，因此UIViewController中的视图控制器切换方法这里都可以直接使用，MSMessagesAppViewController中供开发者进行调用的属性和方法如下：

```objectivec
//当前激活的会话实例 后面会介绍
@property (nonatomic, strong, readonly, nullable) MSConversation *activeConversation;
//当前界面所在的状态
/*
typedef NS_ENUM(NSUInteger, MSMessagesAppPresentationStyle) {
    //紧凑状态
    MSMessagesAppPresentationStyleCompact,
    //扩宽状态
    MSMessagesAppPresentationStyleExpanded
} NS_ENUM_AVAILABLE_IOS(10_0);
*/
@property (nonatomic, assign, readonly) MSMessagesAppPresentationStyle presentationStyle;
//切换界面状态
-(void)requestPresentationStyle:(MSMessagesAppPresentationStyle)presentationStyle;
//调用此方法后，MessagesApp被收回 弹出键盘
-(void)dismiss;
```

MSMessagesAppViewController中新增加的声明周期方法如下：

```objectivec
//当Messages App将要激活时调用
-(void)willBecomeActiveWithConversation:(MSConversation *)conversation;
//当Messages App已经被激活后调用
-(void)didBecomeActiveWithConversation:(MSConversation *)conversation;
//当Messages App将要被注销时调用
-(void)willResignActiveWithConversation:(MSConversation *)conversation;
//当MessageApp已经被注销时调用
-(void)didResignActiveWithConversation:(MSConversation *)conversation;
//消息实体在会话中将要被选中时调用
-(void)willSelectMessage:(MSMessage *)message conversation:(MSConversation *)conversation;
//消息实体在会话中已经被选中时调用
-(void)didSelectMessage:(MSMessage *)message conversation:(MSConversation *)conversation;
//接收到同一Messages App发送的消息实体时调用
-(void)didReceiveMessage:(MSMessage *)message conversation:(MSConversation *)conversation;
//开发发送消息时调用
-(void)didStartSendingMessage:(MSMessage *)message conversation:(MSConversation *)conversation;
//取消发送时调用
-(void)didCancelSendingMessage:(MSMessage *)message conversation:(MSConversation *)conversation;
//控制器界面模式将要改变时调用
-(void)willTransitionToPresentationStyle:(MSMessagesAppPresentationStyle)presentationStyle;
//控制器界面已经改变时调用
-(void)didTransitionToPresentationStyle:(MSMessagesAppPresentationStyle)presentationStyle;

```

#### 4.使用MSConversation会话类来进行消息的发送

        MSConversation类用来描述会话，MSMessagesAppViewController中内置MSConversation对象，开发者可以用它来进行消息传递，示例代码如下：

```objectivec
-(void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath{
    switch (indexPath.row) {
        case 0:
        {
            [self.activeConversation insertText:@"新的信息" completionHandler:^(NSError * error) {
                
            }];
        }
            break;
        case 1:
        {
            [self.activeConversation insertSticker:[[MSSticker alloc] initWithContentsOfFileURL:[[NSURL alloc] initFileURLWithPath:[[NSBundle mainBundle] pathForResource:@"productImage" ofType:@"png"]] localizedDescription:@"image" error:nil] completionHandler:nil];
        }
            break;
        case 2:
        {
            [self.activeConversation insertAttachment:[[NSURL alloc] initFileURLWithPath:[[NSBundle mainBundle] pathForResource:@"file" ofType:nil]] withAlternateFilename:@"文件" completionHandler:nil];
        }
            break;
        case 3:
        {
            MSMessage * message = [[MSMessage alloc]initWithSession:_session];
            message.URL = [NSURL URLWithString:@"http://www.baidu.com"];
            message.accessibilityLabel = @"message";
            message.summaryText = @"message";
            MSMessageTemplateLayout * layout = [[MSMessageTemplateLayout alloc]init];
            layout.caption = @"caption";
            layout.subcaption = @"subcaption";
            layout.trailingCaption = @"trailing";
            layout.trailingSubcaption =@"subtrailing";
            layout.image = [UIImage imageNamed:@"productImage"];
            layout.mediaFileURL =[[NSURL alloc] initFileURLWithPath:[[NSBundle mainBundle] pathForResource:@"productImage" ofType:@"png"]];
            layout.imageTitle = @"杜康";
            layout.imageSubtitle = @"酒水";
            
            message.layout = layout;
            
            [self.activeConversation insertMessage:message completionHandler:nil];
        }
            break;
        case 4:
        {
            [self presentViewController:[StackerViewController new] animated:YES completion:nil
             ];
        }
            break;
        case 5:
        {
            [self presentViewController:[StackController new] animated:YES completion:nil
             ];
        }
            break;
        default:
            break;
    }
}

```

MSConversation支持发送的消息分为4中，分别为文本消息，表情贴图消息，文件消息和Message实体消息，上面代码都做了演示。MSConversation中重要属性和方法解析如下：

```objectivec
//本地的设备UUID
@property (nonatomic, readonly) NSUUID *localParticipantIdentifier;
//会话中远程设备的UUID 支持多人会话
@property (nonatomic, readonly) NSArray<NSUUID *> *remoteParticipantIdentifiers;
//当前选中的消息实体
@property (nonatomic, readonly, nullable) MSMessage *selectedMessage;
//插入文本消息
- (void)insertText:(NSString *)text completionHandler:(nullable void (^)(NSError * _Nullable))completionHandler;
//插入表情贴图消息
- (void)insertSticker:(MSSticker *)sticker completionHandler:(nullable void (^)(NSError * _Nullable))completionHandler;
//插入文件附件消息
- (void)insertAttachment:(NSURL *)URL withAlternateFilename:(nullable NSString *)filename completionHandler:(nullable void (^)(NSError * _Nullable))completionHandler;
//插入Message实体消息
- (void)insertMessage:(MSMessage *)message completionHandler:(nullable void (^)(NSError * _Nullable))completionHandler;
```

效果图如下：

![](https://static.oschina.net/uploads/space/2016/0920/185552_zFnt_2340880.png)        ![](https://static.oschina.net/uploads/space/2016/0920/185634_NM8E_2340880.png)      ![](https://static.oschina.net/uploads/space/2016/0920/185648_AgAD_2340880.png)

#### 5.消息实体MSMessage的应用

        MSMessage是Messages App定义的一种消息实体，其可以用来在Messages App间传递信息，因为它的存在，通过Messages用用实现休闲对战游戏变得十分容易，开发者不需要在写即时通信链接，只需设计游戏逻辑即可。MSMessage不能够完全自定义UI，但是Messages框架中的MSMessageTemplateLayout类可以对其UI进行简单的配置。

        MSMessage类中常用的属性和方法如下：

```objectivec
//初始化方法 可以绑定一个session，同一个session种的消息实体会被归为一类
-(instancetype)initWithSession:(MSSession *)session NS_DESIGNATED_INITIALIZER;
//发送此消息的设备
@property (nonatomic, readonly) NSUUID *senderParticipantIdentifier;
//消息的UI布局信息
@property (nonatomic, copy, nullable) MSMessageLayout* layout;
//消息附带的URL 开发者可以通过这个URL来传值 
@property (nonatomic, copy, nullable) NSURL *URL;
//是否保留过期的消息
@property (nonatomic, assign) BOOL shouldExpire;

//盲人模式中对应的文案
@property (nonatomic, copy, nullable) NSString *accessibilityLabel;
```

#### 6.消息实体布局类MSMessageLayout

        前面介绍，MSMessage类中并没有定义UI，UI部分需要配合MSMessageLayout类来配置。需要注意，MSMessageLayout类是一个抽象类，apple设计的目的可能是为了以后便于扩展多个消息布局模板。目前，开发者只需要使用MSMessageTemplateLayout类来对消息实体进行布局。

        MSMessageTemplateLayout类中可以配置的属性如下：

```objectivec
//设置消息实体的标题
@property (nonatomic, copy, nullable) NSString *caption;
//设置消息实体的子标题
@property (nonatomic, copy, nullable) NSString *subcaption;
//设置消息实体的右侧标题
@property (nonatomic, copy, nullable) NSString *trailingCaption;
//设置消息实体的右侧子标题
@property (nonatomic, copy, nullable) NSString *trailingSubcaption;
//设置消息实体的图片
@property (nonatomic, strong, nullable) UIImage *image;
//设置消息实体的媒体地址 需要注意 如果设置的image属性 这个属性将被忽略
@property (nonatomic, copy, nullable) NSURL *mediaFileURL;
//设置消息实体的图标标题
@property (nonatomic, copy, nullable) NSString *imageTitle;
//设置消息实体的图片子标题
@property (nonatomic, copy, nullable) NSString *imageSubtitle;

```

#### 7.表情贴图类MSSticker与MSStickerView

        在制作表情包Sticker Picks的时候，开发者不需要编写一行代码，实际上如果要通过代码来开发表情包也是没有问题的，这里需要用到的一个类就是MSSticker类，简单理解，MSSticker类对象就是一个表情贴图，但是它不是一个View视图，若想在Messages App中看到这个表情贴图，还需要借助一个类MSStickerView，MSStickerView是用于承载表情贴图的视图类，用户选中它后，可以在Messages应用中进行发送。

        首先，MSSticker类创建方法如下：

```objectivec
//初始化方法 通过文件URL 来创建实例
- (nullable instancetype)initWithContentsOfFileURL:(NSURL *)fileURL localizedDescription:(NSString *)localizedDescription error:(NSError * _Nullable *)error NS_DESIGNATED_INITIALIZER;
```

MSStickerView类解析如下：

```objectivec
//通过MSSticker来进行MSStickerView类的创建
- (instancetype)initWithFrame:(CGRect)frame sticker:(nullable MSSticker *)sticker;

//获取动画播放一遍的时间 如果是gif
@property(nonatomic, readonly) NSTimeInterval animationDuration;
//开始动画
-(void) startAnimating;
//结束动画
-(void) stopAnimating;
//获取动画状态
- (BOOL)isAnimating;
```

需要注意，MSStickerView如果加载的是gif类型的表情贴图，默认不会播放动画，开发者可以调用开始动画的方法来进行gif动画的播放。

#### 8.表情包视图控制器MSStickerBrowserViewController

        其实通过前面的内容，已经可以自定义开发一个表情包Messages App了，但是还有一个视图控制器类MSStickerBrowserViewController，这个类可以更加简单方面的创建表情包视图控制器。要了解MSStickerBrowserViewController类，首先应该先了解MSStickerBrowserView类，这两个类的关系十分类似于UITableViewController与UITableView类的关系。MSStickerBrowserView是用于展示表情视图的容器，其继承自UIView，但却和UICollectionView十分类似，其中方法解析如下：

```objectivec
//初始化方法 设置frame 和其中表情视图的尺寸模式
/*
typedef NS_ENUM(NSInteger, MSStickerSize) {
    //小尺寸
    MSStickerSizeSmall,
    //标准尺寸
    MSStickerSizeRegular,
    //大尺寸
    MSStickerSizeLarge
} NS_ENUM_AVAILABLE_IOS(10_0);
*/
- (instancetype)initWithFrame:(CGRect)frame stickerSize:(MSStickerSize)stickerSize NS_DESIGNATED_INITIALIZER;
//数据源代理
@property (nonatomic, weak, nullable) id <MSStickerBrowserViewDataSource> dataSource;
//当前的滑动位置
@property (nonatomic, assign, readwrite) CGPoint contentOffset;
//内容偏移尺寸
@property (nonatomic, assign, readwrite) UIEdgeInsets contentInset;
//设置当前的滑动位置
- (void)setContentOffset:(CGPoint)contentOffset animated:(BOOL)animated;
//刷新数据
- (void)reloadData;

```

MSStickerBrowserView的数据填充需要在代理方法中实现，如下：

```objectivec
//设置表情贴图个数
- (NSInteger)numberOfStickersInStickerBrowserView:(MSStickerBrowserView *)stickerBrowserView;
//设置具体每个位置的表情贴图
- (MSSticker *)stickerBrowserView:(MSStickerBrowserView *)stickerBrowserView stickerAtIndex:(NSInteger)index;
```

再看MSStickerBrowserViewController就十分容易了，它只是将MSStickerBrowserView封装在了一个UIViewController中，并且这个UIViewController遵守了MSStickerBrowserViewDataSource协议，开发者直接实现协议方法即可。

### 四、开发Messages App中的建议

        下面是Apple对Messages App的定位和一些建议，还有我的一些理解：

1.确保应用是有用的并且易于理解。

2.功能要聚焦单一，不要组合多种功能在一起。

3.Messages通常用在双人非正式的交谈中，应从这里入手，让交流更加有趣。

4.Messages的最大两点是分享，利用这一点出发开发Messages App。

5.插图内容布局要注意，系统会自动将内容变为圆角，不要把重要的信息放在角落。

6.注意，在紧凑模式下，Messages App的界面是不允许水平滚动的。

7.同样，在紧凑模式下，Messages App不允许键盘输入。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

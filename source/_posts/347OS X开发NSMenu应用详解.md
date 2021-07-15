---
title: OS X开发NSMenu应用详解
date: 2017-07-14
categories: macOS开发
tags: []
---
## OS X开发NSMenu应用详解

### 一、引言

    NSMenu在Mac桌面软件开发中往往有3个方面的应用，作为程序的主菜单栏使用，作为视图邮件菜单使用和作为Dock菜单使用。

### 二、主应用菜单

    使用Xcode新建OX S应用时，可以选择使用Storyboard。Storyboard里面会自动创建一个菜单栏，你可以自行在菜单中进行增删改操作，菜单中的Item触发方法也可以直接与AppDelegate进行关联，实现自定义的菜单逻辑，如图：

![](https://static.oschina.net/uploads/space/2017/0714/144023_fg6f_2340880.png)

### 三：Dock菜单

    当一款Mac桌面软件运行时，会在Dock栏上显示一个图标，当在此图标上点击右键时，会出现一个Dock菜单，自定义此Dock菜单也十分容易，直接在AppDelegate中重写如下方法即可：

```objectivec
-(NSMenu *)applicationDockMenu:(NSApplication *)sender{
    NSMenu * menu = [[NSMenu alloc]initWithTitle:@"Menu"];
    NSMenuItem * item1 = [[NSMenuItem alloc]initWithTitle:@"菜单1" action:@selector(click) keyEquivalent:@""];
    item1.target = self;
    NSMenuItem * item2 = [[NSMenuItem alloc]initWithTitle:@"菜单2" action:@selector(click) keyEquivalent:@""];
    item2.target = self;
    NSMenuItem * item3 = [[NSMenuItem alloc]initWithTitle:@"菜单3" action:@selector(click) keyEquivalent:@""];
    NSMenu * subMenu = [[NSMenu alloc]initWithTitle:@"subMenu"];
    NSMenuItem * item4 = [[NSMenuItem alloc]initWithTitle:@"菜单4" action:@selector(click) keyEquivalent:@""];
    item4.target = self;
    [subMenu addItem:item4];
    [menu addItem:item1];
    [menu addItem:item2];
    [menu addItem:item3];
    [menu setSubmenu:subMenu forItem:item3];
    return menu;
}
```

效果如下：

![](https://static.oschina.net/uploads/space/2017/0714/144858_hmTW_2340880.png)

### 四、视图右键弹出菜单

    视图右键弹出菜单是基于NSView视图的，例如：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    NSMenu * menu = [[NSMenu alloc]initWithTitle:@"Menu"];
    NSMenuItem * item1 = [[NSMenuItem alloc]initWithTitle:@"菜单1" action:@selector(click) keyEquivalent:@""];
    item1.target = self;
    NSMenuItem * item2 = [[NSMenuItem alloc]initWithTitle:@"菜单2" action:@selector(click) keyEquivalent:@""];
    item2.target = self;
    NSMenuItem * item3 = [[NSMenuItem alloc]initWithTitle:@"菜单3" action:@selector(click) keyEquivalent:@""];
    NSMenu * subMenu = [[NSMenu alloc]initWithTitle:@"subMenu"];
    NSMenuItem * item4 = [[NSMenuItem alloc]initWithTitle:@"菜单4" action:@selector(click) keyEquivalent:@""];
    item4.target = self;
    [subMenu addItem:item4];
    [menu addItem:item1];
    [menu addItem:item2];
    [menu addItem:item3];
    [menu setSubmenu:subMenu forItem:item3];
    [self.view setMenu:menu];
}
```

效果如下：

![](https://static.oschina.net/uploads/space/2017/0714/145241_FaDJ_2340880.png)

### 五、NSMenuItem详解

    NSMenuItem是菜单中的每一个菜单选项对象，其中常用属性方法如下：

```objectivec
//设置是否启用用户快捷键
+ (void)setUsesUserKeyEquivalents:(BOOL)flag;
//设置用户快捷键启用状态
+ (BOOL)usesUserKeyEquivalents;
//创建一个分割线
+ (NSMenuItem *)separatorItem;
//使用标题，快捷键和方法选择器来对Item进行初始化
- (instancetype)initWithTitle:(NSString *)string action:(nullable SEL)selector keyEquivalent:(NSString *)charCode;
//其所在的菜单对象
@property (nullable, assign) NSMenu *menu;
//其是否有子菜单
@property (readonly) BOOL hasSubmenu;
//子菜单对象
@property (nullable, strong) NSMenu *submenu;
//如果此Item是某个子菜单中的，此属性获取与子菜单关联的父item
@property (nullable, readonly, assign) NSMenuItem *parentItem;
//Item标题
@property (copy) NSString *title;
//富文本标题
@property (nullable, copy) NSAttributedString *attributedTitle;
//是否是分隔线Item
@property (getter=isSeparatorItem, readonly) BOOL separatorItem;
//绑定的快捷键
@property (copy) NSString *keyEquivalent;
//快捷键类型
/*
typedef NS_OPTIONS(NSUInteger, NSEventModifierFlags) {
    NSEventModifierFlagCapsLock           = 1 << 16, // Caps lock键
    NSEventModifierFlagShift              = 1 << 17, // shift键
    NSEventModifierFlagControl            = 1 << 18, // control键
    NSEventModifierFlagOption             = 1 << 19, // option键
    NSEventModifierFlagCommand            = 1 << 20, // command键
    NSEventModifierFlagNumericPad         = 1 << 21, // 小键盘任意键
    NSEventModifierFlagHelp               = 1 << 22, // 帮助键
    NSEventModifierFlagFunction           = 1 << 23, // 任意功能按钮
};
*/
@property NSEventModifierFlags keyEquivalentModifierMask;
//Item图标
@property (nullable, strong) NSImage *image;
//Item状态
@property NSInteger state;
//开启状态下的图标
@property (null_resettable, strong) NSImage *onStateImage;
//关闭状态下的图标
@property (nullable, strong) NSImage *offStateImage;
//混合状态下的图标
@property (null_resettable, strong) NSImage *mixedStateImage;
//是否有效
@property (getter=isEnabled) BOOL enabled;
//是否前置
@property (getter=isAlternate) BOOL alternate;
//Item缩进级别
@property NSInteger indentationLevel;
//设置交互响应者
@property (nullable, weak) id target;
//设置交互相应方法
@property (nullable) SEL action;
//设置tag值
@property NSInteger tag;
//是否高亮
@property (getter=isHighlighted, readonly) BOOL highlighted;
//设置是否隐藏
@property (getter=isHidden) BOOL hidden;
//设置提示文本
@property (nullable, copy) NSString *toolTip;
```

### 六、NSMenu详解

```objectivec
//初始化方法
- (instancetype)initWithTitle:(NSString *)title;
//标题
@property (copy) NSString *title;
//在所在的交互点弹出菜单
+ (void)popUpContextMenu:(NSMenu*)menu withEvent:(NSEvent*)event forView:(NSView*)view;
+ (void)popUpContextMenu:(NSMenu*)menu withEvent:(NSEvent*)event forView:(NSView*)view withFont:(nullable NSFont*)font;
- (BOOL)popUpMenuPositioningItem:(nullable NSMenuItem *)item atLocation:(NSPoint)location inView:(nullable NSView *)view NS_AVAILABLE_MAC(10_6);
//设置菜单栏是否可见
+ (void)setMenuBarVisible:(BOOL)visible;
+ (BOOL)menuBarVisible;
//父菜单
@property (nullable, assign) NSMenu *supermenu;
//插入Item
- (void)insertItem:(NSMenuItem *)newItem atIndex:(NSInteger)index;
- (NSMenuItem *)insertItemWithTitle:(NSString *)string action:(nullable SEL)selector keyEquivalent:(NSString *)charCode atIndex:(NSInteger)index;
//添加Item
- (void)addItem:(NSMenuItem *)newItem;
- (NSMenuItem *)addItemWithTitle:(NSString *)string action:(nullable SEL)selector keyEquivalent:(NSString *)charCode;
//删除某个位置的Item
- (void)removeItemAtIndex:(NSInteger)index;
//删除Item
- (void)removeItem:(NSMenuItem *)item;
//为某个Item设置子菜单
- (void)setSubmenu:(nullable NSMenu *)menu forItem:(NSMenuItem *)item;
//删除所有Item
- (void)removeAllItems;
//Item数组
@property (readonly, copy) NSArray<NSMenuItem *> *itemArray;
//获取Item个数
@property (readonly) NSInteger numberOfItems;
//获取某个位置的Item
- (nullable NSMenuItem *)itemAtIndex:(NSInteger)index;
//获取某个Item的位置
- (NSInteger)indexOfItem:(NSMenuItem *)item;
- (NSInteger)indexOfItemWithTitle:(NSString *)title;
- (NSInteger)indexOfItemWithTag:(NSInteger)tag;
- (NSInteger)indexOfItemWithSubmenu:(nullable NSMenu *)submenu;
- (NSInteger)indexOfItemWithTarget:(nullable id)target andAction:(nullable SEL)actionSelector;
//根据标题获取item
- (nullable NSMenuItem *)itemWithTitle:(NSString *)title;
//根据tag获取Item
- (nullable NSMenuItem *)itemWithTag:(NSInteger)tag;
//刷新菜单
- (void)update;
//获取菜单高度
@property (readonly) CGFloat menuBarHeight;
//取消菜单
- (void)cancelTracking;
- (void)cancelTrackingWithoutAnimation;
//获取高亮的Item
@property (nullable, readonly, strong) NSMenuItem *highlightedItem;
//最小宽度
@property CGFloat minimumWidth;
//尺寸
@property (readonly) NSSize size;
//字体
@property (null_resettable, strong) NSFont *font;
```

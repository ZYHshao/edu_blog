---
title: OS X开发：下拉菜单按钮NSPopUpButton应用
date: 2017-07-24
categories: macOS开发
tags: []
---
## OS X开发：下拉菜单按钮NSPopUpButton应用

    NSPopUpButton是一个下拉按钮，当用户点击时，其会弹出一个下拉选择菜单。一个简单的示例如下：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    NSPopUpButton * popUpButton = [[NSPopUpButton alloc]initWithFrame:CGRectMake(100, 400, 200, 300)];
    //设置弹出菜单
    NSMenu * menu = [[NSMenu alloc]initWithTitle:@"menu"];
    [menu insertItemWithTitle:@"one" action:@selector(null) keyEquivalent:@"" atIndex:0];
    [menu addItemWithTitle:@"two" action:@selector(null) keyEquivalent:@""];
    popUpButton.menu = menu;
    //设置弹出菜单的位置
    popUpButton.preferredEdge = NSRectEdgeMaxX;
    [self.view addSubview:popUpButton];
}
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2017/0724/102801_SccG_2340880.png)

NSPopUpButton继承与NSButton，因此NSButton添加触发事件的方式在NSPopUpButton中依然使用，NSPopUpButton类中属性和方法解析如下：

```objectivec
//初始化方法 flag参数决定是下拉菜单模式还是弹出菜单模式
- (instancetype)initWithFrame:(NSRect)buttonFrame pullsDown:(BOOL)flag;
//设置下拉菜单
@property (nullable, strong) NSMenu *menu;
//设置当交互事件发生时，是否禁用选项
@property BOOL autoenablesItems;
//风格设置是否为下拉菜单
@property BOOL pullsDown;
//设置菜单弹出的优先位置
@property NSRectEdge preferredEdge;

//列表按钮相关
//添加一个按钮
- (void)addItemsWithTitles:(NSArray<NSString *> *)itemTitles;
//插入一个按钮
- (void)insertItemWithTitle:(NSString *)title atIndex:(NSInteger)index;
//通过标题移除一个按钮
- (void)removeItemWithTitle:(NSString *)title;
//通过索引移除按钮
- (void)removeItemAtIndex:(NSInteger)index;
//移除所有按钮
- (void)removeAllItems;
//所有列表选项按钮数组
@property (readonly, copy) NSArray<NSMenuItem *> *itemArray;
//按钮个数
@property (readonly) NSInteger numberOfItems;
//获取按钮索引的方法
- (NSInteger)indexOfItem:(NSMenuItem *)item;
- (NSInteger)indexOfItemWithTitle:(NSString *)title;
- (NSInteger)indexOfItemWithTag:(NSInteger)tag;
- (NSInteger)indexOfItemWithRepresentedObject:(nullable id)obj;
- (NSInteger)indexOfItemWithTarget:(nullable id)target andAction:(nullable SEL)actionSelector;
//获取按钮的方法
- (nullable NSMenuItem *)itemAtIndex:(NSInteger)index;
- (nullable NSMenuItem *)itemWithTitle:(NSString *)title;
//获取最后一个按钮
@property (nullable, readonly, strong) NSMenuItem *lastItem;
//选择某个按钮的方法
- (void)selectItem:(nullable NSMenuItem *)item;
- (void)selectItemAtIndex:(NSInteger)index;
- (void)selectItemWithTitle:(NSString *)title;
- (BOOL)selectItemWithTag:(NSInteger)tag;
- (void)setTitle:(NSString *)string;
//获取选中的按钮
@property (nullable, readonly, strong) NSMenuItem *selectedItem;
//获取已经选中的按钮索引
@property (readonly) NSInteger indexOfSelectedItem;
//获取已经选中的按钮tag
@property (readonly) NSInteger selectedTag;
//将选中的标题显示进行同步
- (void)synchronizeTitleAndSelectedItem;

//获取某个索引按钮的标题
- (NSString *)itemTitleAtIndex:(NSInteger)index;
//获取按钮标题数组
@property (readonly, copy) NSArray<NSString *> *itemTitles;
//获取选中的按钮标题
@property (nullable, readonly, copy) NSString *titleOfSelectedItem;
//当下拉菜单弹出时发送的通知
APPKIT_EXTERN NSNotificationName NSPopUpButtonWillPopUpNotification;
```

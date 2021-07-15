---
title: macOS开发之NSTableView的应用详解
date: 2017-04-25
categories: macOS开发
tags: []
---
## NSTableView的应用详解

### 一、引言

    和iOS开发中的UITableView有很大差别，NSTableView并非是一个可滚动的列表视图，其是一个不可滚动、支持多列多行的原始列表视图。若要使NSTableView支持滚动，通常会将其嵌套入NSScrollView控件中。与UITableView类似，NSTableView的数据也是用过DataSource代理来提供，通过Delegate代理来进行表格视图的定制化。在OS X v10.6版本之前，NSTableView中行数据载体视图必须是NSCell的子类，之后版本的OS X支持开发者创建基于View的TableView视图，同样也支持基于Cell的TabelView视图，在开发者，我们可以根据实际需求选择。

### 二、构建一个简单的列表视图

    首先新建一个测试工程，在ViewController.m文件中编写如下代码：

```objectivec
#import "ViewController.h"

@interface ViewController()<NSTableViewDelegate,NSTableViewDataSource>

@end

@implementation ViewController
{
    NSTableView * _tableView;
    NSMutableArray * _dataArray;
}
- (void)viewDidLoad {
    [super viewDidLoad];
    _dataArray = [NSMutableArray array];
    for (int i=0; i<20; i++) {
        [_dataArray addObject:[NSString stringWithFormat:@"%d行数据",i]];
    }
    NSScrollView * scrollView    = [[NSScrollView alloc] init];
    scrollView.hasVerticalScroller  = YES;
    scrollView.frame = self.view.bounds;
    [self.view addSubview:scrollView];
    _tableView = [[NSTableView alloc]initWithFrame:self.view.bounds];
    NSTableColumn * column = [[NSTableColumn alloc]initWithIdentifier:@"test"];
    [_tableView addTableColumn:column];
    _tableView.delegate = self;
    _tableView.dataSource = self;
    [_tableView reloadData];
    scrollView.contentView.documentView = _tableView;
}

-(NSInteger)numberOfRowsInTableView:(NSTableView *)tableView{
    return _dataArray.count;
}

-(id)tableView:(NSTableView *)tableView objectValueForTableColumn:(NSTableColumn *)tableColumn row:(NSInteger)row{
    return _dataArray[row];
}

@end
```

运行工程效果如下图：

![](https://static.oschina.net/uploads/space/2017/0418/135653_Gxzs_2340880.png)

这是一个最简单的TableView示例，但是细读代码，麻雀虽小五脏俱全。首先NSTableView中的列是由NSTableColumn类描述的。一个列表可以有多个列。也正如前面所说，numberOfRowsInTableView方法为数据源代理必须实现的方法，其中需要返回列表的行数。objectValueForTableColumn方法则是基于Cell的TableView必须实现的方法，其中需要返回每个列表行所填充的数据。

### 三、关于NSTableColume的探究

    NSTableColume简单理解就是一列，其中可以进行此列样式的相关设置，NSTableColumn类中常用属性解析如下：

```objectivec
//初始化方法，指定一个列ID
- (instancetype)initWithIdentifier:(NSString *)identifier;
//与此列关联的ID
@property (copy) NSString *identifier;
//关联的TableView
@property (nullable, assign) NSTableView *tableView;
//设置列宽度
@property CGFloat width;
//设置最小列宽度
@property CGFloat minWidth;
//设置最大列宽度
@property CGFloat maxWidth;
//设置类标题
@property (copy) NSString *title;
/*
列标题视图 开发者可以对其进行修改
需要注意，NSTableHeaderCell是继承自NSTextFieldCell
*/
@property (strong) __kindof NSTableHeaderCell *headerCell;
//设置此列是否可以进行编辑
@property (getter=isEditable) BOOL editable;
//进行列尺寸的调整 以列标题视图的宽度为标准 
- (void)sizeToFit;
//提供了这个属性，会在列标题那里显示一个排序按钮 点击列标题后可以进行排序操作(会回调相关协议方法)
@property (nullable, copy) NSSortDescriptor *sortDescriptorPrototype;
//设置列尺寸的调整模式 枚举如下
/*
typedef NS_OPTIONS(NSUInteger, NSTableColumnResizingOptions) {
    NSTableColumnNoResizing = 0, //不允许进行宽度调整
    //详见NSTabelView的columnAutoresizingStyle属性
    NSTableColumnAutoresizingMask = ( 1 << 0 ), //使用tableView的column调整策略
    NSTableColumnUserResizingMask = ( 1 << 1 ), //允许用户进行尺寸调整
};
*/
@property NSTableColumnResizingOptions resizingMask;
//设置列头的提示标题 当鼠标悬停在类标题上时  会显示此提示
@property (nullable, copy) NSString *headerToolTip;
//设置此列是否隐藏
@property (getter=isHidden) BOOL hidden;
//设置此列所有行的数据载体视图 如果不设置 默认为NSTextFieldCell
@property (strong) id dataCell;
//为TableView列表提供数据载体视图
- (id)dataCellForRow:(NSInteger)row;
```

### 四、Cell-Base：基于Cell的TableView视图

    Cell-Base是OS X早起版本中常用的构造TabelView的方式，其中每一行的数据载体都必须是NSCell的子类。如本文开头的示例代码，Cell-Base的TableView必须实现的两个协议方法是numberOfRowsInTableView和objectValueForTableColumn方法，第一个方法设置列表行数，第2个方法设置每个数据载体对应的具体数据。需要注意，如果只实现这两个方法，则NSTableView会自动从列对象NSTableColume中取具体的行视图，通过dataCellForRow方法。当objectValueForTableColumn方法将每个行具体的数据返回后，会调用cell的setObjectValue方法(因此如果要自定义cell，必须实现这个方法)。如果我们要对Cell的渲染进行一些定制，可以在如下方法中实现：

```objectivec
//将要渲染cell调用的方法 开发者可以拿到cell对象做定制
- (void)tableView:(NSTableView *)tableView willDisplayCell:(id)cell forTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row;
```

实现下面的方法可以返回一个自定义的Cell，如果实现了这个方法，则TableView不会再从NSTableColumn对象中拿Cell实例：

```objectivec
//返回自定义的Cell实例
/*
需要注意，这个方法在第一次调用的时候 tableColumu对象是nil 如果这时返回了Cell，则此Cell宽度会覆盖整个列表
在使用时要多加注意
*/
- (nullable NSCell *)tableView:(NSTableView *)tableView dataCellForTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row;
```

其他方法的实例代码如下：

```objectivec
#import "ViewController.h"
#import "MyCell.h"
@interface ViewController()<NSTableViewDelegate,NSTableViewDataSource>

@end

@implementation ViewController
{
    NSTableView * _tableView;
    NSMutableArray * _dataArray;
}
- (void)viewDidLoad {
    [super viewDidLoad];
    _dataArray = [NSMutableArray array];
    for (int i=0; i<20; i++) {
        [_dataArray addObject:[NSString stringWithFormat:@"%d行数据",i]];
    }
    NSScrollView * scrollView    = [[NSScrollView alloc] init];
    scrollView.hasVerticalScroller  = YES;
    scrollView.frame = self.view.bounds;
    [self.view addSubview:scrollView];
    _tableView = [[NSTableView alloc]initWithFrame:self.view.bounds];
    NSTableColumn * column = [[NSTableColumn alloc]initWithIdentifier:@"test"];
    NSTableColumn * column2 = [[NSTableColumn alloc]initWithIdentifier:@"test2"];
    column2.width = 100;
    column2.minWidth = 100;
    column2.maxWidth = 100;
    column2.title = @"数据";
    column2.editable = YES ;
    column2.headerToolTip = @"提示";
    column2.hidden=NO;
    column2.sortDescriptorPrototype = [NSSortDescriptor sortDescriptorWithKey:@"title" ascending:NO];
    column.resizingMask =NSTableColumnUserResizingMask;
//    column.dataCell = [[NSButtonCell alloc]initTextCell:@""];
    [_tableView addTableColumn:column];
    [_tableView addTableColumn:column2];
    _tableView.delegate = self;
    _tableView.dataSource = self;
    scrollView.contentView.documentView = _tableView;
}

//设置行数 通用
-(NSInteger)numberOfRowsInTableView:(NSTableView *)tableView{
    return _dataArray.count;
}
//绑定数据
-(id)tableView:(NSTableView *)tableView objectValueForTableColumn:(NSTableColumn *)tableColumn row:(NSInteger)row{
    return _dataArray[row];
}
//用户编辑列表
- (void)tableView:(NSTableView *)tableView setObjectValue:(nullable id)object forTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row{
    NSLog(@"%@",object);
    _dataArray[row] = object;
}
//cell-base的cell展示前调用 可以进行自定制
- (void)tableView:(NSTableView *)tableView willDisplayCell:(id)cell forTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row{
    NSTextFieldCell * _cell = cell;
   _cell.textColor = [NSColor redColor];
}
//设置是否可以进行编辑
- (BOOL)tableView:(NSTableView *)tableView shouldEditTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row{
    return YES;
}
//设置鼠标悬停在cell上显示的提示文本
- (NSString *)tableView:(NSTableView *)tableView toolTipForCell:(NSCell *)cell rect:(NSRectPointer)rect tableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row mouseLocation:(NSPoint)mouseLocation{
    return @"tip";
}
//当列表长度无法展示完整某行数据时 当鼠标悬停在此行上 是否扩展显示
- (BOOL)tableView:(NSTableView *)tableView shouldShowCellExpansionForTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row{
    return YES;
}
//设置cell的交互能力
/*
如果返回YES，则Cell的交互能力会变强，例如NSButtonCell的点击将会调用- (void)tableView:(NSTableView *)tableView setObjectValue方法
*/
- (BOOL)tableView:(NSTableView *)tableView shouldTrackCell:(NSCell *)cell forTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row{
    return YES;
}
//自定义cell
- (nullable NSCell *)tableView:(NSTableView *)tableView dataCellForTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row{
    if (tableColumn!=nil) {
        MyCell * cell = [[MyCell alloc]init];
        return cell;
    }
    return nil;
    
}
-(CGFloat)tableView:(NSTableView *)tableView heightOfRow:(NSInteger)row{
    return 30;
}
//排序回调函数
-(void)tableView:(NSTableView *)tableView sortDescriptorsDidChange:(NSArray<NSSortDescriptor *> *)oldDescriptors{
    NSLog(@"%@",oldDescriptors[0]);
}

@end
```

### 五、View-Base：基于View的TableView视图

    基于View-Base的TableView要比基于Cell的TableView更加灵活，其中每行数据载体可以是任意NSView的子类。代码示例如下：

```objectivec
//
//  ViewController.m
//  TableView
//
//  Created by jaki on 17/4/14.
//  Copyright © 2017年 jaki. All rights reserved.
//

#import "ViewController.h"
#import "MyCell.h"
#import "TableRow.h"
@interface ViewController()<NSTableViewDelegate,NSTableViewDataSource>

@end

@implementation ViewController
{
    NSTableView * _tableView;
    NSMutableArray * _dataArray;
}
- (void)viewDidLoad {
    [super viewDidLoad];
    _dataArray = [NSMutableArray array];
    for (int i=0; i<20; i++) {
        [_dataArray addObject:[NSString stringWithFormat:@"%d行数据",i]];
    }
    NSScrollView * scrollView    = [[NSScrollView alloc] init];
    scrollView.hasVerticalScroller  = YES;
    scrollView.frame = self.view.bounds;
    [self.view addSubview:scrollView];
    _tableView = [[NSTableView alloc]initWithFrame:self.view.bounds];
    NSTableColumn * column = [[NSTableColumn alloc]initWithIdentifier:@"test"];
    NSTableColumn * column2 = [[NSTableColumn alloc]initWithIdentifier:@"test2"];
    column2.width = 100;
    column2.minWidth = 100;
    column2.maxWidth = 100;
    column2.title = @"数据";
    column2.editable = YES ;
    column2.headerToolTip = @"提示";
    column2.hidden=NO;
    column2.sortDescriptorPrototype = [NSSortDescriptor sortDescriptorWithKey:@"title" ascending:NO];
    column.resizingMask =NSTableColumnUserResizingMask;
    _tableView.delegate = self;
    _tableView.dataSource = self;
    [_tableView addTableColumn:column];
    [_tableView addTableColumn:column2];
    scrollView.contentView.documentView = _tableView;
}

//设置行数 通用
-(NSInteger)numberOfRowsInTableView:(NSTableView *)tableView{
    return _dataArray.count;
}
//View-base
//设置某个元素的具体视图
- (nullable NSView *)tableView:(NSTableView *)tableView viewForTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row{
    //根据ID取视图
    NSTextField * view = [tableView makeViewWithIdentifier:@"cellId" owner:self];
    if (view==nil) {
        view = [[NSTextField alloc]initWithFrame:CGRectMake(0, 0, 100, 30)];
        view.backgroundColor = [NSColor clearColor];
        view.identifier = @"cellId";
    }
    return view;
}
//设置每行容器视图
- (nullable NSTableRowView *)tableView:(NSTableView *)tableView rowViewForRow:(NSInteger)row{
    TableRow * rowView = [[TableRow alloc]init];
    return rowView;
}
//当添加行时调用的回调
- (void)tableView:(NSTableView *)tableView didAddRowView:(NSTableRowView *)rowView forRow:(NSInteger)row{
    NSLog(@"add");
}
//当移除行时调用的回调
- (void)tableView:(NSTableView *)tableView didRemoveRowView:(NSTableRowView *)rowView forRow:(NSInteger)row{
    NSLog(@"remove");
}
@end

```

上面代码中用到了TableRow类，其实它是一个自定义的继承自NSTableRowView的类，实现如下：

```objectivec
#import "TablerRow.h"
@implementation TablerRow
//绘制选中状态的背景
-(void)drawSelectionInRect:(NSRect)dirtyRect{
    NSRect selectionRect = NSInsetRect(self.bounds, 5.5, 5.5);
    [[NSColor colorWithCalibratedWhite:.72 alpha:1.0] setStroke];
    [[NSColor colorWithCalibratedWhite:.82 alpha:1.0] setFill];
    NSBezierPath *selectionPath = [NSBezierPath bezierPathWithRoundedRect:selectionRect xRadius:10 yRadius:10];
    [selectionPath fill];
    [selectionPath stroke];
}
//绘制背景
-(void)drawBackgroundInRect:(NSRect)dirtyRect{
    [super drawBackgroundInRect:dirtyRect];
    [[NSColor greenColor]setFill];
    NSRectFill(dirtyRect);
}
@end

```

关于NSTableRowView类我们下面来做具体介绍。

### 六、NSTableRowView解析

    NSTableRowView用在View-Base的TableView中，其作为行容器存在。

```objectivec
//选中的高亮风格
/*
typedef NS_ENUM(NSInteger, NSTableViewSelectionHighlightStyle) {
    //无高亮风格
    NSTableViewSelectionHighlightStyleNone,
    //规则的高亮风格
    NSTableViewSelectionHighlightStyleRegular = 0,
    //源列表风格
    NSTableViewSelectionHighlightStyleSourceList = 1,
};
*/
@property NSTableViewSelectionHighlightStyle selectionHighlightStyle;
//是否强调
@property(getter=isEmphasized) BOOL emphasized;
//设置是否行组风格
@property(getter=isGroupRowStyle) BOOL groupRowStyle;
//是否选中状态
@property(getter=isSelected) BOOL selected;
//其前一行的选中状态
@property(getter=isPreviousRowSelected) BOOL previousRowSelected;
//其后一行的选中状态
@property(getter=isNextRowSelected) BOOL nextRowSelected;
//设置此行是否浮动
@property(getter=isFloating) BOOL floating;
//拖放拖动效果
@property(getter=isTargetForDropOperation) BOOL targetForDropOperation;
//拖放风格
@property NSTableViewDraggingDestinationFeedbackStyle draggingDestinationFeedbackStyle;
//设置拖放目标的缩进量
@property CGFloat indentationForDropOperation;
//背景色
@property(copy) NSColor *backgroundColor;

//子类重写下面方法来进行行容器视图的自定义
//画背景色
- (void)drawBackgroundInRect:(NSRect)dirtyRect;
//画选中背景
- (void)drawSelectionInRect:(NSRect)dirtyRect;
//画分割线
- (void)drawSeparatorInRect:(NSRect)dirtyRect;
//绘制拖放时的用户反馈IU
- (void)drawDraggingDestinationFeedbackInRect:(NSRect)dirtyRect;

//列数
@property(readonly) NSInteger numberOfColumns;
//提供的访问特定视图的方法
- (nullable id)viewAtColumn:(NSInteger)column;
```

### 七、来总结下NSTableViewDataSource协议

```objectivec
/*
无论基于Cell还是基于View，这个方法都需要实现，用来设置列表的行数
*/
- (NSInteger)numberOfRowsInTableView:(NSTableView *)tableView;
/*
如果使用cell-base的TableView视图，这个方法是必须实现的，其为要渲染的cell提供数据
*/
- (nullable id)tableView:(NSTableView *)tableView objectValueForTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row;
/*
这个函数当用户编辑了cell中的内容时会被调用，一般需要在其中进行数据源的修改
*/
- (void)tableView:(NSTableView *)tableView setObjectValue:(nullable id)object forTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row;
/*
当用户修改了行排序规则时调用的回调
*/
- (void)tableView:(NSTableView *)tableView sortDescriptorsDidChange:(NSArray<NSSortDescriptor *> *)oldDescriptors;

//下面这些方法全部与列表的数据拖拽相关
- (nullable id <NSPasteboardWriting>)tableView:(NSTableView *)tableView pasteboardWriterForRow:(NSInteger)row;
- (void)tableView:(NSTableView *)tableView draggingSession:(NSDraggingSession *)session willBeginAtPoint:(NSPoint)screenPoint forRowIndexes:(NSIndexSet *)rowIndexes NS_AVAILABLE_MAC(10_7);
- (void)tableView:(NSTableView *)tableView draggingSession:(NSDraggingSession *)session endedAtPoint:(NSPoint)screenPoint operation:(NSDragOperation)operation NS_AVAILABLE_MAC(10_7);
- (void)tableView:(NSTableView *)tableView updateDraggingItemsForDrag:(id <NSDraggingInfo>)draggingInfo NS_AVAILABLE_MAC(10_7);
- (BOOL)tableView:(NSTableView *)tableView writeRowsWithIndexes:(NSIndexSet *)rowIndexes toPasteboard:(NSPasteboard *)pboard;
- (NSDragOperation)tableView:(NSTableView *)tableView validateDrop:(id <NSDraggingInfo>)info proposedRow:(NSInteger)row proposedDropOperation:(NSTableViewDropOperation)dropOperation;
- (BOOL)tableView:(NSTableView *)tableView acceptDrop:(id <NSDraggingInfo>)info row:(NSInteger)row dropOperation:(NSTableViewDropOperation)dropOperation;
- (NSArray<NSString *> *)tableView:(NSTableView *)tableView namesOfPromisedFilesDroppedAtDestination:(NSURL *)dropDestination forDraggedRowsWithIndexes:(NSIndexSet *)indexSet;
```

### 八、来总结下NSTableViewDelegate协议

```objectivec
//view-base的TableView相关delegate方法
/*
设置每个数据载体的View
*/
- (nullable NSView *)tableView:(NSTableView *)tableView viewForTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row;
/*
自定义行视图
*/
- (nullable NSTableRowView *)tableView:(NSTableView *)tableView rowViewForRow:(NSInteger)row NS_AVAILABLE_MAC(10_7);
/*
添加一行时会调用的回调
*/
- (void)tableView:(NSTableView *)tableView didAddRowView:(NSTableRowView *)rowView forRow:(NSInteger)row;
/*
移除一行时会调用的回调
*/
- (void)tableView:(NSTableView *)tableView didRemoveRowView:(NSTableRowView *)rowView forRow:(NSInteger)row;

//cell-base的TableView相关delegate方法
/*
cell将要渲染时调用的回调，可以在其中对cell进行定制
*/
- (void)tableView:(NSTableView *)tableView willDisplayCell:(id)cell forTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row;
/*
设置某个cell是否可以编辑
*/
- (BOOL)tableView:(NSTableView *)tableView shouldEditTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row;
/*
设置当鼠标悬停在cell上时 显示的提示文案
*/
- (NSString *)tableView:(NSTableView *)tableView toolTipForCell:(NSCell *)cell rect:(NSRectPointer)rect tableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row mouseLocation:(NSPoint)mouseLocation;
/*
当cell的宽度不够显示完全cell的内容时，设置是否允许鼠标放置扩展cell
*/
- (BOOL)tableView:(NSTableView *)tableView shouldShowCellExpansionForTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row;
/*
设置是否加强cell的交互能力，这样一些按钮状态的修改也会触发cell编辑的状态
*/
- (BOOL)tableView:(NSTableView *)tableView shouldTrackCell:(NSCell *)cell forTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row;
/*
设置自定义cell
*/
- (nullable NSCell *)tableView:(NSTableView *)tableView dataCellForTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row;

//通用的TableView代理方法
/*
设置是否允许修改选中
*/
- (BOOL)selectionShouldChangeInTableView:(NSTableView *)tableView;
/*
设置某行是否可以选中
*/
- (BOOL)tableView:(NSTableView *)tableView shouldSelectRow:(NSInteger)row;
/*
当用户通过键盘或鼠标将要选中某行时，返回设置要选中的行
如果实现了这个方法，上面一个方法将不会被调用
*/
- (NSIndexSet *)tableView:(NSTableView *)tableView selectionIndexesForProposedSelection:(NSIndexSet *)proposedSelectionIndexes;
/*
设置某列是否可以被选中
*/
- (BOOL)tableView:(NSTableView *)tableView shouldSelectTableColumn:(nullable NSTableColumn *)tableColumn;
/*
用户点击列头时调用的方法
*/
- (void)tableView:(NSTableView *)tableView mouseDownInHeaderOfTableColumn:(NSTableColumn *)tableColumn;
/*
用法同上
*/
- (void)tableView:(NSTableView *)tableView didClickTableColumn:(NSTableColumn *)tableColumn;
/*
对列进行拖拽改变顺序时调用的方法
*/
- (void)tableView:(NSTableView *)tableView didDragTableColumn:(NSTableColumn *)tableColumn;
/*
设置行高
*/
- (CGFloat)tableView:(NSTableView *)tableView heightOfRow:(NSInteger)row;
/*
下面这些方法与行检索有关
*/
- (nullable NSString *)tableView:(NSTableView *)tableView typeSelectStringForTableColumn:(nullable NSTableColumn *)tableColumn row:(NSInteger)row NS_AVAILABLE_MAC(10_5);
- (NSInteger)tableView:(NSTableView *)tableView nextTypeSelectMatchFromRow:(NSInteger)startRow toRow:(NSInteger)endRow forString:(NSString *)searchString NS_AVAILABLE_MAC(10_5);
- (BOOL)tableView:(NSTableView *)tableView shouldTypeSelectForEvent:(NSEvent *)event withCurrentSearchString:(nullable NSString *)searchString NS_AVAILABLE_MAC(10_5);
/*
设置某行是否绘制成组样式
*/
- (BOOL)tableView:(NSTableView *)tableView isGroupRow:(NSInteger)row;
/*
调整列宽度
*/
- (CGFloat)tableView:(NSTableView *)tableView sizeToFitWidthOfColumn:(NSInteger)column;
/*
设置是否支持列的移动排序
*/
- (BOOL)tableView:(NSTableView *)tableView shouldReorderColumn:(NSInteger)columnIndex toColumn:(NSInteger)newColumnIndex;
//设置某行向左或向右滑动时要显示的功能按钮
/*
typedef NS_ENUM(NSInteger, NSTableRowActionEdge) {
    NSTableRowActionEdgeLeading, // 左划
    NSTableRowActionEdgeTrailing, // 右划
} NS_ENUM_AVAILABLE_MAC(10_11);
*/
- (NSArray<NSTableViewRowAction *> *)tableView:(NSTableView *)tableView rowActionsForRow:(NSInteger)row edge:(NSTableRowActionEdge)edge NS_AVAILABLE_MAC(10_11);
/*
TableView选中修改时调用
*/
- (void)tableViewSelectionDidChange:(NSNotification *)notification;
/*
TableView列移动完成时调用的函数
*/
- (void)tableViewColumnDidMove:(NSNotification *)notification;
/*
TableView列宽度变化时调用的函数
*/
- (void)tableViewColumnDidResize:(NSNotification *)notification;
/*
TableView选中正在修改时调用的函数
*/
- (void)tableViewSelectionIsChanging:(NSNotification *)notification;
```

### 九、NSTableView中常用的属性和方法

```objectivec
//初始化方法
- (instancetype)initWithFrame:(NSRect)frameRect;
- (nullable instancetype)initWithCoder:(NSCoder *)coder;

//设置代理
@property (nullable, weak) id <NSTableViewDataSource> dataSource;
@property (nullable, weak) id <NSTableViewDelegate> delegate;

//设置TableView的头视图 会被列头图就行覆盖
@property (nullable, strong) NSTableHeaderView *headerView;
//设置头图右侧视图 可以自定义图标
@property (nullable, strong) NSView *cornerView;
//设置是否允许列拖拽排序
@property BOOL allowsColumnReordering;
//设置是否允许调整列宽度
@property BOOL allowsColumnResizing;
//调整列宽度的风格
/*
typedef NS_ENUM(NSUInteger, NSTableViewColumnAutoresizingStyle) {
    //不可调整
    NSTableViewNoColumnAutoresizing = 0,
    //平分
    NSTableViewUniformColumnAutoresizingStyle,
    //从后往前调整
    NSTableViewSequentialColumnAutoresizingStyle,  
    //从前往后调整
    NSTableViewReverseSequentialColumnAutoresizingStyle, 
    //最后一列可调整
    NSTableViewLastColumnOnlyAutoresizingStyle,
    //第一列可调整
    NSTableViewFirstColumnOnlyAutoresizingStyle
};
*/
@property NSTableViewColumnAutoresizingStyle columnAutoresizingStyle;
//设置分割线风格
/*
typedef NS_OPTIONS(NSUInteger, NSTableViewGridLineStyle) {
    //无分割线
    NSTableViewGridNone                    = 0,
    //竖直分割线
    NSTableViewSolidVerticalGridLineMask   = 1 << 0,
    //水平分割线
    NSTableViewSolidHorizontalGridLineMask = 1 << 1,
    //水平虚线分割线
    NSTableViewDashedHorizontalGridLineMask ,
};
*/
@property NSTableViewGridLineStyle gridStyleMask;
//设置cell之间的间隔 需要设置为NSSize对象
@property NSSize intercellSpacing;
//是否开启斑马纹
@property BOOL usesAlternatingRowBackgroundColors;
//背景色
@property (copy) NSColor *backgroundColor;
//设置分割线颜色
@property (copy) NSColor *gridColor;
//设置行尺寸风格
/*
typedef NS_ENUM(NSInteger, NSTableViewRowSizeStyle) {
    //默认
    NSTableViewRowSizeStyleDefault = -1,
    //自定义
    NSTableViewRowSizeStyleCustom = 0,
    //小尺寸风格
    NSTableViewRowSizeStyleSmall = 1,
    //中等尺寸风格
    NSTableViewRowSizeStyleMedium = 2,
    //大尺寸风格
    NSTableViewRowSizeStyleLarge = 3,
} NS_ENUM_AVAILABLE_MAC(10_7);
*/
@property NSTableViewRowSizeStyle rowSizeStyle;
//行高
@property CGFloat rowHeight;
//获取所有列对象
@property (readonly, copy) NSArray<NSTableColumn *> *tableColumns;
//获取列数
@property (readonly) NSInteger numberOfColumns;
//获取行数
@property (readonly) NSInteger numberOfRows;
//添加一列
- (void)addTableColumn:(NSTableColumn *)tableColumn;
//移除一列
- (void)removeTableColumn:(NSTableColumn *)tableColumn;
//移动列
- (void)moveColumn:(NSInteger)oldIndex toColumn:(NSInteger)newIndex;
//根据id获取列的下标
- (NSInteger)columnWithIdentifier:(NSString *)identifier;
//根据id获取列对象
- (nullable NSTableColumn *)tableColumnWithIdentifier:(NSString *)identifier;
//滚动到指定行可见
- (void)scrollRowToVisible:(NSInteger)row;
//滚动到指定列可见
- (void)scrollColumnToVisible:(NSInteger)column;
//重新加载数据
- (void)reloadData;
//重新加载指定位置的数据
- (void)reloadDataForRowIndexes:(NSIndexSet *)rowIndexes columnIndexes:(NSIndexSet *)columnIndexes;
//获取编辑的列
@property (readonly) NSInteger editedColumn;
//获取编辑的行
@property (readonly) NSInteger editedRow;
//获取点击的列
@property (readonly) NSInteger clickedColumn;
//获取点击的行
@property (readonly) NSInteger clickedRow;
//设置列头提示图片
- (void)setIndicatorImage:(nullable NSImage *)image inTableColumn:(NSTableColumn *)tableColumn;
//获取列头提示图片
- (nullable NSImage *)indicatorImageInTableColumn:(NSTableColumn *)tableColumn;

//下面这些方法与列表拖拽有关
@property BOOL verticalMotionCanBeginDrag;
- (BOOL)canDragRowsWithIndexes:(NSIndexSet *)rowIndexes atPoint:(NSPoint)mouseDownPoint;
- (NSImage *)dragImageForRowsWithIndexes:(NSIndexSet *)dragRows tableColumns:(NSArray<NSTableColumn *> *)tableColumns event:(NSEvent *)dragEvent offset:(NSPointPointer)dragImageOffset;
- (void)setDraggingSourceOperationMask:(NSDragOperation)mask forLocal:(BOOL)isLocal;
- (void)setDropRow:(NSInteger)row dropOperation:(NSTableViewDropOperation)dropOperation;

//下面这些方法与列表选中有关
//是否支持多选
@property BOOL allowsMultipleSelection;
//是否允许都不选中
@property BOOL allowsEmptySelection;
//是否支持选中列 如果设置为YES 点击列头会将整列选中
@property BOOL allowsColumnSelection;
//全选 用于子类重写
- (void)selectAll:(nullable id)sender;
//全不选 用于子类重写
- (void)deselectAll:(nullable id)sender;
//进行列选中
- (void)selectColumnIndexes:(NSIndexSet *)indexes byExtendingSelection:(BOOL)extend;
//进行行选中
- (void)selectRowIndexes:(NSIndexSet *)indexes byExtendingSelection:(BOOL)extend;
//获取所有选中列index
@property (readonly, copy) NSIndexSet *selectedColumnIndexes;
//获取所有选中行index
@property (readonly, copy) NSIndexSet *selectedRowIndexes;
//取消某列的选中
- (void)deselectColumn:(NSInteger)column;
//取消某行的选中
- (void)deselectRow:(NSInteger)row;
//判断某列是否被选中
- (BOOL)isColumnSelected:(NSInteger)column;
//判断某行是否被选中
- (BOOL)isRowSelected:(NSInteger)row;
//获取选中的列数
@property (readonly) NSInteger numberOfSelectedColumns;
//获取选中的行数
@property (readonly) NSInteger numberOfSelectedRows;
//获取某列的位置尺寸
- (NSRect)rectOfColumn:(NSInteger)column;
//获取某行的位置尺寸
- (NSRect)rectOfRow:(NSInteger)row;
//获取某个范围内的列
- (NSIndexSet *)columnIndexesInRect:(NSRect)rect;
//获取某个范围内的行
- (NSRange)rowsInRect:(NSRect)rect;
//获取包含某个点的列
- (NSInteger)columnAtPoint:(NSPoint)point;
//获取包含某个点的行
- (NSInteger)rowAtPoint:(NSPoint)point;
//获取某个cell的位置尺寸
- (NSRect)frameOfCellAtColumn:(NSInteger)column row:(NSInteger)row;
//获取某个位置的View，用于view-base
- (nullable __kindof NSView *)viewAtColumn:(NSInteger)column row:(NSInteger)row makeIfNecessary:(BOOL)makeIfNecessary;
//获取某行的视图 用于view-base
- (nullable __kindof NSTableRowView *)rowViewAtRow:(NSInteger)row makeIfNecessary:(BOOL)makeIfNecessary;
//获取某个View所在的行 用于view-base
- (NSInteger)rowForView:(NSView *)view;
//获取某个View所在的列 用于view-base
- (NSInteger)columnForView:(NSView *)view;
//创建一个用于渲染的View 用于view-base
- (nullable __kindof NSView *)makeViewWithIdentifier:(NSString *)identifier owner:(nullable id)owner;

//下面这些方法用来根据列表数据
//开始更新
- (void)beginUpdates NS_AVAILABLE_MAC(10_7);
//结束更新
- (void)endUpdates NS_AVAILABLE_MAC(10_7);
//插入行
- (void)insertRowsAtIndexes:(NSIndexSet *)indexes withAnimation:(NSTableViewAnimationOptions)animationOptions NS_AVAILABLE_MAC(10_7);
//删除行
- (void)removeRowsAtIndexes:(NSIndexSet *)indexes withAnimation:(NSTableViewAnimationOptions)animationOptions NS_AVAILABLE_MAC(10_7);
//移动行
- (void)moveRowAtIndex:(NSInteger)oldIndex toIndex:(NSInteger)newIndex NS_AVAILABLE_MAC(10_7);
//隐藏行
- (void)hideRowsAtIndexes:(NSIndexSet *)indexes withAnimation:(NSTableViewAnimationOptions)rowAnimation NS_AVAILABLE_MAC(10_11);
//取消隐藏行
- (void)unhideRowsAtIndexes:(NSIndexSet *)indexes withAnimation:(NSTableViewAnimationOptions)rowAnimation NS_AVAILABLE_MAC(10_11);
//所有隐藏状态的行
@property (readonly, copy) NSIndexSet *hiddenRowIndexes;
```

### 十、相关通知

```objectivec
//列表选择改变后发的通知
APPKIT_EXTERN NSNotificationName NSTableViewSelectionDidChangeNotification;
//列移动后发的通知
APPKIT_EXTERN NSNotificationName NSTableViewColumnDidMoveNotification;    
//列宽度改变后发的通知
APPKIT_EXTERN NSNotificationName NSTableViewColumnDidResizeNotification;
//选择改变时发的通知   
APPKIT_EXTERN NSNotificationName NSTableViewSelectionIsChangingNotification;
```

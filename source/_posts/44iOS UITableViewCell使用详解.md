---
title: iOS UITableViewCell使用详解
date: 2015-05-04
categories: iOS之UI控件
tags: []
---
## iOS中UITableViewCell使用详解

\- (instancetype)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier;

Cell的初始化方法，可以设置一个风格和标识符，风格的枚举如下：

```
typedef NS_ENUM(NSInteger, UITableViewCellStyle) {
    UITableViewCellStyleDefault,    // 默认风格，自带标题和一个图片视图，图片在左
    UITableViewCellStyleValue1,        // 只有标题和副标题 副标题在右边
    UITableViewCellStyleValue2,        // 只有标题和副标题，副标题在左边标题的下边
    UITableViewCellStyleSubtitle    // 自带图片视图和主副标题，主副标题都在左边，副标题在下
};
```

[@property](http://my.oschina.net/property) (nonatomic, readonly, retain) UIImageView *imageView;

图片视图，风格允许时才会创建

[@property](http://my.oschina.net/property) (nonatomic, readonly, retain) UILabel *textLabel;

标题标签

[@property](http://my.oschina.net/property) (nonatomic, readonly, retain) UILabel *detailTextLabel;

副标题标签

[@property](http://my.oschina.net/property) (nonatomic, readonly, retain) UIView      *contentView;

容纳视图，任何cell的子视图都应该添加在这个上面

[@property](http://my.oschina.net/property) (nonatomic, retain) UIView                *backgroundView;

背景视图

@property (nonatomic, retain) UIView                *selectedBackgroundView;

选中状态下的背景视图

@property (nonatomic, retain) UIView              *multipleSelectionBackgroundView;

多选选中时的背景视图

@property (nonatomic, readonly, copy) NSString      *reuseIdentifier;

cell的标识符

\- (void)prepareForReuse; 

当被重用的cell将要显示时，会调用这个方法，这个方法最大的用武之地是当你自定义的cell上面有图片时，如果产生了重用，图片可能会错乱（当图片来自异步下载时及其明显），这时我们可以重写这个方法把内容抹掉。

@property (nonatomic) UITableViewCellSelectionStyle selectionStyle;

cell被选中时的风格，枚举如下：

```
typedef NS_ENUM(NSInteger, UITableViewCellSelectionStyle) {
    UITableViewCellSelectionStyleNone,//无
    UITableViewCellSelectionStyleBlue,//蓝色
    UITableViewCellSelectionStyleGray,//灰色
    UITableViewCellSelectionStyleDefault//默认 为蓝色
};
```

@property (nonatomic, getter=isSelected) BOOL         selected;  

设置cell是否选中状态

@property (nonatomic, getter=isHighlighted) BOOL      highlighted;   

设置cell是否高亮状态

\- (void)setSelected:(BOOL)selected animated:(BOOL)animated;  

\- (void)setHighlighted:(BOOL)highlighted animated:(BOOL)animated; 

与上面的两个属性对应

@property (nonatomic, readonly) UITableViewCellEditingStyle editingStyle;

获取cell的编辑状态，枚举如下

```
typedef NS_ENUM(NSInteger, UITableViewCellEditingStyle) {
    UITableViewCellEditingStyleNone,//无编辑
    UITableViewCellEditingStyleDelete,//删除编辑
    UITableViewCellEditingStyleInsert//插入编辑
};
```

@property (nonatomic) BOOL                            showsReorderControl; 

设置是否显示cell自带的自动排序控件

注意：要让cell实现拖动排序的功能，除了上面设置为YES，还需实现代理中的如下方法：

-(BOOL)tableView:(UITableView *)tableView canMoveRowAtIndexPath:(NSIndexPath *)indexPath{

 return  YES;

}

-(void)tableView:(UITableView *)tableView moveRowAtIndexPath:(NSIndexPath *)sourceIndexPath toIndexPath:(NSIndexPath *)destinationIndexPath{

}

@property (nonatomic) BOOL                            shouldIndentWhileEditing;

设置编辑状态下是否显示缩进

@property (nonatomic) UITableViewCellAccessoryType    accessoryType; 

设置附件视图的风格(cell最右侧显示的视图) 枚举如下：

```
typedef NS_ENUM(NSInteger, UITableViewCellAccessoryType) {
    UITableViewCellAccessoryNone,                   // 没有视图
    UITableViewCellAccessoryDisclosureIndicator,    // cell右侧显示一个灰色箭头
    UITableViewCellAccessoryDetailDisclosureButton, // 显示详情符号和灰色箭头
    UITableViewCellAccessoryCheckmark,              // cell右侧显示蓝色对号
    UITableViewCellAccessoryDetailButton  // cell右侧显示一个详情符号
};
```

@property (nonatomic, retain) UIView                 *accessoryView;  

附件视图

@property (nonatomic) UITableViewCellAccessoryType    editingAccessoryType; 

cell编辑时的附件视图风格

@property (nonatomic, retain) UIView                 *editingAccessoryView;  

cell编辑时的附件视图

@property (nonatomic) NSInteger                       indentationLevel; 

设置内容区域的缩进级别

@property (nonatomic) CGFloat                         indentationWidth; 

设置每个级别的缩进宽度

@property (nonatomic) UIEdgeInsets                    separatorInset;

设置分割线的偏移量

@property (nonatomic, getter=isEditing) BOOL          editing; 

\- (void)setEditing:(BOOL)editing animated:(BOOL)animated;

设置是否编辑状态

@property(nonatomic, readonly) BOOL                   showingDeleteConfirmation;

返回是否目前正在显示删除按钮

\- (void)willTransitionToState:(UITableViewCellStateMask)state;

cell状态将要转换时调用的函数，可以在子类中重写

\- (void)didTransitionToState:(UITableViewCellStateMask)state;

cell状态已经转换时调用的函数，可以在子类中重写，状态枚举如下：

```
typedef NS_OPTIONS(NSUInteger, UITableViewCellStateMask) {
    UITableViewCellStateDefaultMask                     = 0,//默认状态
    UITableViewCellStateShowingEditControlMask          = 1 << 0,//编辑状态
    UITableViewCellStateShowingDeleteConfirmationMask   = 1 << 1//确认删除状态
};
```

注意：下面这些方法已经全部在IOS3.0后被废弃了，虽然还有效果，但是会被警告

@property (nonatomic, copy) NSString *text;

设置标题

@property (nonatomic, retain) UIFont *font;

设置字体

@property (nonatomic) NSTextAlignment   textAlignment;

设置对其模式

@property (nonatomic) NSLineBreakMode   lineBreakMode;

设置断行模式

@property (nonatomic, retain) UIColor  *textColor;

设置字体颜色

@property (nonatomic, retain) UIColor  *selectedTextColor;

设置选中状态下的字体颜色

@property (nonatomic, retain) UIImage  *image;

设置图片

@property (nonatomic, retain) UIImage  *selectedImage;

设置选中状态时的图片

@property (nonatomic) BOOL              hidesAccessoryWhenEditing;

设置编辑的时候是否隐藏附件视图

学习使用 欢迎转载

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

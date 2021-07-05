---
title: iOS中UISearchBar(搜索框)使用总结
date: 2015-09-22
categories: iOS之UI控件
tags: []
---
## iOS中UISearchBar(搜索框)使用总结

初始化：UISearchBar继承于UIView，我们可以像创建View那样创建searchBar

```
    UISearchBar * bar = [[UISearchBar alloc]initWithFrame:CGRectMake(20, 100, 250, 40)];
    [self.view addSubview:bar];
```

@property(nonatomic)        UIBarStyle              barStyle; 

这个属性可以设置searchBar的搜索框的风格，枚举如下：

```
typedef NS_ENUM(NSInteger, UIBarStyle) {
    UIBarStyleDefault          = 0,//默认风格 白色搜索框，多出的背景为灰色
    UIBarStyleBlack            = 1,//黑色风格，黑色的搜索框
    //下面两个枚举已经被禁用，作用和黑色风格一样
    UIBarStyleBlackOpaque      = 1, // Deprecated. Use UIBarStyleBlack
    UIBarStyleBlackTranslucent = 2, // Deprecated. Use UIBarStyleBlack and set the translucent property to YES
};
```

@property(nonatomic,copy)   NSString               *text; 

设置搜索框中的文字

@property(nonatomic,copy)   NSString               *prompt; 

这个属性的官方解释是在搜索框顶部显示一行文字，其实就是背景文字，上图说明：

```
   bar.prompt = @"搜索框";
   bar.text=@"321111111111111111111111111"
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/0922/171722_IiCf_2340880.png)

@property(nonatomic,copy)   NSString               *placeholder;  

和其他文本输入控件的placeholder相同，在输入文字时就会消失

@property(nonatomic)        BOOL                    showsBookmarkButton; 

是否在搜索框右侧显示一个图书的按钮，默认为NO，YES的效果如下：

![](http://static.oschina.net/uploads/space/2015/0922/172040_6Imi_2340880.png)

@property(nonatomic)        BOOL                    showsCancelButton;

是否显示取消按钮，默认为NO，YES的效果如下：

![](http://static.oschina.net/uploads/space/2015/0922/172206_lasH_2340880.png)

@property(nonatomic)        BOOL                    showsSearchResultsButton;

是否显示搜索结果按钮，默认为NO，YES效果如下：

![](http://static.oschina.net/uploads/space/2015/0922/172743_efn5_2340880.png)

@property(nonatomic, getter=isSearchResultsButtonSelected) BOOL searchResultsButtonSelected ;

设置搜索结果按钮的选中状态

\- (void)setShowsCancelButton:(BOOL)showsCancelButton animated:(BOOL)animated;

设置显示取消按钮

@property(nonatomic,retain) UIColor *tintColor;

设置这个颜色值会影响搜索框中的光标的颜色

@property(nonatomic,retain) UIColor *barTintColor;

设置这个颜色会影响搜索框的背景颜色

@property (nonatomic) UISearchBarStyle searchBarStyle;

设置搜索框整体的风格，枚举如下：

```
typedef NS_ENUM(NSUInteger, UISearchBarStyle) {
    UISearchBarStyleDefault,    // currently UISearchBarStyleProminent
    UISearchBarStyleProminent,  // 显示背景
    UISearchBarStyleMinimal     // 不显示背景
} NS_ENUM_AVAILABLE_IOS(7_0);
```

@property(nonatomic,assign,getter=isTranslucent) BOOL translucent;

设置是否半透明

@property(nonatomic)      BOOL       showsScopeBar ;

是否显示搜索栏的附件选择按钮试图，要想显示这个试图，首先要将这个属性设置为YES，之后给按钮数组中添加按钮，使用下面这个属性：

@property(nonatomic,copy) NSArray   *scopeButtonTitles ；

设置选择按钮试图的按钮标题

@property(nonatomic)      NSInteger  selectedScopeButtonIndex;

设置一个默认的选中按钮

```
    bar = [[UISearchBar alloc]initWithFrame:CGRectMake(20, 100, 250, 200)];
    bar.showsScopeBar=YES;
    bar.scopeButtonTitles = @[@"12",@"2",@"3",@"4"];
```

![](http://static.oschina.net/uploads/space/2015/0922/183544_lq37_2340880.png)

@property (nonatomic, readwrite, retain) UIView *inputAccessoryView;

键盘的附属试图

@property(nonatomic,retain) UIImage *backgroundImage;

设置搜索框的背景图案

@property(nonatomic,retain) UIImage *scopeBarBackgroundImage;

设置附属选择按钮视图的背景图案

\- (void)setBackgroundImage:(UIImage *)backgroundImage forBarPosition:(UIBarPosition)barPosition barMetrics:(UIBarMetrics)barMetrics ;  

\- (UIImage *)backgroundImageForBarPosition:(UIBarPosition)barPosition barMetrics:(UIBarMetrics)barMetrics

这一对方法可以设置和获取某个状态枚举下的搜索框的背景图案

\- (void)setSearchFieldBackgroundImage:(UIImage *)backgroundImage forState:(UIControlState)state;

\- (UIImage *)searchFieldBackgroundImageForState:(UIControlState)state;

这一对方法用于设置和获取搜索框中TextField的背景图案

\- (void)setImage:(UIImage *)iconImage forSearchBarIcon:(UISearchBarIcon)icon state:(UIControlState)state ;

\- (UIImage *)imageForSearchBarIcon:(UISearchBarIcon)icon state:(UIControlState)state ;

这一对方法用于获取和设置搜索栏icon图片的图案

\- (void)setScopeBarButtonBackgroundImage:(UIImage *)backgroundImage forState:(UIControlState)state; 

\- (UIImage *)scopeBarButtonBackgroundImageForState:(UIControlState)state;

这一对方法用于设置和获取搜索框的附加选择按钮视图的背景图案

\- (void)setScopeBarButtonDividerImage:(UIImage *)dividerImage forLeftSegmentState:(UIControlState)leftState rightSegmentState:(UIControlState)rightState;

\- (UIImage *)scopeBarButtonDividerImageForLeftSegmentState:(UIControlState)leftState rightSegmentState:(UIControlState)rightState;

这一对方法用于获取和设置附加选择按钮视图中切换按钮的图案

\- (void)setScopeBarButtonTitleTextAttributes:(NSDictionary *)attributes forState:(UIControlState)state;

\- (NSDictionary *)scopeBarButtonTitleTextAttributesForState:(UIControlState)state;

这一对方法用于设置和获取切换按钮标题文字的字体属性字典

@property(nonatomic) UIOffset searchFieldBackgroundPositionAdjustment;

搜索文字在搜索框中的位置偏移

@property(nonatomic) UIOffset searchTextPositionAdjustment;

textfield在搜索框中的位置偏移

\- (void)setPositionAdjustment:(UIOffset)adjustment forSearchBarIcon:(UISearchBarIcon)icon;

\- (UIOffset)positionAdjustmentForSearchBarIcon:(UISearchBarIcon)icon;

设置搜索栏中图片的位置偏移，图片的枚举如下：

```
typedef NS_ENUM(NSInteger, UISearchBarIcon) {
    UISearchBarIconSearch, //搜索图标
    UISearchBarIconClear, // 清除图标
    UISearchBarIconBookmark, // 书本图标
    UISearchBarIconResultsList, // 结果列表图标
};
```

下面是搜索框控件的一些代理方法：

\- (BOOL)searchBarShouldBeginEditing:(UISearchBar *)searchBar;           

将要开始编辑时的回调，返回为NO，则不能编辑

\- (void)searchBarTextDidBeginEditing:(UISearchBar *)searchBar;                  

已经开始编辑时的回调

\- (BOOL)searchBarShouldEndEditing:(UISearchBar *)searchBar;                

将要结束编辑时的回调

\- (void)searchBarTextDidEndEditing:(UISearchBar *)searchBar;                   

已经结束编辑的回调

\- (void)searchBar:(UISearchBar *)searchBar textDidChange:(NSString *)searchText;   编辑文字改变的回调

\- (BOOL)searchBar:(UISearchBar *)searchBar shouldChangeTextInRange:(NSRange)range replacementText:(NSString *)text ; 

编辑文字改变前的回调，返回NO则不能加入新的编辑文字

\- (void)searchBarSearchButtonClicked:(UISearchBar *)searchBar;       

搜索按钮点击的回调

\- (void)searchBarBookmarkButtonClicked:(UISearchBar *)searchBar;             

书本按钮点击的回调

\- (void)searchBarCancelButtonClicked:(UISearchBar *)searchBar;               

取消按钮点击的回调

\- (void)searchBarResultsListButtonClicked:(UISearchBar *)searchBar; 

搜索结果按钮点击的回调

\- (void)searchBar:(UISearchBar *)searchBar selectedScopeButtonIndexDidChange:(NSInteger)selectedScope;

搜索栏的附加试图中切换按钮触发的回调

  
 

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

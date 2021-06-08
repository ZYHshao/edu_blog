---
title: iOS UIButton解读
date: 2015-04-11  
categories: iOS之UI控件
tags: [UIButton,iOS编程]              
---
UIButton控件是应用界面中常用的一个控件，用法总结：

一、初始化

UIButton的初始化一般使用其类方法，+ (id)buttonWithType:(UIButtonType)buttonType;

风格的枚举如下：

typedef NS_ENUM(NSInteger, UIButtonType) {
    //用户自定义，无风格
    UIButtonTypeCustom = 0,  
    //系统默认风格                       
    UIButtonTypeSystem NS_ENUM_AVAILABLE_IOS(7_0), 
    //一下这三种创建出来的按钮一样，一个蓝色的圆圈，中间有个叹号
    UIButtonTypeDetailDisclosure,
    UIButtonTypeInfoLight,
    UIButtonTypeInfoDark,
    //创建+号按钮
    UIButtonTypeContactAdd,
    //废弃
    UIButtonTypeRoundedRect = UIButtonTypeSystem,
};

二、属性设置

@property(nonatomic) UIEdgeInsets contentEdgeInsets UI_APPEARANCE_SELECTOR;
//这个属性设置button里内容的偏移量，包括title和image，可以用如下方法设置
btn.contentEdgeInsets=UIEdgeInsetsMake(20, 20, 0, 0);

@property(nonatomic) UIEdgeInsets titleEdgeInsets;
//这个属性设置标题的偏移量         
@property(nonatomic) BOOL reversesTitleShadowWhenHighlighted;
//按钮高亮时，是否改变阴影效果
@property(nonatomic) UIEdgeInsets imageEdgeInsets;
//图片的偏移量              
@property(nonatomic)BOOL  adjustsImageWhenHighlighted;
//设置图片的绘制是否高亮时变暗   
@property(nonatomic)BOOL  adjustsImageWhenDisabled;
//设置图片是否轻绘制当按钮禁用时
@property(nonatomic)BOOL showsTouchWhenHighlighted;
//设置是否显示手指印在按钮高亮的时候
@property(nonatomic,retain)   UIColor     *tintColor NS_AVAILABLE_IOS(5_0); 
//这个属性会作用于标题和图片，但是如果你是自定义风格的按钮，这个属性将不起任何作用，它只作用于系统的
@property(nonatomic,readonly) UIButtonType buttonType;
//设置button的风格

三、一些set方法

- (void)setTitle:(NSString *)title forState:(UIControlState)state;
//设置标题和显示当前标题的按钮状态 
- (void)setTitleColor:(UIColor *)color forState:(UIControlState)state;
//设置标题颜色和显示当前颜色的按钮状态 
- (void)setTitleShadowColor:(UIColor *)color forState:(UIControlState)state; 
//设置标题阴影颜色及显示时的状态
- (void)setImage:(UIImage *)image forState:(UIControlState)state; 
//设置按钮图片和显示当前图片时的状态
- (void)setBackgroundImage:(UIImage *)image forState:(UIControlState)state;
//设置按钮背景图片和显示图片时的状态
- (void)setAttributedTitle:(NSAttributedString *)title forState:(UIControlState)state NS_AVAILABLE_IOS(6_0);
//通过AttributeString创建标题

注意：按钮图片设置和背景图片的不同在于：

        1、设置图片，如果有标题会和标题并列显示

        2、设置背景图片会出现在标题下面

        3、图片的偏移量可以设置，背景图片不可以。

四、一些get方法，可以得到上述设置的属性

- (NSString *)titleForState:(UIControlState)state;       
- (UIColor *)titleColorForState:(UIControlState)state;
- (UIColor *)titleShadowColorForState:(UIControlState)state;
- (UIImage *)imageForState:(UIControlState)state;
- (UIImage *)backgroundImageForState:(UIControlState)state;
- (NSAttributedString *)attributedTitleForState:(UIControlState)state NS_AVAILABLE_IOS(6_0);

五、一些只读属性

@property(nonatomic,readonly,retain) NSString *currentTitle;        
@property(nonatomic,readonly,retain) UIColor  *currentTitleColor;    
@property(nonatomic,readonly,retain) UIColor  *currentTitleShadowColor; 
@property(nonatomic,readonly,retain) UIImage  *currentImage; 
@property(nonatomic,readonly,retain) UIImage  *currentBackgroundImage; 
@property(nonatomic,readonly,retain) NSAttributedString *currentAttributedTitle NS_AVAILABLE_IOS(6_0); 
//这两个参数需要注意，虽然他们是只读属性不能重新设置，但是我们可以设置label和imageView的相关属性
@property(nonatomic,readonly,retain) UILabel     *titleLabel NS_AVAILABLE_IOS(3_0);
@property(nonatomic,readonly,retain) UIImageView *imageView  NS_AVAILABLE_IOS(3_0);

六、下面这些函数，都会返回一个CGRect 矩形范围

- (CGRect)backgroundRectForBounds:(CGRect)bounds;
//返回背景大小
- (CGRect)contentRectForBounds:(CGRect)bounds;
//返回视图大小，包括标题和图片
- (CGRect)titleRectForContentRect:(CGRect)contentRect;
//返回标题大小
- (CGRect)imageRectForContentRect:(CGRect)contentRect;
//返回图片大小

关于触发事件，button是继承于UIControl,这里不再叙述。

专注技术，热爱生活，交流技术，也做朋友。

——珲少 QQ群：203317592

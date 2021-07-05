---
title: iOS选择器视图控件(UIPickerView)使用方法总结
date: 2015-06-10
categories: iOS之UI控件
tags: []
---
## iOS中UIPickerView使用总结

UIPickerView是iOS中的原生选择器控件，使用方便，用法简单，效果漂亮。

[@property](http://my.oschina.net/property)(nonatomic,assign) id<UIPickerViewDataSource\> dataSource;                

[@property](http://my.oschina.net/property)(nonatomic,assign) id<UIPickerViewDelegate\>   delegate; 

设置数据源和代理

[@property](http://my.oschina.net/property)(nonatomic) BOOL showsSelectionIndicator;

是否显示选择框，在iOS7之后这个属性没有任何效果

[@property](http://my.oschina.net/property)(nonatomic,readonly) NSInteger numberOfComponents;

获取分区数

\- (NSInteger)numberOfRowsInComponent:(NSInteger)component;

获取某一分区的行数

\- (CGSize)rowSizeForComponent:(NSInteger)component;

获取某一分区行的尺寸

\- (UIView *)viewForRow:(NSInteger)row forComponent:(NSInteger)component;

获取某一分区某一行的视图

\- (void)reloadAllComponents;

重载所有分区

\- (void)reloadComponent:(NSInteger)component;

重载某一分区

\- (void)selectRow:(NSInteger)row inComponent:(NSInteger)component animated:(BOOL)animated; 

设置选中某一分区某一行

\- (NSInteger)selectedRowInComponent:(NSInteger)component;  

返回某一分区选中的行

数据源代理中的方法：

\- (NSInteger)numberOfComponentsInPickerView:(UIPickerView *)pickerView;

设置分区数

\- (NSInteger)pickerView:(UIPickerView *)pickerView numberOfRowsInComponent:(NSInteger)component;

根据分区设置行数

代理中的方法：

\- (CGFloat)pickerView:(UIPickerView *)pickerView widthForComponent:(NSInteger)component;

设置分区宽度

\- (CGFloat)pickerView:(UIPickerView *)pickerView rowHeightForComponent:(NSInteger)component;

设置分区行高

\- (NSString *)pickerView:(UIPickerView *)pickerView titleForRow:(NSInteger)row forComponent:(NSInteger)component;

设置某一行显示的标题

\- (NSAttributedString *)pickerView:(UIPickerView *)pickerView attributedTitleForRow:(NSInteger)row forComponent:(NSInteger)component;

通过属性字符串设置某一行显示的标题

\- (UIView *)pickerView:(UIPickerView *)pickerView viewForRow:(NSInteger)row forComponent:(NSInteger)component reusingView:(UIView *)view;

设置某一行显示的view视图

\- (void)pickerView:(UIPickerView *)pickerView didSelectRow:(NSInteger)row inComponent:(NSInteger)component;

选中某一行时执行的回调

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

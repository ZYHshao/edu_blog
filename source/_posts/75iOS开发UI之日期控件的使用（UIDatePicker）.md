---
title: iOS开发UI之日期控件的使用（UIDatePicker）
date: 2015-06-09
categories: iOS之UI控件
tags: []
---
## iOS日期控件UIDatePicker用法总结

[@property](http://my.oschina.net/property) (nonatomic) UIDatePickerMode datePickerMode; 

设置控件模式，枚举如下：

```
typedef NS_ENUM(NSInteger, UIDatePickerMode) {
    UIDatePickerModeTime,           //时间模式，显示时分和上下午
    UIDatePickerModeDate,           //日期模式显示年月日
    UIDatePickerModeDateAndTime,    //时间和日期模式，显示月日星期，时分上下午
    UIDatePickerModeCountDownTimer, //计时模式，显示时和分
};
```

[@property](http://my.oschina.net/property) (nonatomic, retain) NSLocale   *locale;

设置本地化环境

[@property](http://my.oschina.net/property) (nonatomic, copy)   NSCalendar *calendar;

设置日历

[@property](http://my.oschina.net/property) (nonatomic, retain) NSTimeZone *timeZone;

设置时区

[@property](http://my.oschina.net/property) (nonatomic, retain) NSDate *date; 

设置当前时间

@property (nonatomic, retain) NSDate *minimumDate;

设置最小时间点

@property (nonatomic, retain) NSDate *maximumDate;

设置最大时间点

@property (nonatomic) NSTimeInterval countDownDuration;

只适用于计时模式，设置时间

@property (nonatomic) NSInteger      minuteInterval;  
设置每一格的时间差

\- (void)setDate:(NSDate *)date animated:(BOOL)animated;

设置到一个时间，有动画效果

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

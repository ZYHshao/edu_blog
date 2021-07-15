---
title: iOS开发之EventKitUI框架的应用
date: 2019-06-26
categories: iOS逻辑初窥
tags: []
---
## iOS开发之EventKitUI框架的应用

      前面博客，有介绍EventKit这个框架的使用，使用EventKit可以与系统的日历和提醒应用进行交互，读写用户的日程事件。EventKitUI，顾名思义，其实基于EventKit框架，提供了一套系统的事件管理界面。EventKit的基础内容介绍如下：

[https://my.oschina.net/u/2340880/blog/3066175](https://my.oschina.net/u/2340880/blog/3066175)

### 一、EKCalendarChooser日历选择页面

      EKCalendarChooser提供了选择日历，即选择EKCalendar对象的视图控制器，示例如下：

```objectivec
EKCalendarChooser *chooser = [[EKCalendarChooser alloc] initWithSelectionStyle:EKCalendarChooserSelectionStyleSingle displayStyle:EKCalendarChooserDisplayAllCalendars eventStore:self.eventStore];
chooser.showsDoneButton = YES;
chooser.showsCancelButton = YES;
chooser.delegate = self;
[self.navigationController pushViewController:chooser animated:YES];
```

需要注意，在实例化EKCalendarChooser的时候，需要关联一个EKEventStore对象，用来进行数据操作。

EKCalendarChooser中属性方法如下：

```objectivec
// 实例化方法
/*
typedef NS_ENUM(NSInteger, EKCalendarChooserSelectionStyle) {
    EKCalendarChooserSelectionStyleSingle,    // 单选模式
    EKCalendarChooserSelectionStyleMultiple   // 多选模式
};

typedef NS_ENUM(NSInteger, EKCalendarChooserDisplayStyle) {
    EKCalendarChooserDisplayAllCalendars,         // 展示全部日历
    EKCalendarChooserDisplayWritableCalendarsOnly // 只展示可写的日历
};
*/
- (id)initWithSelectionStyle:(EKCalendarChooserSelectionStyle)selectionStyle
                displayStyle:(EKCalendarChooserDisplayStyle)displayStyle
                  eventStore:(EKEventStore *)eventStore;
// 实例化方法 entityType参数决定是 系统的日历 还是 提醒 对应的 EKCalander
- (id)initWithSelectionStyle:(EKCalendarChooserSelectionStyle)style 
                displayStyle:(EKCalendarChooserDisplayStyle)displayStyle
                  entityType:(EKEntityType)entityType
                  eventStore:(EKEventStore *)eventStore;
// 获取用户选中的日历 集合
@property(nonatomic, copy) NSSet<EKCalendar *> *selectedCalendars;
// 选择的风格
@property(nonatomic, readonly) EKCalendarChooserSelectionStyle    selectionStyle;
// 代理对象
@property(nonatomic, weak, nullable) id<EKCalendarChooserDelegate>        delegate;
// 是否展示完成按钮 在导航上
@property(nonatomic) BOOL showsDoneButton;
// 是否展示取消按钮在导航上
@property(nonatomic) BOOL showsCancelButton;
```

EKCalendarChooserDelegate代理中定义的方法如下：

```objectivec
@protocol EKCalendarChooserDelegate <NSObject>
@optional
// 用户选择改变后触发的回调
- (void)calendarChooserSelectionDidChange:(EKCalendarChooser *)calendarChooser;
// 用户选择完成后触发的回调
- (void)calendarChooserDidFinish:(EKCalendarChooser *)calendarChooser;
// 用户取消选择后触发的回调
- (void)calendarChooserDidCancel:(EKCalendarChooser *)calendarChooser;
@end
```

### 二、EKEventViewController事件详情页面

      EKEventViewController提供了展示某个事件详情的试图控制器，示例如下：

```objectivec
- (void)queryEvent {
    for (EKCalendar *cal in [self.eventStore calendarsForEntityType:EKEntityTypeEvent]) {
        if ([cal.title isEqualToString:@"珲少的事项日历"]) {
            NSCalendar *calendar = [NSCalendar currentCalendar];
         
            NSDateComponents *oneMonthFromNowComponents = [[NSDateComponents alloc] init];
            oneMonthFromNowComponents.month = 1;
            NSDate *oneMonthFromNow = [calendar dateByAddingComponents:oneMonthFromNowComponents
                                                                toDate:[NSDate date]
                                                               options:0];

            NSPredicate*predicate = [self.eventStore predicateForEventsWithStartDate:[NSDate date] endDate:oneMonthFromNow calendars:@[cal]];
            
            NSArray *eventArray = [self.eventStore eventsMatchingPredicate:predicate];
            // 打开控制器
            EKEventViewController *controller = [[EKEventViewController alloc] init];
            controller.event = eventArray.firstObject;
            [self presentViewController:controller animated:YES completion:nil];
        }
    }
}

```

EKEventViewController也支持进行事件的编辑，其中属性方法如下：

```objectivec
@interface EKEventViewController : UIViewController
// 代理对象
@property(nonatomic, weak, nullable) id<EKEventViewDelegate>  delegate;
// 对应的事件对象，在使用控制器时，必须设置这个属性
@property(nonatomic, retain, null_unspecified) EKEvent *event;
// 设置是否允许编辑
@property(nonatomic) BOOL      allowsEditing;
// 设置是否允许日历预览
@property(nonatomic) BOOL      allowsCalendarPreview;
@end
```

EKEventViewDelegate中只定义了一个方法，如下：

```objectivec
@protocol EKEventViewDelegate <NSObject>
@required
// 完成某个行为后会调用的代理回调
/*
typedef NS_ENUM(NSInteger, EKEventViewAction) {
    EKEventViewActionDone,                  // 完成了事件
    EKEventViewActionResponded,             // 回复了事件
    EKEventViewActionDeleted,               // 删除了事件
};
*/
- (void)eventViewController:(EKEventViewController *)controller didCompleteWithAction:(EKEventViewAction)action __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_4_2);
@end
```

### 三、EKEventEditViewController事件编辑控制器

      EKEventEditViewController提供了事件编辑的视图控制器，对于可编辑的EKEventViewController视图控制器，当用户点击的编辑按钮后，也会调用EKEventEditViewController视图控制器进行编辑，示例如下：

```objectivec
EKEventEditViewController *controller = [[EKEventEditViewController alloc] init];
controller.event = eventArray.firstObject;
[self presentViewController:controller animated:YES completion:nil];
```

其中属性方法如下：

```objectivec
@interface EKEventEditViewController : UINavigationController
// 代理对象
@property(nonatomic, weak, nullable) id<EKEventEditViewDelegate>  editViewDelegate;
// 编辑行为完成后，进行数据操作的EKEventStore对象
@property(nonatomic, retain, null_unspecified) EKEventStore   *eventStore;
// 要进行编辑的事件对象
@property(nonatomic, retain, nullable) EKEvent *event;
// 取消编辑
- (void)cancelEditing;
@end
```

EKEventEditViewDelegate解析如下：

```objectivec
@protocol EKEventEditViewDelegate <NSObject>
@required
// 完成某个编辑动作后调用
- (void)eventEditViewController:(EKEventEditViewController *)controller didCompleteWithAction:(EKEventEditViewAction)action;
@optional
// 设置新建事件默认对象的日历
- (EKCalendar *)eventEditViewControllerDefaultCalendarForNewEvents:(EKEventEditViewController *)controller;
@end
```

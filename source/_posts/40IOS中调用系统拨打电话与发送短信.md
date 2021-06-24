---
title: iOS中调用系统拨打电话与发送短信
date: 2015-04-29
categories: iOS之UI控件
tags: []
---
## IOS中调用系统拨打电话发送短信

### 一、调用打电话界面

\[\[UIApplication  sharedApplication\] openURL:\[NSURL  URLWithString:\[NSString  stringWithFormat:@"tel://%@",_phoneNumber\]\]\];

### 二、发送短消息界面

调用系统的发送短信的界面，需要引入以下头文件：

#import <MessageUI/MessageUI.h>

系统短信界面的调用很简单，只需下面几句代码：

```
         MFMessageComposeViewController * con = [[MFMessageComposeViewController alloc]init];
            if ([MFMessageComposeViewController canSendText]) {
                con.recipients=@[_phoneNumber];//电话数组
                con.messageComposeDelegate=self;
                [self presentViewController:con animated:YES completion:nil];
```

下面将MessageUI的一些常用方法总结如下：

\+ (BOOL)canSendText

判断是否支持发送文字

\+ (BOOL)canSendSubject;

判断是否支持发送主题信息

\+ (BOOL)canSendAttachments;

判断是否支持发送附件

\+ (BOOL)isSupportedAttachmentUTI:(NSString *)uti;

判断是否支持统一标示附件

\- (void)disableUserAttachments;

禁止发送附件

[@property](http://my.oschina.net/property)(nonatomic,copy) NSArray *recipients;

联系人数组，会显示在发送人列表里

[@property](http://my.oschina.net/property)(nonatomic,copy) NSString *body;

信息主体内容

[@property](http://my.oschina.net/property)(nonatomic,copy) NSString *subject;

信息标题

@property(nonatomic,copy, readonly) NSArray *attachments;

信息附件数组 只读的 里面是字典

\- (BOOL)addAttachmentURL:(NSURL *)attachmentURL withAlternateFilename:(NSString *)alternateFilename;

根据URL路径和添加附件，返回YES表示添加成功

\- (BOOL)addAttachmentData:(NSData *)attachmentData typeIdentifier:(NSString *)uti filename:(NSString *)filename;

根据Data数据添加附件

\- (void)messageComposeViewController:(MFMessageComposeViewController *)controller didFinishWithResult:(MessageComposeResult)result;

MFMessageComposeViewControllerDelegate的代理方法，result会传回来一个结果，枚举如下：

```
enum MessageComposeResult {
    //取消发送
    MessageComposeResultCancelled,
    //发送成功
    MessageComposeResultSent,
    //发送失败
    MessageComposeResultFailed
};
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

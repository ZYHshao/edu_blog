---
title: iOS调用系统通讯录
date: 2015-04-29
categories: iOS逻辑初窥
tags: []
---
## iOS调用系统通讯录

上一篇博客详细介绍了在IOS开发中，我们如何获取通讯录联系人的信息，即对其进行增删改查的操作：[http://my.oschina.net/u/2340880/blog/407347](http://my.oschina.net/u/2340880/blog/407347)。而在一些开发项目中，如果没有特殊需求，并且我们只是需要一些通讯录信息，并不做修改操作，我们完全可以采取另一种更加方便的方式，直接调用系统的通讯录。

首先，导入这个头文件：

```
#import <AddressBookUI/AddressBookUI.h>
```

注意：需要在项目中链接如下两个库：

![](http://static.oschina.net/uploads/space/2015/0429/104433_MXYa_2340880.png)

只需简单的几句代码，就可以弹出系统的通讯录界面：

```
    ABPeoplePickerNavigationController * con = [[ABPeoplePickerNavigationController   alloc]init];
    con.peoplePickerDelegate=self;
    [self presentViewController:con animated:YES completion:nil];
```

点击联系人后执行的方法，我们只需要实现下面的代理方法即可

```
-(void)peoplePickerNavigationController:(ABPeoplePickerNavigationController *)peoplePicker didSelectPerson:(ABRecordRef)person{
   //person参数就是选择的联系人的引用 具体含义和数据获取，在上一篇博客中有详细介绍
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

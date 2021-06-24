---
title: iOS获取通讯录联系人信息
date: 2015-04-28
categories: iOS逻辑初窥
tags: []
---
## iOS获取系统通讯录联系人信息

### 一、权限注册

随着apple对用户隐私的越来越重视，IOS系统的权限设置也更加严格，在获取系统通讯录之前，我们必须获得用户的授权。权限申请代码示例如下：

```
    //这个变量用于记录授权是否成功，即用户是否允许我们访问通讯录
    int __block tip=0;
    //声明一个通讯簿的引用
    ABAddressBookRef addBook =nil;
    //因为在IOS6.0之后和之前的权限申请方式有所差别，这里做个判断
    if ([[UIDevice currentDevice].systemVersion floatValue]>=6.0) {
        //创建通讯簿的引用
        addBook=ABAddressBookCreateWithOptions(NULL, NULL);
        //创建一个出事信号量为0的信号
        dispatch_semaphore_t sema=dispatch_semaphore_create(0);
        //申请访问权限
        ABAddressBookRequestAccessWithCompletion(addBook, ^(bool greanted, CFErrorRef error)        {
            //greanted为YES是表示用户允许，否则为不允许
            if (!greanted) {
                tip=1;
            }
            //发送一次信号
            dispatch_semaphore_signal(sema);
        });
        //等待信号触发
         dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
    }else{
        //IOS6之前
        addBook =ABAddressBookCreate();
    }
    if (tip) {
    //做一个友好的提示
        UIAlertView * alart = [[UIAlertView alloc]initWithTitle:@"温馨提示" message:@"请您设置允许APP访问您的通讯录\nSettings>General>Privacy" delegate:self cancelButtonTitle:@"确定" otherButtonTitles:nil, nil];
        [alart show];
        return;
    }
```

几点注意：1、dispatch\_semaphore\_t三个相关的操作为

   dispatch\_semaphore\_create　　　创建一个信号  
　　                  dispatch\_semaphore\_signal　　　发送一个信号  
　　                  dispatch\_semaphore\_wait               等待信号触发   

 dispatch\_semaphore\_create()创建一个信号，后面可以跟一个参数，表示信号量，当信号量正值时，dispatch\_semaphore\_wait后面的代码会被执行，否则程序将会一直等待在dispatch\_semaphore\_wait。 dispatch\_semaphore\_signal的作用是发送一个信号，会使信号量加1，相对的，dispatch\_semaphore\_wait执行后会使信号量减1.

                  2、因为是否被授权是在ABAddressBookRequestAccessWithCompletion的block回调中获取的，所以我们需要在外面做一个线程等待。

### 二、获取通讯录联系人详细信息

```
    //获取所有联系人的数组
    CFArrayRef allLinkPeople = ABAddressBookCopyArrayOfAllPeople(addBook);
    //获取联系人总数
    CFIndex number = ABAddressBookGetPersonCount(addBook);
    //进行遍历
    for (NSInteger i=0; i<number; i++) {
        //获取联系人对象的引用
        ABRecordRef  people = CFArrayGetValueAtIndex(allLinkPeople, i);
        //获取当前联系人名字
        NSString*firstName=(__bridge NSString *)(ABRecordCopyValue(people, kABPersonFirstNameProperty));
        //获取当前联系人姓氏
        NSString*lastName=(__bridge NSString *)(ABRecordCopyValue(people, kABPersonLastNameProperty));
        //获取当前联系人中间名
        NSString*middleName=(__bridge NSString*)(ABRecordCopyValue(people, kABPersonMiddleNameProperty));
        //获取当前联系人的名字前缀
        NSString*prefix=(__bridge NSString*)(ABRecordCopyValue(people, kABPersonPrefixProperty));
        //获取当前联系人的名字后缀
        NSString*suffix=(__bridge NSString*)(ABRecordCopyValue(people, kABPersonSuffixProperty));
        //获取当前联系人的昵称
        NSString*nickName=(__bridge NSString*)(ABRecordCopyValue(people, kABPersonNicknameProperty));
        //获取当前联系人的名字拼音
        NSString*firstNamePhoneic=(__bridge NSString*)(ABRecordCopyValue(people, kABPersonFirstNamePhoneticProperty));
        //获取当前联系人的姓氏拼音
        NSString*lastNamePhoneic=(__bridge NSString*)(ABRecordCopyValue(people, kABPersonLastNamePhoneticProperty));
        //获取当前联系人的中间名拼音
        NSString*middleNamePhoneic=(__bridge NSString*)(ABRecordCopyValue(people, kABPersonMiddleNamePhoneticProperty));
        //获取当前联系人的公司
        NSString*organization=(__bridge NSString*)(ABRecordCopyValue(people, kABPersonOrganizationProperty));
        //获取当前联系人的职位
        NSString*job=(__bridge NSString*)(ABRecordCopyValue(people, kABPersonJobTitleProperty));
        //获取当前联系人的部门
        NSString*department=(__bridge NSString*)(ABRecordCopyValue(people, kABPersonDepartmentProperty));
        //获取当前联系人的生日
        NSString*birthday=(__bridge NSDate*)(ABRecordCopyValue(people, kABPersonBirthdayProperty));
        NSMutableArray * emailArr = [[NSMutableArray alloc]init];
        //获取当前联系人的邮箱 注意是数组
        ABMultiValueRef emails= ABRecordCopyValue(people, kABPersonEmailProperty);
        for (NSInteger j=0; j<ABMultiValueGetCount(emails); j++) {
            [emailArr addObject:(__bridge NSString *)(ABMultiValueCopyValueAtIndex(emails, j))];
        }
        //获取当前联系人的备注
       NSString*notes=(__bridge NSString*)(ABRecordCopyValue(people, kABPersonNoteProperty));
       //获取当前联系人的电话 数组
        NSMutableArray * phoneArr = [[NSMutableArray alloc]init];
        ABMultiValueRef phones= ABRecordCopyValue(people, kABPersonPhoneProperty);
        for (NSInteger j=0; j<ABMultiValueGetCount(phones); j++) {
            [phonerr addObject:(__bridge NSString *)(ABMultiValueCopyValueAtIndex(phones, j))];
        }
        //获取创建当前联系人的时间 注意是NSDate
        NSDate*creatTime=(__bridge NSDate*)(ABRecordCopyValue(people, kABPersonCreationDateProperty));
        //获取最近修改当前联系人的时间
        NSDate*alterTime=(__bridge NSDate*)(ABRecordCopyValue(people, kABPersonModificationDateProperty));
        //获取地址
        ABMultiValueRef address = ABRecordCopyValue(people, kABPersonAddressProperty);
        for (int j=0; j<ABMultiValueGetCount(address); j++) {
            //地址类型
            NSString * type = (__bridge NSString *)(ABMultiValueCopyLabelAtIndex(address, j));
            NSDictionary * temDic = (__bridge NSDictionary *)(ABMultiValueCopyValueAtIndex(address, j));
            //地址字符串，可以按需求格式化
            NSString * adress = [NSString stringWithFormat:@"国家:%@\n省:%@\n市:%@\n街道:%@\n邮编:%@",[temDic valueForKey:(NSString*)kABPersonAddressCountryKey],[temDic valueForKey:(NSString*)kABPersonAddressStateKey],[temDic valueForKey:(NSString*)kABPersonAddressCityKey],[temDic valueForKey:(NSString*)kABPersonAddressStreetKey],[temDic valueForKey:(NSString*)kABPersonAddressZIPKey]];
        }
        //获取当前联系人头像图片
        NSData*userImage=(__bridge NSData*)(ABPersonCopyImageData(people));
        //获取当前联系人纪念日
        NSMutableArray * dateArr = [[NSMutableArray alloc]init];
        ABMultiValueRef dates= ABRecordCopyValue(people, kABPersonDateProperty);
        for (NSInteger j=0; j<ABMultiValueGetCount(dates); j++) {
            //获取纪念日日期
            NSDate * data =(__bridge NSDate*)(ABMultiValueCopyValueAtIndex(dates, j));
            //获取纪念日名称
            NSString * str =(__bridge NSString*)(ABMultiValueCopyLabelAtIndex(dates, j));
            NSDictionary * temDic = [NSDictionary dictionaryWithObject:data forKey:str];
            [dateArr addObject:temDic];
        }
```

一点扩展：相同的方法，可以获取关联人信息，社交信息，邮箱信息，各种类型的电话信息，字段如下：

```
 //相关人，组织字段
const ABPropertyID kABPersonKindProperty; 
const CFNumberRef kABPersonKindPerson;
const CFNumberRef kABPersonKindOrganization;

// 电话相关字段
AB_EXTERN const ABPropertyID kABPersonPhoneProperty;
AB_EXTERN const CFStringRef kABPersonPhoneMobileLabel;
AB_EXTERN const CFStringRef kABPersonPhoneIPhoneLabel 
AB_EXTERN const CFStringRef kABPersonPhoneMainLabel;
AB_EXTERN const CFStringRef kABPersonPhoneHomeFAXLabel;
AB_EXTERN const CFStringRef kABPersonPhoneWorkFAXLabel;
AB_EXTERN const CFStringRef kABPersonPhoneOtherFAXLabel
AB_EXTERN const CFStringRef kABPersonPhonePagerLabel;

// 即时聊天信息相关字段
AB_EXTERN const ABPropertyID kABPersonInstantMessageProperty;    
AB_EXTERN const CFStringRef kABPersonInstantMessageServiceKey;    
AB_EXTERN const CFStringRef kABPersonInstantMessageServiceYahoo;
AB_EXTERN const CFStringRef kABPersonInstantMessageServiceJabber;
AB_EXTERN const CFStringRef kABPersonInstantMessageServiceMSN;
AB_EXTERN const CFStringRef kABPersonInstantMessageServiceICQ;
AB_EXTERN const CFStringRef kABPersonInstantMessageServiceAIM;
AB_EXTERN const CFStringRef kABPersonInstantMessageServiceQQ 
AB_EXTERN const CFStringRef kABPersonInstantMessageServiceGoogleTalk;
AB_EXTERN const CFStringRef kABPersonInstantMessageServiceSkype;
AB_EXTERN const CFStringRef kABPersonInstantMessageServiceFacebook;
AB_EXTERN const CFStringRef kABPersonInstantMessageServiceGaduGadu;
AB_EXTERN const CFStringRef kABPersonInstantMessageUsernameKey; 

// 个人网页相关字段
AB_EXTERN const ABPropertyID kABPersonURLProperty;
AB_EXTERN const CFStringRef kABPersonHomePageLabel; 
//相关人姓名字段
AB_EXTERN const ABPropertyID kABPersonRelatedNamesProperty;   
AB_EXTERN const CFStringRef kABPersonFatherLabel;    // Father
AB_EXTERN const CFStringRef kABPersonMotherLabel;    // Mother
AB_EXTERN const CFStringRef kABPersonParentLabel;    // Parent
AB_EXTERN const CFStringRef kABPersonBrotherLabel;   // Brother
AB_EXTERN const CFStringRef kABPersonSisterLabel;    // Sister
AB_EXTERN const CFStringRef kABPersonChildLabel;      // Child
AB_EXTERN const CFStringRef kABPersonFriendLabel;    // Friend
AB_EXTERN const CFStringRef kABPersonSpouseLabel;    // Spouse
AB_EXTERN const CFStringRef kABPersonPartnerLabel;   // Partner
AB_EXTERN const CFStringRef kABPersonAssistantLabel; // Assistant
AB_EXTERN const CFStringRef kABPersonManagerLabel;   // Manager
```

### 三、通讯录“写”的相关操作

看到上面读取信息的代码，你可能觉得一阵目炫，其实只是字段比较长，逻辑还是很简单的，同样，写的操作与之类似，创建，修改，删除，是我们对通讯录“写”的常用操作。

#### 1、创建一个联系人

```
    //创建一个联系人引用
    ABRecordRef person = ABPersonCreate();
    NSString *firstName = @"哈";
    NSString *lastName = @"哈";
    // 电话号码数组
    NSArray *phones = [NSArray arrayWithObjects:@"123",@"456",nil];
    // 电话号码对应的名称
    NSArray *labels = [NSArray arrayWithObjects:@"iphone",@"home",nil];
    //这里的字段和上面的字段完全相同
    // 设置名字属性
    ABRecordSetValue(person, kABPersonFirstNameProperty,(__bridge CFStringRef)firstName, NULL);
    // 设置姓氏属性
    ABRecordSetValue(person, kABPersonLastNameProperty, (__bridge CFStringRef)lastName, NULL);
    // 设置生日属性
    ABRecordSetValue(person, kABPersonBirthdayProperty,(__bridge CFDateRef)birthday, NULL);
    // 字典引用
    ABMultiValueRef dic =ABMultiValueCreateMutable(kABMultiStringPropertyType);
    // 添加电话号码与其对应的名称内容
    for (int i = 0; i < [phones count]; i ++) {
        ABMultiValueIdentifier obj = ABMultiValueAddValueAndLabel(dic,(__bridge CFStringRef)[phones objectAtIndex:i], (__bridge CFStringRef)[labels objectAtIndex:i], &obj);
    }
    // 设置phone属性
    ABRecordSetValue(person, kABPersonPhoneProperty, dic, NULL);
    // 将新建的联系人添加到通讯录中
    ABAddressBookAddRecord(addBook, person, NULL);
    // 保存通讯录数据
    ABAddressBookSave(addBook, NULL);
```

#### 2、修改联系人

修改联系人的操作就是将获取和添加和在一起，先获取到相应的联系人引用，重设其属性字段即可。

#### 3.删除联系人

```
     //获取所有联系人
     NSArray *array = (__bridge NSArray*)ABAddressBookCopyArrayOfAllPeople(addBook);
    // 遍历所有的联系人
    for (id obj in array) {
        ABRecordRef people = (__bridge ABRecordRef)obj;
        NSString *firstName = (__bridge NSString*)ABRecordCopyValue(people, kABPersonFirstNameProperty);
        NSString *lastName = (__bridge NSString*)ABRecordCopyValue(people, kABPersonLastNameProperty);
        if ([firstName isEqualToString:@"哈"] &&[lastName isEqualToString:@"哈"]) {
            ABAddressBookRemoveRecord(addBook, people,NULL);
        }
    }
    // 保存修改的通讯录对象
    ABAddressBookSave(addBook, NULL);
```

### 四、重中之重-关于内存管理

上面的代码为了演示方便，创建的全部引用都没有释放，势必是造成内存泄露，在我们用ABAddressBookCreate()创建一个引用对象时，切记无论ARC还MRC，要用CFRelease()进行释放引用，例如上面的例子，我们需要加上这句代码

CFRelease(addBook);

如果你耐心的看到了这里，我想你一定明白了我为什么不在前边的代码里说明这个问题，因为在ARC项目普及的现在，这的确是重中之重。

疏漏之处 欢迎指正

学习使用 欢迎转载

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

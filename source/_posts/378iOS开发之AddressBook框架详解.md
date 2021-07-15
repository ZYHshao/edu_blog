---
title: iOS开发之AddressBook框架详解
date: 2018-08-19
categories: iOS逻辑初窥
tags: []
---
## iOS开发之AddressBook框架详解

### 一、写在前面

    首先，AddressBook框架是一个已经过时的框架，iOS9之后官方提供了Contacts框架来进行用户通讯录相关操作。尽管如此，AddressBook框架依然是一个非常优雅并且使用方便的通讯录帮助库。本篇博客只要总结AddressBook框架的相关使用方法。

    在AddressBook框架中，两个最重要的数据模型为ABAddressbookRef与ABRecordRef。前者我们可以理解为通讯录的抽象对象，用它来具体操作通讯录的行为，后者可以理解为通讯录中记录的抽象对象，其中封装了联系人的相关信息。如下图所示：

![](https://oscimg.oschina.net/oscnet/c034a0f13bd8724a8dec6b42cc3e80abc5f.jpg)

### 二、关于用户权限申请

    在应用程序内，若需要使用用户的通讯录权限需要征得用户的同意(毕竟通讯录属于用户隐私)。因此，在使用之前，开发者首先需要进行权限的申请，首先，需要在info.plist文件中添加如下键：

Privacy - Contacts Usage Description

使用如下代码进行使用权限的申请：

```objectivec
//获取用户授权状态
ABAuthorizationStatus status = ABAddressBookGetAuthorizationStatus();
//如果尚未获取过授权 进行授权申请
if (status==kABAuthorizationStatusNotDetermined) {
    //创建通讯录对象 这个方法中第1个参数为预留参数 传NULL 即可 第2个参数可以传一个CFErrorRef的指针
    ABAddressBookRef addressBookRef = ABAddressBookCreateWithOptions(NULL, NULL);
    //请求授权
    ABAddressBookRequestAccessWithCompletion(addressBookRef, ^(bool granted, CFErrorRef error) {
        if (granted) {//请求授权页面用户同意授权
            //可以进行使用
        }
        //释放内存
        CFRelease(addressBookRef);
    });
}
```

ABAuthorizationStatus是授权状态的枚举，意义如下：

```objectivec
typedef CF_ENUM(CFIndex, ABAuthorizationStatus) {
    kABAuthorizationStatusNotDetermined = 0,    // 尚未申请过授权
    kABAuthorizationStatusRestricted,           // 授权被限制 无法使用
    kABAuthorizationStatusDenied,               // 用户拒绝授权
    kABAuthorizationStatusAuthorized            // 已经授权
}
```

### 三、获取基础的通讯录信息

    下面代码演示了如何获取基础的通讯录联系人信息：

```objectivec
        //获取通讯录
        ABAddressBookRef addressBook = ABAddressBookCreateWithOptions(NULL, NULL);
        //获取联系人数量
        CFIndex personCount = ABAddressBookGetPersonCount(addressBook);
        //拿到所有联系人
        CFArrayRef peopleArray = ABAddressBookCopyArrayOfAllPeople(addressBook);
        for (int i = 0; i < personCount; i++) {
            //获取记录
            ABRecordRef person = CFArrayGetValueAtIndex(peopleArray, i);
            //拿到姓名
            //姓 需要转换成NSString类型
            NSString *lastNameValue = (__bridge_transfer NSString *)ABRecordCopyValue(person, kABPersonLastNameProperty);
            //名
            NSString *firstNameValue = (__bridge_transfer NSString *)ABRecordCopyValue(person, kABPersonFirstNameProperty);
            NSLog(@"%@:%@",lastNameValue,firstNameValue);
            //拿到电话 电话可能有多个
            ABMultiValueRef phones = ABRecordCopyValue(person, kABPersonPhoneProperty);
            //解析电话数据
            CFIndex phoneCount = ABMultiValueGetCount(phones);
            for (int j = 0; j < phoneCount ; j++) {
                //电话标签本地化(例如是住宅,工作等)
                NSString *phoneLabel = (__bridge_transfer NSString *)ABAddressBookCopyLocalizedLabel(ABMultiValueCopyLabelAtIndex(phones, j));
                //拿到标签下对应的电话号码
                NSString *phoneValue = (__bridge_transfer NSString *)ABMultiValueCopyValueAtIndex(phones, j);
                NSLog(@"%@:%@",phoneLabel,phoneValue);
            }
            CFRelease(phones);
            
            //邮箱 可能多个
            ABMultiValueRef emails = ABRecordCopyValue(person, kABPersonEmailProperty);
            CFIndex emailCount = ABMultiValueGetCount(emails);
            for (int k = 0; k < emailCount; k++) {
                NSString *emailLabel = (__bridge_transfer NSString *)ABAddressBookCopyLocalizedLabel(ABMultiValueCopyLabelAtIndex(emails, k));
                NSString *emailValue = (__bridge_transfer NSString *)ABMultiValueCopyValueAtIndex(emails, k);
                NSLog(@"%@:%@",emailLabel,emailValue);
            }
            NSLog(@"==============");
            CFRelease(emails);
        }
        CFRelease(addressBook);
        CFRelease(peopleArray);
```

打印信息如下：

![](https://oscimg.oschina.net/oscnet/071948438daf8eaff82d4b774e2900e90de.jpg)

关于可获取到的联系人属性，键值列举如下：

```objectivec
//名
kABPersonFirstNameProperty
//姓
kABPersonLastNameProperty
//中间名
kABPersonMiddleNameProperty
//前缀 用户在存储联系人时 可以添加自定义的前缀 例如 女士 男士等等
kABPersonPrefixProperty
//后缀
kABPersonSuffixProperty
//昵称
kABPersonNicknameProperty
//姓发音说明字段 用户自定义的
kABPersonFirstNamePhoneticProperty
//名发音说明字段 用户自定义的
kABPersonLastNamePhoneticProperty
//中间名发音说明字段 用户自定义的
kABPersonMiddleNamePhoneticProperty
//公司名
kABPersonOrganizationProperty
//部门名
kABPersonDepartmentProperty
//头衔
kABPersonJobTitleProperty
//电子邮件信息 返回一组 需要手动解析
kABPersonEmailProperty
//返回生日信息 日期对象
kABPersonBirthdayProperty
//笔记信息
kABPersonNoteProperty
//记录的创建日期
kABPersonCreationDateProperty
//记录的最后修改日期
kABPersonModificationDateProperty
//地址信息 返回 一组
/*
例如：
ABMultiValueRef address = ABRecordCopyValue(person, kABPersonAddressProperty);
     for (int j=0; j<ABMultiValueGetCount(address); j++) {
         //地址类型
         NSString *type = (__bridge NSString *)(ABMultiValueCopyLabelAtIndex(address, j));
         NSDictionary * temDic = (__bridge NSDictionary *)(ABMultiValueCopyValueAtIndex(address, j));
        //地址字符串，可以按需求格式化
        NSString * adress = [NSString stringWithFormat:@"国家:%@\n省:%@\n市:%@\n街道:%@\n邮编:%@",[temDic valueForKey:(NSString*)kABPersonAddressCountryKey],[temDic valueForKey:(NSString*)kABPersonAddressStateKey],[temDic valueForKey:(NSString*)kABPersonAddressCityKey],[temDic valueForKey:(NSString*)kABPersonAddressStreetKey],[temDic valueForKey:(NSString*)kABPersonAddressZIPKey]];
     }
*/
kABPersonAddressProperty
//地址字典中的街道信息键
kABPersonAddressStreetKey
//地址字典中的城市信息键
kABPersonAddressCityKey
//地址字典中的地区信息键
kABPersonAddressStateKey
//地址字典中的压缩码信息键
kABPersonAddressZIPKey
//地址字典中的国家信息键
kABPersonAddressCountryKey
//地址字典中的国家编码信息键
kABPersonAddressCountryCodeKey
//获取 一组 纪念日日期 
kABPersonDateProperty
//从具体的日期实体中获取纪念日 标签
kABPersonAnniversaryLabel
//获取一组电话号码
kABPersonPhoneProperty
//下面这些对应电话类型
kABPersonPhoneMobileLabel
kABPersonPhoneIPhoneLabel
kABPersonPhoneMainLabel
kABPersonPhoneHomeFAXLabel
kABPersonPhoneWorkFAXLabel
kABPersonPhoneOtherFAXLabel
kABPersonPhonePagerLabel
//获取社交相关信息
kABPersonInstantMessageProperty
//下面这些对应社交平台
kABPersonInstantMessageServiceKey         
kABPersonInstantMessageServiceYahoo
kABPersonInstantMessageServiceJabber
kABPersonInstantMessageServiceMSN
kABPersonInstantMessageServiceICQ
kABPersonInstantMessageServiceAIM
kABPersonInstantMessageServiceQQ 
kABPersonInstantMessageServiceGoogleTalk
kABPersonInstantMessageServiceSkype
kABPersonInstantMessageServiceFacebook
kABPersonInstantMessageServiceGaduGadu
//社交用户名
kABPersonInstantMessageUsernameKey
//获取一组url
kABPersonURLProperty
//url相关标签
kABPersonHomePageLabel
//关联信息
kABPersonRelatedNamesProperty
//关联信息相关的标签
kABPersonFatherLabel                           
kABPersonMotherLabel                           
kABPersonParentLabel
kABPersonBrotherLabel                       
kABPersonSisterLabel                     
kABPersonChildLabel
kABPersonFriendLabel
kABPersonSpouseLabel 
kABPersonPartnerLabel 
kABPersonAssistantLabel
kABPersonManagerLabel
//获取社交账户相关
kABPersonSocialProfileProperty
//社交账户相关key
kABPersonSocialProfileURLKey
kABPersonSocialProfileServiceKey
kABPersonSocialProfileUsernameKey
kABPersonSocialProfileUserIdentifierKey
kABPersonSocialProfileServiceTwitter
kABPersonSocialProfileServiceSinaWeibo
kABPersonSocialProfileServiceGameCenter
kABPersonSocialProfileServiceFacebook
kABPersonSocialProfileServiceMyspace
kABPersonSocialProfileServiceLinkedIn
kABPersonSocialProfileServiceFlickr
//周期性日期信息
kABPersonAlternateBirthdayProperty
//周期性日期相关键
kABPersonAlternateBirthdayCalendarIdentifierKey
kABPersonAlternateBirthdayEraKey
kABPersonAlternateBirthdayYearKey
kABPersonAlternateBirthdayMonthKey
kABPersonAlternateBirthdayIsLeapMonthKey
kABPersonAlternateBirthdayDayKey
```

### 四、关于ABMultiValueRef

    如上所述，我们在获取联系人相关信息时，很多场景都会获取一组，例如对于电话号码，地址，邮箱这类的数据。在AddressBook框架中，这一组数据被封装为ABMultiValueRef对象。对于ABMultiValueRef对象，有如下方法可以对其进行处理：

```objectivec
//获取内部封装值的类型
/*
enum {
    kABInvalidPropertyType         = 0x0,                                           // 无效
    kABStringPropertyType          = 0x1,                                           // 字符串
    kABIntegerPropertyType         = 0x2,                                           // 整数
    kABRealPropertyType            = 0x3,                                           // 属性
    kABDateTimePropertyType        = 0x4,                                           // 时间
    kABDictionaryPropertyType      = 0x5,                                           // 字典
    kABMultiStringPropertyType     = kABMultiValueMask | kABStringPropertyType,     // 聚合字符串
    kABMultiIntegerPropertyType    = kABMultiValueMask | kABIntegerPropertyType,    // 聚合整型
    kABMultiRealPropertyType       = kABMultiValueMask | kABRealPropertyType,       // 聚合属性
    kABMultiDateTimePropertyType   = kABMultiValueMask | kABDateTimePropertyType,   // 聚合时间
    kABMultiDictionaryPropertyType = kABMultiValueMask | kABDictionaryPropertyType, // 聚合字典
};
*/
ABPropertyType ABMultiValueGetPropertyType(ABMultiValueRef multiValue)
//获取其中封装的值的个数
CFIndex ABMultiValueGetCount(ABMultiValueRef multiValue)
//根据索引获取值
ABMultiValueCopyValueAtIndex(ABMultiValueRef multiValue, CFIndex index)
//获取所有的值 组成数组
CFArrayRef ABMultiValueCopyArrayOfAllValues(ABMultiValueRef multiValue)
//根据索引获取标签
CFStringRef ABMultiValueCopyLabelAtIndex(ABMultiValueRef multiValue, CFIndex index)
//根据ID获取值
ABMultiValueIdentifier ABMultiValueGetIdentifierAtIndex(ABMultiValueRef multiValue, CFIndex index)
//根据ID 获取索引
CFIndex ABMultiValueGetIndexForIdentifier(ABMultiValueRef multiValue, ABMultiValueIdentifier identifier)
//获取第一个值
CFIndex ABMultiValueGetFirstIndexOfValue(ABMultiValueRef multiValue, CFTypeRef value)
//下面这些与写联系人信息相关
//创建一个可变的数据对象 
ABMutableMultiValueRef ABMultiValueCreateMutable(ABPropertyType type)
//为某个标签添加值
bool ABMultiValueAddValueAndLabel(ABMutableMultiValueRef multiValue, CFTypeRef value, CFStringRef label, ABMultiValueIdentifier *outIdentifier)
//在某个索引处插入 标签和值
bool ABMultiValueInsertValueAndLabelAtIndex(ABMutableMultiValueRef multiValue, CFTypeRef value, CFStringRef label, CFIndex index, ABMultiValueIdentifier *outIdentifier)
//删除某个索引处的标签和值
bool ABMultiValueRemoveValueAndLabelAtIndex(ABMutableMultiValueRef multiValue, CFIndex index)
//替换某个索引处的值
bool ABMultiValueReplaceValueAtIndex(ABMutableMultiValueRef multiValue, CFTypeRef value, CFIndex index)
//替换某个索引处的标签
bool ABMultiValueReplaceLabelAtIndex(ABMutableMultiValueRef multiValue, CFStringRef label, CFIndex index)
```

### 五、ABRecordRef对象

    ABRecordRef虽然很多时候，我们可以把它理解为联系人对象，但是其实并不正确，实际上它是一个抽象的记录对象，在AddressBook框架中有3种类型的ABRecordRef：

```objectivec
enum {
    kABPersonType = 0, //联系人类型
    kABGroupType  = 1, //组类型
    kABSourceType = 2  //资源类型
};
```

可以操作ABRecordRef的方法非常简单，无非读与写，如下：

```objectivec
//获取记录类型
ABRecordType ABRecordGetRecordType(ABRecordRef record);
//获取记录中的数据
CFTypeRef ABRecordCopyValue(ABRecordRef record, ABPropertyID property);
//设置记录中的数据
bool ABRecordSetValue(ABRecordRef record, ABPropertyID property, CFTypeRef value, CFErrorRef* error);
//删除记录中的数据
bool ABRecordRemoveValue(ABRecordRef record, ABPropertyID property, CFErrorRef* error);
```

### 六、联系人组

    在iOS系统的联系人应用中，我们可以对联系人进行分组，如下图所示：

![](https://oscimg.oschina.net/oscnet/b2410a93c1ff384b007f91c725de5cb289f.jpg)

AddressBook框架中的如下方法与联系人组操作相关：

```objectivec
//创建一个联系人组记录
ABRecordRef ABGroupCreate(void);
//在指定的资源中创建
ABRecordRef ABGroupCreateInSource(ABRecordRef source);
//获取资源
ABRecordRef ABGroupCopySource(ABRecordRef group);
//获取组中的所有成员
CFArrayRef ABGroupCopyArrayOfAllMembers(ABRecordRef group);
//根据指定的排序方法获取组中所有成员
CFArrayRef ABGroupCopyArrayOfAllMembersWithSortOrdering(ABRecordRef group, ABPersonSortOrdering sortOrdering);
//向组中添加成员
bool ABGroupAddMember(ABRecordRef group, ABRecordRef person, CFErrorRef* error);
//删除组中的成员
bool ABGroupRemoveMember(ABRecordRef group, ABRecordRef member, CFErrorRef* error);
//根据某条记录获取组
ABRecordRef ABAddressBookGetGroupWithRecordID(ABAddressBookRef addressBook, ABRecordID recordID);
//获取通讯录中所有组的个数
CFIndex ABAddressBookGetGroupCount(ABAddressBookRef addressBook);
//获取通讯录中所有组
CFArrayRef ABAddressBookCopyArrayOfAllGroups(ABAddressBookRef addressBook);
CFArrayRef ABAddressBookCopyArrayOfAllGroupsInSource(ABAddressBookRef addressBook, ABRecordRef source);
```

### 七、重中之重——ABAddressBookRef对象

    前面介绍了许多操作联系人，操作组的方法，所有这些操作的基础都是基于一个通讯录对象ABAddressBookRef的。除了最前面演示的申请权限的相关方法，如下列举了与ABAddressBookRef相关的其他常用函数：

```objectivec
//存储通讯录
bool ABAddressBookSave(ABAddressBookRef addressBook, CFErrorRef* error);
//获取通讯录是否有未保存的更改
bool ABAddressBookHasUnsavedChanges(ABAddressBookRef addressBook);
//向通讯录中添加一条记录
bool ABAddressBookAddRecord(ABAddressBookRef addressBook, ABRecordRef record, CFErrorRef* error);
//移除通讯录中的一条记录
bool ABAddressBookRemoveRecord(ABAddressBookRef addressBook, ABRecordRef record, CFErrorRef* error);
//注册通讯录发生变化时的外部监听
void ABAddressBookRegisterExternalChangeCallback(ABAddressBookRef addressBook, ABExternalChangeCallback callback, void *context);
//移除通讯录发生变化时的外部监听
void ABAddressBookUnregisterExternalChangeCallback(ABAddressBookRef addressBook, ABExternalChangeCallback callback, void *context);
//将未保存的更改重置
void ABAddressBookRevert(ABAddressBookRef addressBook);
//工具方法，进行标签的本地化转换
CFStringRef ABAddressBookCopyLocalizedLabel(CFStringRef label);
```

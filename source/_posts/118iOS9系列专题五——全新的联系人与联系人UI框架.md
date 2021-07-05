---
title: iOS9系列专题五——全新的联系人与联系人UI框架
date: 2015-09-28
categories: iOS9专题
tags: []
---
## iOS9全新的联系人相关框架——Contacts Framework

### 一、引言

        在以前iOS开发中，涉及联系人相关的编程，代码都非常繁琐，并且框架的设计也不是Objective-C风格的，这使开发者用起来非常的难受。在iOS9中，apple终于解决了这个问题，全新的Contacts Framework将完全替代AddressBookFramework，AddressBookFramework也将成为历史被弃用。至于AddressBookFramework的相关api如何繁琐，在以前的博客中有记录，地址如下：

联系人信息相关编程：[http://my.oschina.net/u/2340880/blog/407347](http://my.oschina.net/u/2340880/blog/407347)。

联系人UI界面相关编程：[http://my.oschina.net/u/2340880/blog/407973](http://my.oschina.net/u/2340880/blog/407973)。

        这一新的框架是iOS9新特性中十分受欢迎的一个。apple的Objective—C体系也更加完善与强大。

### 二、让我们来添加一个联系人

        新的框架的整体思路是通过配置与请求来管理联系人，这样做有一个非常大的好处，逻辑简单，代码层次清晰。如下，通过添加一个联系人来向大家做演示：

#### 1、联系人对象：CNContact

这个对象是用来配置联系人信息的，有可变的CNMutaleContact和CNContact，区别用来读取和创建联系人。CNContact对象中有许多属性，对应联系人的一些信息。

首先，创建CNMutableContact对象：

```
 CNMutableContact * contact = [[CNMutableContact alloc]init];
```

设置联系人头像：

```
contact.imageData = UIImagePNGRepresentation([UIImage imageNamed:@"Icon-114.png"]);
```

设置联系人姓名：

```
    //设置名字
    contact.givenName = @"jaki";
    //设置姓氏
    contact.familyName = @"zhang";
```

设置联系人邮箱：

```
     CNLabeledValue *homeEmail = [CNLabeledValue labeledValueWithLabel:CNLabelHome value:@"316045346@qq.com"];
     CNLabeledValue *workEmail =[CNLabeledValue labeledValueWithLabel:CNLabelWork value:@"316045346@qq.com"];
     contact.emailAddresses = @[homeEmail,workEmail];
```

这里需要注意，emailAddresses属性是一个数组，数组中是才CNLabeledValue对象，CNLabeledValue对象主要用于创建一些联系人属性的键值对应，通过这些对应，系统会帮我们进行数据的格式化，例如CNLabelHome，就会将号码格式成家庭邮箱的格式，相应的其他键如下：

```
//家庭
CONTACTS_EXTERN NSString * const CNLabelHome                             NS_AVAILABLE(10_11, 9_0);
//工作
CONTACTS_EXTERN NSString * const CNLabelWork                             NS_AVAILABLE(10_11, 9_0);
//其他
CONTACTS_EXTERN NSString * const CNLabelOther                            NS_AVAILABLE(10_11, 9_0);

// 邮箱地址
CONTACTS_EXTERN NSString * const CNLabelEmailiCloud                      NS_AVAILABLE(10_11, 9_0);

// url地址
CONTACTS_EXTERN NSString * const CNLabelURLAddressHomePage               NS_AVAILABLE(10_11, 9_0);

// 日期
CONTACTS_EXTERN NSString * const CNLabelDateAnniversary                  NS_AVAILABLE(10_11, 9_0);
```

设置联系人电话：

```
contact.phoneNumbers = @[[CNLabeledValue labeledValueWithLabel:CNLabelPhoneNumberiPhone value:[CNPhoneNumber phoneNumberWithStringValue:@"12344312321"]]];
```

联系人电话的配置方式和邮箱类似，键值如下：

```
CONTACTS_EXTERN NSString * const CNLabelPhoneNumberiPhone                NS_AVAILABLE(10_11, 9_0);
CONTACTS_EXTERN NSString * const CNLabelPhoneNumberMobile                NS_AVAILABLE(10_11, 9_0);
CONTACTS_EXTERN NSString * const CNLabelPhoneNumberMain                  NS_AVAILABLE(10_11, 9_0);
CONTACTS_EXTERN NSString * const CNLabelPhoneNumberHomeFax               NS_AVAILABLE(10_11, 9_0);
CONTACTS_EXTERN NSString * const CNLabelPhoneNumberWorkFax               NS_AVAILABLE(10_11, 9_0);
CONTACTS_EXTERN NSString * const CNLabelPhoneNumberOtherFax              NS_AVAILABLE(10_11, 9_0);
CONTACTS_EXTERN NSString * const CNLabelPhoneNumberPager                 NS_AVAILABLE(10_11, 9_0);
```

这里的CNPhoneNumber对象也是iOS9中的一个新的类，专门用来创建电话号码，之中方法如下：

```
@interface CNPhoneNumber : NSObject <NSCopying, NSSecureCoding>

//通过类方法创建
+ (instancetype)phoneNumberWithStringValue:(NSString *)stringValue;
//通过初始化方法创建
- (instancetype)initWithStringValue:(NSString *)string;

@property (readonly, copy, NS_NONATOMIC_IOSONLY) NSString *stringValue;

@end
```

设置联系人地址：

```
  CNMutablePostalAddress * homeAdress = [[CNMutablePostalAddress alloc]init];
    homeAdress.street = @"贝克街";
    homeAdress.city = @"伦敦";
    homeAdress.state = @"英国";
    homeAdress.postalCode = @"221B";
    contact.postalAddresses = @[[CNLabeledValue labeledValueWithLabel:CNLabelHome value:homeAdress]];
```

设置生日：

```
NSDateComponents * birthday = [[NSDateComponents  alloc]init];
    birthday.day=7;
    birthday.month=5;
    birthday.year=1992;
    contact.birthday=birthday;
```

#### 2、创建添加联系人请求：CNSaveRequest

CNSaveRequest是用于存储联系人的请求类，通过这个类，我们可以创建批量添加、修改或者删除联系人的请求，例如添加上面我们创建的联系人对象：

```
   //初始化方法
    CNSaveRequest * saveRequest = [[CNSaveRequest alloc]init];
    //添加联系人
    [saveRequest addContact:contact toContainerWithIdentifier:nil];
```

这个类中还有许多方便我们操作的方法：

```
@interface CNSaveRequest : NSObject
//添加一个联系人
- (void)addContact:(CNMutableContact *)contact toContainerWithIdentifier:(nullable NSString *)identifier;

//更新一个联系人
- (void)updateContact:(CNMutableContact *)contact;
//删除一个联系人
- (void)deleteContact:(CNMutableContact *)contact;
//添加一组联系人
- (void)addGroup:(CNMutableGroup *)group toContainerWithIdentifier:(nullable NSString *)identifier;
//更新一组联系人
- (void)updateGroup:(CNMutableGroup *)group;
//删除一组联系人
- (void)deleteGroup:(CNMutableGroup *)group;
//向组中添加子组
- (void)addSubgroup:(CNGroup *)subgroup toGroup:(CNGroup *)group NS_AVAILABLE(10_11, NA);
//在组中删除子组
- (void)removeSubgroup:(CNGroup *)subgroup fromGroup:(CNGroup *)group NS_AVAILABLE(10_11, NA);
//向组中添加成员
- (void)addMember:(CNContact *)contact toGroup:(CNGroup *)group;
//向组中移除成员
- (void)removeMember:(CNContact *)contact fromGroup:(CNGroup *)group;

@end
```

#### 3、进行联系人的写入操作:CNContactStore

CNContactStore是一个用于存取联系人的上下文桥梁，现在，把我们创建的添加联系人的请求写入：

```
    CNContactStore * store = [[CNContactStore alloc]init];
    [store executeSaveRequest:saveRequest error:nil];
```

在模拟器上运行程序，打开联系人，效果如下：

联系人界面：

![](http://static.oschina.net/uploads/space/2015/0928/145818_XI8e_2340880.png)

联系人详情：

![](http://static.oschina.net/uploads/space/2015/0928/145821_wx4x_2340880.png)

  
 

### 三、获取格式化的联系人信息

iOS9中，ContactFramework也为开发者提供了非常方便的格式化信息的方法，还拿我们上面创建的联系人对象举例：

#### 1、获取格式化的联系人姓名

```
    NSString * foematter =[CNContactFormatter stringFromContact:contact style:CNContactFormatterStyleFullName];
    NSLog(@"%@",foematter);
```

这个运行后会打印出jaki zhang，其中style风格枚举如下：

```
typedef NS_ENUM(NSInteger, CNContactFormatterStyle)
{
    //获取全名
    CNContactFormatterStyleFullName,
   //获取拼音全名
    CNContactFormatterStylePhoneticFullName,
} NS_ENUM_AVAILABLE(10_11, 9_0);
```

#### 2、获取格式化的联系人地址

```
    NSString * foematter =[CNPostalAddressFormatter stringFromPostalAddress:homeAdress style:CNPostalAddressFormatterStyleMailingAddress];
    NSLog(@"%@",foematter);
```

打印如下：

![](http://static.oschina.net/uploads/space/2015/0928/151520_bayu_2340880.png)

### 四、提取联系人

        在开发中，提取联系人的使用率要远远高于创建联系人，ContactFramework提取联系人的方式，类似于数据库的检索方式，通过配置条件，提取出我们需要的数据，例如：

```
    CNContactStore * stroe = [[CNContactStore alloc]init];
    //检索条件，检索所有名字中有zhang的联系人
    NSPredicate * predicate = [CNContact predicateForContactsMatchingName:@"zhang"];
    //提取数据
    NSArray * contacts = [stroe unifiedContactsMatchingPredicate:predicate keysToFetch:@[CNContactGivenNameKey] error:nil];
```

keysToFetch是设置提取联系人的哪些数据，如上则只提取出检索联系人的名字。

同样，也可以通过请求的方式来对联系人进行遍历：

```
    CNContactStore * stroe = [[CNContactStore alloc]init];
    CNContactFetchRequest * request = [[CNContactFetchRequest alloc]initWithKeysToFetch:@[CNContactPhoneticFamilyNameKey]];
    [stroe enumerateContactsWithFetchRequest:request error:nil usingBlock:^(CNContact * _Nonnull contact, BOOL * _Nonnull stop) {
        NSLog(@"%@",contact);
    }];
```

### 五、ContactFramework UI相关

iOS9中，系统也为我们封装好了一套联系人的UI界面，用起来也十分方便，主要新增的controller有两个：

CNContactPickerViewController：展示联系人列表的controller

CNContactViewController：展示联系人详细信息的controller

示例如下：

弹出联系人列表：

```
    CNContactPickerViewController * con = [[CNContactPickerViewController alloc]init];
    [self presentViewController:con animated:YES completion:nil];
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/0928/163302_RfwI_2340880.png)

联系人逻辑的相关处理主要在CNContactPickerDelegate中完成：

```
//视图取消时 调用的方法
- (void)contactPickerDidCancel:(CNContactPickerViewController *)picker;
//选中与取消选中时调用的方法
- (void)contactPicker:(CNContactPickerViewController *)picker didSelectContact:(CNContact *)contact;
- (void)contactPicker:(CNContactPickerViewController *)picker didSelectContactProperty:(CNContactProperty *)contactProperty;
- (void)contactPicker:(CNContactPickerViewController *)picker didSelectContacts:(NSArray<CNContact*> *)contacts;
- (void)contactPicker:(CNContactPickerViewController *)picker didSelectContactProperties:(NSArray<CNContactProperty*> *)contactProperties;
```

CNContactViewController则是用来显示具体联系人的详细信息的，比如：

```
    CNContactViewController * con = [CNContactViewController viewControllerForContact:contact];
    [self presentViewController:con animated:YES completion:nil];
```

相关代理回调函数如下：

```
//将要展示联系人信息与已经展示联系人信息的回调
- (BOOL)contactViewController:(CNContactViewController *)viewController shouldPerformDefaultActionForContactProperty:(CNContactProperty *)property;
- (void)contactViewController:(CNContactViewController *)viewController didCompleteWithContact:(nullable CNContact *)contact;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

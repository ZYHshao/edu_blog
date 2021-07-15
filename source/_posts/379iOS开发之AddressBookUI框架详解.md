---
title: iOS开发之AddressBookUI框架详解
date: 2018-08-23
categories: iOS逻辑初窥
tags: []
---
## iOS开发之AddressBookUI框架详解

### 一、关于AddressBookUI框架

    AddressbookUI是iOS开发框架中提供的一套通讯录界面组件。其中封装好了一套选择联系人，查看联系人的界面，在需要时开发者可以直接调用。当然对于联系人界面，开发者也可以进行完全的自定义，下面链接博客中介绍了如何使用AddressBook框架操作通讯录与联系人。

[https://my.oschina.net/u/2340880/blog/1930414](https://my.oschina.net/u/2340880/blog/1930414)

    AddressBookUI框架主要提供了如下几个类：

ABNewPersonViewController：新建联系人界面视图控制器

ABPeoplePickerNavigationController：从通讯录选择联系人界面视图控制器

ABPersonViewController：联系人详情界面视图控制器

ABUnknownPersonViewController：一个未在当前通讯录中的联系人查看界面，可以添加和编辑

### 二、ABNewPersonViewController新建联系人界面

    ABNewPersonViewController类的使用非常简单，示例如下：

```objectivec
ABNewPersonViewController *picker = [[ABNewPersonViewController alloc] init];
picker.newPersonViewDelegate = self;
[self presentModalViewController:picker animated:YES];
```

效果如下图所示：

![](https://oscimg.oschina.net/oscnet/8eedefe345a3deb00747e8036aee4e1feb3.jpg)

ABNewPersonViewController解析如下：

```objectivec
//代理
@property(nonatomic,assign,nullable) id<ABNewPersonViewControllerDelegate> newPersonViewDelegate;
//通讯录实例 只读
@property(nonatomic,readwrite,nullable) ABAddressBookRef addressBook;
//联系人 只读
@property(nonatomic,readwrite,nullable) ABRecordRef displayedPerson;
//联系人组 只读
@property(nonatomic,readwrite,nullable) ABRecordRef parentGroup;
```

联系人的新建回调可以在代理方法中处理，如下：

```objectivec
@protocol ABNewPersonViewControllerDelegate <NSObject>
//新建联系人完成后的回调
- (void)newPersonViewController:(ABNewPersonViewController *)newPersonView didCompleteWithNewPerson:(nullable ABRecordRef)person;
@end
```

### 三、ABPeoplePickerNavigationController选择联系人界面

    ABPeoplePickerNavigationController是用户通讯录界面，开发者在需要用户选择联系人时，可以直接调用这个界面来让用户进行选择，示例如下：

```objectivec
ABPeoplePickerNavigationController *vc = [[ABPeoplePickerNavigationController alloc] init];
vc.peoplePickerDelegate = self;
[self presentViewController:vc animated:YES completion:nil];

```

效果如下图：

![](https://oscimg.oschina.net/oscnet/52aa2a5be3c62c2cd72d8bab4f8f2ff7b41.jpg)

ABPeoplePickerNavigationController解析如下：

```objectivec
//代理
@property(nonatomic,assign,nullable) id<ABPeoplePickerNavigationControllerDelegate> peoplePickerDelegate;
//需要展示的用户联系人属性字段  数组中为属性的ID 在AddressBook框架介绍的博客中有讲解
@property(nonatomic,copy,nullable) NSArray<NSNumber*> *displayedProperties;
//通讯录实例
@property(nonatomic,readwrite,nullable) ABAddressBookRef addressBook;
//设置一个筛选条件 过滤掉不可显示的联系人
@property(nonatomic,copy,nullable) NSPredicate *predicateForEnablingPerson;
//设置一个筛选条件 过滤掉不可选择的联系人
@property(nonatomic,copy,nullable) NSPredicate *predicateForSelectionOfPerson;
//设置一个筛选条件 过滤掉不可显示的属性
@property(nonatomic,copy,nullable) NSPredicate *predicateForSelectionOfProperty;
```

用来进行联系人筛选的属性定义如下：

```objectivec
extern NSString * const ABPersonNamePrefixProperty NS_AVAILABLE_IOS(8_0);
extern NSString * const ABPersonGivenNameProperty NS_AVAILABLE_IOS(8_0); 
extern NSString * const ABPersonMiddleNameProperty NS_AVAILABLE_IOS(8_0); 
extern NSString * const ABPersonFamilyNameProperty NS_AVAILABLE_IOS(8_0);  
extern NSString * const ABPersonNameSuffixProperty NS_AVAILABLE_IOS(8_0); 
extern NSString * const ABPersonPreviousFamilyNameProperty NS_AVAILABLE_IOS(8_0);  
extern NSString * const ABPersonNicknameProperty NS_AVAILABLE_IOS(8_0); 
extern NSString * const ABPersonPhoneticGivenNameProperty NS_AVAILABLE_IOS(8_0);  
extern NSString * const ABPersonPhoneticMiddleNameProperty NS_AVAILABLE_IOS(8_0); 
extern NSString * const ABPersonPhoneticFamilyNameProperty NS_AVAILABLE_IOS(8_0);    
extern NSString * const ABPersonOrganizationNameProperty NS_AVAILABLE_IOS(8_0);        
extern NSString * const ABPersonDepartmentNameProperty NS_AVAILABLE_IOS(8_0);       
extern NSString * const ABPersonJobTitleProperty NS_AVAILABLE_IOS(8_0);          
extern NSString * const ABPersonBirthdayProperty NS_AVAILABLE_IOS(8_0);        
extern NSString * const ABPersonNoteProperty NS_AVAILABLE_IOS(8_0);      
extern NSString * const ABPersonPhoneNumbersProperty NS_AVAILABLE_IOS(8_0);   
extern NSString * const ABPersonEmailAddressesProperty NS_AVAILABLE_IOS(8_0);  
extern NSString * const ABPersonUrlAddressesProperty NS_AVAILABLE_IOS(8_0);  
extern NSString * const ABPersonDatesProperty NS_AVAILABLE_IOS(8_0);    
extern NSString * const ABPersonInstantMessageAddressesProperty NS_AVAILABLE_IOS(8_0); 
extern NSString * const ABPersonRelatedNamesProperty NS_AVAILABLE_IOS(8_0);     
extern NSString * const ABPersonSocialProfilesProperty NS_AVAILABLE_IOS(8_0); 
extern NSString * const ABPersonPostalAddressesProperty NS_AVAILABLE_IOS(8_0);
```

ABPeoplePickerNavigationControllerDelegate中方法解释如下：

```objectivec
//选中联系人进行回调
- (void)peoplePickerNavigationController:(ABPeoplePickerNavigationController*)peoplePicker didSelectPerson:(ABRecordRef)person;
//选择联系人属性
- (void)peoplePickerNavigationController:(ABPeoplePickerNavigationController*)peoplePicker didSelectPerson:(ABRecordRef)person property:(ABPropertyID)property identifier:(ABMultiValueIdentifier)identifier;
//取消选择
- (void)peoplePickerNavigationControllerDidCancel:(ABPeoplePickerNavigationController *)peoplePicker;
```

### 四、ABPersonViewController联系人详情界面

    ABPersonViewController是联系人的详情展示界面，简单使用如下：

```objectivec
CFErrorRef error = NULL;
ABAddressBookRef addressBook = ABAddressBookCreateWithOptions(NULL, &error);
CFArrayRef peopleArray = ABAddressBookCopyArrayOfAllPeople(addressBook);
ABRecordRef person = CFArrayGetValueAtIndex(peopleArray, 0);
ABPersonViewController *viewController = [[ABPersonViewController alloc] init];
viewController.personViewDelegate = self;
viewController.displayedPerson = person;
viewController.allowsActions = NO;
viewController.allowsEditing = YES;
viewController.displayedProperties = @[[NSNumber numberWithInt:kABPersonPhoneProperty]];
[self presentViewController:viewController animated:YES completion:nil];
```

界面如下：

![](https://oscimg.oschina.net/oscnet/dbdceff3f2d043caa93be1503b323e10608.jpg)

ABPersonViewController中常用属性方法解析如下：

```objectivec
//代理
@property(nonatomic,assign,nullable) id<ABPersonViewControllerDelegate> personViewDelegate;
//通讯录实例
@property(nonatomic,readwrite,nullable) ABAddressBookRef addressBook;
//联系人记录实例
@property(nonatomic,readwrite) ABRecordRef displayedPerson;
//展示的属性字段
@property(nonatomic,copy,nullable) NSArray<NSNumber*> *displayedProperties;
//是否允许编辑
@property(nonatomic) BOOL allowsEditing;
//是否允许活动按钮 例如分享
@property(nonatomic) BOOL allowsActions;
//是否允许关联其他联系人
@property(nonatomic) BOOL shouldShowLinkedPeople;
//设置属性高亮
- (void)setHighlightedItemForProperty:(ABPropertyID)property withIdentifier:(ABMultiValueIdentifier)identifier;
```

ABPersonViewControllerDelegate中方法解释如下：

```objectivec
//选择属性发送时调用
- (BOOL)personViewController:(ABPersonViewController *)personViewController shouldPerformDefaultActionForPerson:(ABRecordRef)person property:(ABPropertyID)property identifier:(ABMultiValueIdentifier)identifier;
```

### 五、关于ABUnknownPersonViewController

     ABUnknownPersonViewController界面与ABPersonViewController基本一致，不同的是，ABPersonViewController需要使用一个通讯录中已经存在的联系人作为参数进行展示，ABUnknownPersonViewController则不然，你可以使用一个通讯录中不存在的联系人对象来进行界面的渲染，并且支持用户选择将此联系人存入通讯录中。示例如下：

```objectivec
ABUnknownPersonViewController *unknown=[[ABUnknownPersonViewController alloc]init];
unknown.displayedPerson=ABPersonCreate();
unknown.allowsAddingToAddressBook=YES;//允许添加
[self presentViewController:unknown animated:YES completion:nil];
```

ABUnknownPersonViewController中属性方法解释如下：

```objectivec
//代理
@property(nonatomic,assign,nullable) id<ABUnknownPersonViewControllerDelegate> unknownPersonViewDelegate;
//通讯录实例对象
@property(nonatomic,readwrite,nullable) ABAddressBookRef addressBook;
//联系人实例
@property(nonatomic,readwrite) ABRecordRef displayedPerson;
//提示名字
@property(nonatomic,copy,nullable) NSString *alternateName;
//提示信息
@property(nonatomic,copy,nullable) NSString *message;
//是否允许活动
@property(nonatomic) BOOL allowsActions;
//是否允许添加电话本
@property(nonatomic) BOOL allowsAddingToAddressBook;
```

ABUnknownPersonViewControllerDelegate方法：

```objectivec
//联系人解释时调用
- (void)unknownPersonViewController:(ABUnknownPersonViewController *)unknownCardViewController didResolveToPerson:(nullable ABRecordRef)person;
//发送活动
- (BOOL)unknownPersonViewController:(ABUnknownPersonViewController *)personViewController shouldPerformDefaultActionForPerson:(ABRecordRef)person property:(ABPropertyID)property identifier:(ABMultiValueIdentifier)identifier;
```

---
title: iOS中表单视图第三方控件——FXForms
date: 2016-07-04
categories: iOS第三方库
tags: []
---
## iOS中表单视图第三方控件——FXForms

### 一、引言

        表单视图是移动开发中十分常用的一种UI方式。在iOS开发中，系统的UITableView可以用来创建表单视图，其界面的渲染与逻辑的处理需要开发者实现许多代理方法。FXForms是一个第三方的表单创建工具，其通过配置的方式来进行表单界面的创建，并且其中为开发者封装好了各种常用类型的表单cell。

        FXForms的github地址如下：[https://github.com/nicklockwood/FXForms](https://github.com/nicklockwood/FXForms)。

### 二、使用FXForms进行表单视图的创建

        FXForms框架中提供了一个FXFormViewController视图控制器类，开发者可以直接编写继承于这个类的ViewController来便捷的创建表单界面，首先，FXForms是通过节点配置的方式来进行表单的创建的，表单中每一个cell都是一个节点，这个节点可以是简单的单节点，也可以是父节点，点击父节点后，会跳转新的视图控制器，父节点中可以进行层层嵌套。对于每一个节点，开发者可以设置一个节点类型，不同的节点类型将展现不同的UI，实现不同的功能。

        FXForms中的节点由FXForm协议来进行配置，创建一个简单的表单视图，示例如下：

```objectivec
//视图控制器类部分
@interface ViewController : FXFormViewController
@end
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    //节点信息设置
    self.formController.form = [MyForm new];
}
@end
//节点类配置
@interface MyForm : NSObject<FXForm>
@property(nonatomic,strong)NSString * email;
@property(nonatomic,strong)NSString * password;
@property(nonatomic,assign)BOOL rememberMe;
@end
@implementation MyForm
@end
```

上面的MyForm类中只定义了一些属性，并没有进行任何方法的实现，FXForms框架中实现了这样的功能，如果开发者不进行节点信息的配置，则FXForms会自动根据节点配置类中所有的属性来推断节点的类型，如上所示，NSString类型的属性会被自动推断成带文本框的cell，BOOL类型的属性会被自动推断成带UISwitch控件的cell。运行效果如下：

![](http://static.oschina.net/uploads/space/2016/0704/113502_cL03_2340880.png)

开发者可以为节点配置类中的每一个属性提供一个约定好的方法，在方法中对此属性对应的节点进行配置，这个约定好的方法名需要与属性对应，其格式是使用属性名加上Field，示例如下：

```objectivec

@implementation MyForm
//方法名必须是 属性名+Field 返回为NSDictionary字典 字典中为节点的配置信息
-(NSDictionary *)emailField{
    //配置节点的类型 点击后 将弹出时间选择控件
    return @{FXFormFieldType:FXFormFieldTypeDate};
}
-(NSDictionary *)passwdField{
    //设置节点名称
    return @{FXFormFieldTitle:@"名称"};
}
-(NSDictionary *)rememberMeField{
    //设置节点头视图名称
    return @{FXFormFieldHeader:@"配置"};
}
@end
```

运行工程，效果如下图所示：

![](http://static.oschina.net/uploads/space/2016/0704/114842_GSRM_2340880.png)

返回的配置字典中可以用来配置的属性定义如下：

```objectivec
//配置此节点的标识符
UIKIT_EXTERN NSString *const FXFormFieldKey;
//配置此节点的类型
UIKIT_EXTERN NSString *const FXFormFieldType; 
//指定当前节点属性对应的类 一般不需设置
UIKIT_EXTERN NSString *const FXFormFieldClass;
//设置当前节点对应的cell类
UIKIT_EXTERN NSString *const FXFormFieldCell;
//设置当前节点显示的名称
UIKIT_EXTERN NSString *const FXFormFieldTitle;
//设置当前节点的placeHolder
UIKIT_EXTERN NSString *const FXFormFieldPlaceholder;
//设置节点上默认显示的文字
UIKIT_EXTERN NSString *const FXFormFieldDefaultValue; 
//设置选项数组 这个属性的设置 必须配合特定配型的cell使用
UIKIT_EXTERN NSString *const FXFormFieldOptions;
//如果某个节点是一个数组 则FXFormFieldTemplate可以用来设置数组中节点的属性
UIKIT_EXTERN NSString *const FXFormFieldTemplate;
//进行类型转换
UIKIT_EXTERN NSString *const FXFormFieldValueTransformer;
//设置节点的触发方法
UIKIT_EXTERN NSString *const FXFormFieldAction;
//连接StoryboardSegue
UIKIT_EXTERN NSString *const FXFormFieldSegue;
//设置节点头部内容
UIKIT_EXTERN NSString *const FXFormFieldHeader;
//设置节点尾部内容
UIKIT_EXTERN NSString *const FXFormFieldFooter;
//设置是否是内嵌节点 对于父节点或者数组类界定 这个如果设置为@YES 则会在当前界面中展示表单 如果设置为@NO，则会在新的视图控制器中展示
UIKIT_EXTERN NSString *const FXFormFieldInline;
//对于数组类型的节点，设置是否支持排序 设置为@YES则为支持排序
UIKIT_EXTERN NSString *const FXFormFieldSortable;
//设置选中cell后跳转的ViewController
UIKIT_EXTERN NSString *const FXFormFieldViewController;
```

关于设置节点的类型，FXFormFieldType可以设置的值有如下几种：

```objectivec
//默认的节点类型
UIKIT_EXTERN NSString *const FXFormFieldTypeDefault;
//文本标签节点类型
UIKIT_EXTERN NSString *const FXFormFieldTypeLabel;
//输入框节点类型
UIKIT_EXTERN NSString *const FXFormFieldTypeText;
//长文本输入节点类型
UIKIT_EXTERN NSString *const FXFormFieldTypeLongText; 
//URL节点类型
UIKIT_EXTERN NSString *const FXFormFieldTypeURL;
//Email节点类型
UIKIT_EXTERN NSString *const FXFormFieldTypeEmail; 
//号码节点类型
UIKIT_EXTERN NSString *const FXFormFieldTypePhone; 
//密码框节点类型
UIKIT_EXTERN NSString *const FXFormFieldTypePassword;
//数字节点类型
UIKIT_EXTERN NSString *const FXFormFieldTypeNumber;
//只允许输入整数的节点类型
UIKIT_EXTERN NSString *const FXFormFieldTypeInteger;
//无符号整数节点类型
UIKIT_EXTERN NSString *const FXFormFieldTypeUnsigned; 
//浮点节点类型
UIKIT_EXTERN NSString *const FXFormFieldTypeFloat;
//BOOL节点类型 默认带UISwitch控件
UIKIT_EXTERN NSString *const FXFormFieldTypeBoolean;
//选项节点类型 默认带对号符号
UIKIT_EXTERN NSString *const FXFormFieldTypeOption;
//日期节点类型
UIKIT_EXTERN NSString *const FXFormFieldTypeDate;
//时间节点类型
UIKIT_EXTERN NSString *const FXFormFieldTypeTime;
//日期时间节点类型
UIKIT_EXTERN NSString *const FXFormFieldTypeDateTime;
//图片节点类型
UIKIT_EXTERN NSString *const FXFormFieldTypeImage;
```

FXForms中也提供了许多封装好的cell，列举如下：

```objectivec
//默认的cell
@interface FXFormDefaultCell : FXFormBaseCell
@end

//带文本输入框的cell
@interface FXFormTextFieldCell : FXFormBaseCell
@property (nonatomic, readonly) UITextField *textField;
@end

//带文本输入视图的cell
@interface FXFormTextViewCell : FXFormBaseCell
@property (nonatomic, readonly) UITextView *textView;
@end

//带UISwitch控件的cell
@interface FXFormSwitchCell : FXFormBaseCell
@property (nonatomic, readonly) UISwitch *switchControl;
@end

//带UIStepper控件的cell
@interface FXFormStepperCell : FXFormBaseCell
@property (nonatomic, readonly) UIStepper *stepper;
@end

//带UISlider控件的cell
@interface FXFormSliderCell : FXFormBaseCell
@property (nonatomic, readonly) UISlider *slider;
@end

//带日期选择控件的cell
@interface FXFormDatePickerCell : FXFormBaseCell
@property (nonatomic, readonly) UIDatePicker *datePicker;
@end

//带图片选择控件的cell
@interface FXFormImagePickerCell : FXFormBaseCell
@property (nonatomic, readonly) UIImageView *imagePickerView;
@property (nonatomic, readonly) UIImagePickerController *imagePickerController;
@end

//带自定义PickerView的cell
@interface FXFormOptionPickerCell : FXFormBaseCell
@property (nonatomic, readonly) UIPickerView *pickerView;
@end

//带UISegmentedControl控件的cell
@interface FXFormOptionSegmentsCell : FXFormBaseCell
@property (nonatomic, readonly) UISegmentedControl *segmentedControl;
@end

```

还有一点需要注意，如果是继承与FXFormViewController的视图控制器，其节点设置的action方法要在视图控制器中进行实现。

### 三、通过协议方法来进行节点配置

        上面演示的创建表格视图的方式是在节点配置类中创建属性，分别配置属性的节点信息来创建每一个cell，开发者也可以不创建属性，或者创建属性但是不以属性为节点来进行cell配置，使用FXFrom协议的方法，也可以完成节点的创建和配置，示例如下：

```objectivec
@interface MyForm : NSObject<FXForm>
@end
@implementation MyForm
//创建与配置节点
- (NSArray *)fields
{
    return @[
             //这里面配置字典的方法和属性字典的配置方法一一致
             @{FXFormFieldKey: @"email", FXFormFieldTitle: @"email"},
             @{FXFormFieldKey: @"phone", FXFormFieldTitle: @"phone"},
             @{FXFormFieldKey: @"address", FXFormFieldTitle: @"address"},
             @{FXFormFieldKey: @"name", FXFormFieldTitle: @"name"}
             ];
}

@end
```

效果如下：

![](http://static.oschina.net/uploads/space/2016/0704/143132_JV8d_2340880.png)

-(NSArray *)fields方法是FXForm协议中的一个方法，在这个方法中，可以直接进行节点的创建和配置，FXForm协议中还提供了两个方法，意义如下：

```objectivec
//这个方法用于配置额外的节点，如果需要某些节点不对应任何属性，可以在这个方法中配置
- (NSArray *)extraFields;
//这个方法需要返回一个字符串数组，如果需要某些属性不对应节点，即有属性的存在，但是不生成cell，可以将属性名传入返回
- (NSArray *)excludedFields;
```

        节点也可以进行复合，例如可以将一个节点配置类作为属性设置给另一个节点配置类，示例如下：

```objectivec
//子节点信息配置类
@interface SubForm : NSObject<FXForm>
@property(nonatomic,assign)NSInteger age;
@property(nonatomic,assign)NSDate * date;
@end
@implementation SubForm
@end

//父节点信息配置类
@interface MyForm : NSObject<FXForm>
@property(nonatomic,strong)NSString * email;
@property(nonatomic,strong)NSString * passwd;
@property(nonatomic,assign)BOOL rememberMe;
//其中有属性为子节点
@property(nonatomic,strong)SubForm * subForm;
@end
@implementation MyForm
@end
```

子节点会被默认包装在新的视图控制器中，也可以设置FXFormFieldInline为@YES来使其复合进当前视图控制器，效果如下：

![](http://static.oschina.net/uploads/space/2016/0704/144157_JXx3_2340880.png)              ![](http://static.oschina.net/uploads/space/2016/0704/144210_mGiO_2340880.png)

### 四、关于自定义视图控制器

        如果开发者的视图控制器并不是继承于FXFormViewController，也可以使用FXForms来快捷的创建表单视图，开发者自定义的视图控制器需要遵守FXFormControllerDelegate协议，示例如下:

```objectivec
@interface ViewController : UIViewController<FXFormControllerDelegate>
//系统的tableView
@property(nonatomic,strong)UITableView * tableView;
//FX表单控制器
@property(nonatomic,strong)FXFormController * formController;
@end
@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.tableView = [[UITableView alloc]initWithFrame:self.view.frame style:UITableViewStyleGrouped];
    self.formController = [[FXFormController alloc] init];
    self.formController.tableView = self.tableView;
    self.formController.delegate = self;
    self.formController.form = [[MyForm alloc] init];
    self.formController.form = [MyForm new];
    [self.view addSubview:self.tableView];
}
@end
```

上面的代码极大了简化了ViewController中的代码量。

### 五、对Cell进行属性设置

        在进行节点属性字典的配置时，可以通过访问属性路径的方式来对cell的属性进行一些配置，例如：

```objectivec
-(NSDictionary *)passwdField{
    return @{@"textLabel.textColor":[UIColor redColor]};
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

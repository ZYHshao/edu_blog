---
title: iOS中正则表达式的使用
date: 2015-04-19
categories: iOS逻辑初窥
tags: [iOS编程]              
---
## 正则表达式在iOS开发中的应用

正则表达式在字符串查找，替换，检测中的应用非常广泛，正则表达式是什么，有怎样的语法，我的另一篇博客中有详细的介绍：[http://my.oschina.net/u/2340880/blog/403508](http://my.oschina.net/u/2340880/blog/403508)。这里只简单说一下其概念 ，正则表达式是一种语法小巧简单的语言，用来约束一些过滤字符串条的条件。很多开发工具都有支持正则表达式的内容，IOS也不例外，在IOS中NSRegularExpression类就是一个专门来处理正则表达式的类。

### 一、初始化方法

初始化NSRegularExpression的方法有两种，一个init方法和一个类方法。其作用基本是一样的

\+ (NSRegularExpression *)regularExpressionWithPattern:(NSString *)pattern options:(NSRegularExpressionOptions)options error:(NSError **)error;

\- (instancetype)initWithPattern:(NSString *)pattern options:(NSRegularExpressionOptions)options error:(NSError **)error 

其中，pattern是正则表达式，options是参数。对于option参数，它是一个枚举，表示正则模式的设置，如下：

```
typedef NS_OPTIONS(NSUInteger, NSRegularExpressionOptions) {
   NSRegularExpressionCaseInsensitive             = 1 << 0, //不区分字母大小写的模式
   NSRegularExpressionAllowCommentsAndWhitespace  = 1 << 1, //忽略掉正则表达式中的空格和#号之后的字符
   NSRegularExpressionIgnoreMetacharacters        = 1 << 2, //将正则表达式整体作为字符串处理
   NSRegularExpressionDotMatchesLineSeparators    = 1 << 3, //允许.匹配任何字符，包括换行符  
   NSRegularExpressionAnchorsMatchLines           = 1 << 4, //允许^和$符号匹配行的开头和结尾
   NSRegularExpressionUseUnixLineSeparators       = 1 << 5, //设置\n为唯一的行分隔符，否则所有的都有效。
   NSRegularExpressionUseUnicodeWordBoundaries    = 1 << 6 //使用Unicode TR#29标准作为词的边界，否则所有传统正则表达式的词边界都有效
};
```

注意：1、NSRegularExpressionCaseInsensitive模式下正则表达式 aBc 会匹配到abc.

            2、NSRegularExpressionIgnoreMetacharacters模式下正则表达式a b c 会匹配到abc，正则表达式ab#c会匹配到ab。

            3、NSRegularExpressionAllowCommentsAndWhitespace模式下正则表达式\[a-z\]，会匹配到\[a-z\]。

### 二、获取查询结果

初始化完毕正则表达式的处理类后，我们需要进行正则表达式的查询，IOS官方提供了两种模式：

#### 1、带block模式的方法：

\- (void)enumerateMatchesInString:(NSString *)string options:(NSMatchingOptions)options range:(NSRange)range usingBlock:(void (^)(NSTextCheckingResult \*result, NSMatchingFlags flags, BOOL \*stop))block;

使用举例：

```
NSRegularExpression * regex = [[NSRegularExpression alloc]initWithPattern:@"[a-z]" options:NSRegularExpressionCaseInsensitive error:nil];
    [regex enumerateMatchesInString:@"124a" options:NSMatchingReportProgress range:NSMakeRange(0, 4) usingBlock:^(NSTextCheckingResult *result, NSMatchingFlags flags, BOOL *stop) {
        NSLog(@"%@",result);
    } ];
```

注意：1、这个函数的一个参数options是一个枚举，设置回调的方式，如下：

```
typedef NS_OPTIONS(NSUInteger, NSMatchingOptions) {
   NSMatchingReportProgress         = 1 << 0, //找到最长的匹配字符串后调用block回调
   NSMatchingReportCompletion       = 1 << 1, //找到任何一个匹配串后都回调一次block
   NSMatchingAnchored               = 1 << 2, //从匹配范围的开始出进行极限匹配
   NSMatchingWithTransparentBounds  = 1 << 3, //允许匹配的范围超出设置的范围
   NSMatchingWithoutAnchoringBounds = 1 << 4  //禁止^和$自动匹配行还是和结束
};
```

            2、block回调中的flags枚举对应如下：

```
typedef NS_OPTIONS(NSUInteger, NSMatchingFlags) {
   NSMatchingProgress               = 1 << 0, //匹配到最长串是被设置     
   NSMatchingCompleted              = 1 << 1, //全部分配完成后被设置    
   NSMatchingHitEnd                 = 1 << 2, //匹配到设置范围的末尾时被设置   
   NSMatchingRequiredEnd            = 1 << 3, //当前匹配到的字符串在匹配范围的末尾时被设置     
   NSMatchingInternalError          = 1 << 4  //由于错误导致的匹配失败时被设置   
};
```

            3、还有一点需要注意，就是那个bool值stop，我们可以在block块中设置它为YES，之后便会停止查找。

#### 2、非block的方法

这个方法会返回一个结果数组，将所有匹配的结果返回

\- (NSArray *)matchesInString:(NSString *)string options:(NSMatchingOptions)options range:(NSRange)range;

这个方法会返回匹配到得字符串的个数

\- (NSUInteger)numberOfMatchesInString:(NSString *)string options:(NSMatchingOptions)options range:(NSRange)range;

这个方法会返回第一个查询到得结果，这个NSTextCheckingResult对象中有一个range属性，可以得到匹配到的字符串的范围。

\- (NSTextCheckingResult *)firstMatchInString:(NSString *)string options:(NSMatchingOptions)options range:(NSRange)range;

这个方法直接返回匹配到得范围，NSRange。

\- (NSRange)rangeOfFirstMatchInString:(NSString *)string options:(NSMatchingOptions)options range:(NSRange)range;

### 三、一个辅助方法

在NSRegularExpression类中还提供了一个辅助方法：

\+ (NSString *)escapedPatternForString:(NSString *)string;

它可以帮助我们将正则表达式加上"\\"进行保护，将元字符转化成字面值。

到此，在IOS中正则表达式的基本用法就介绍完了，希望正则表达式的应用，能为你的项目节省更多时间。

疏漏之处 欢迎指正

学习使用 欢迎转载

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592
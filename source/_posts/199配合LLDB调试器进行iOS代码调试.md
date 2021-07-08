---
title: 配合LLDB调试器进行iOS代码调试
date: 2016-04-24
categories: 日常技巧
tags: []
---
## 配合LLDB调试器进行iOS代码调试

        在一款完整iOS移动应用的开发中，代码的调试和编写占着同等重要的地位。Xcode默认使用LLDB作为代码调试器，LLDB功能丰富且强大，恰当的使用它，可以帮助开发者事半功倍的完成代码调试的工作。

#### 1.expression代码执行指令

        关于LLDB调试器，最常用的指令应该是p与po了，开发者常用这两个命令来进行对象的打印操作，p会打印出对象地址和类型，po则会额外打印出对象的值得内容，实际上，这两个命令都是expression相关命令的简写。expression命令也并非简单的打印命令，实际上它是一个执行代码命令，执行后将返回值进行打印，这个命令有一个十分强大的特点，它可以真实改变程序运行中变量的值。例如在如下代码中的int c = a+b 一行添加一个断点，运行工程。

```
    int a = 0;
    int b = 1;
    int c = a+b;
    NSLog(@"%d",c);
```

如果开发者不进行任何认为操作，此时打印出的值应该是1，为了测试，可以在调试区输入如下命令：

```
(lldb) expression a=1
```

此后跳过断点继续运行程序，可以看到打印的结果如下，c变成2。

```
(lldb) expression a=1
(int) $0 = 1
2016-04-24 11:39:40.213 BreakPointTest[1010:79065] 2
```

通过上面的演示，我们发现使用LLDB调试代码十分方便的一个特点，当我们知道程序某个地方可能会出现问题，为了找到解决方法，不使用LLDB时我们可能需要在代码中添加大量的打印函数，并且多次尝试修改源代码才能解决问题，如果使用LLDB的expression命令，我们不仅不需要添加额外的打印代码，也不需要直接修改源代码，在调试区进行多次调试，直到找到正确的修改方法后再对源代码修改一次即可。

#### 2.frame代码堆栈块信息相关指令

      当Xcode进入断点调试或者遇到异常程序崩溃时，在Xcode左侧的导航区都会将程序运行中的相关堆栈块信息列举出来，例如使用如下测试代码，在text方法中的int c = a+b 一行添加一个断点。

```
#import "ViewController.h"
@interface ViewController ()
{
    int ab;
}
@end
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    ab = 1;
    [self test];  
}
-(void)test{
    int a = 0;
    int b = 1;
    int c = a+b;
    NSLog(@"%d",c);
}
@end
```

当程序运行到断点处断开时，Xcode左侧的堆栈块如下图所示：

![](http://static.oschina.net/uploads/space/2016/0424/120438_RdDl_2340880.png)

从图中可以看出，程序当前处于激活状态的线程有5个，程序目前断在线程1中的test方法堆栈块中，使用frame info指令可以打印当前堆栈块的信息，示例如下：

```
(lldb) frame info
frame #0: 0x0000000102497905 BreakPointTest`-[ViewController test](self=0x00007fcd5b413320, _cmd="test") + 37 at ViewController.m:39
```

在打印的信息中，会有所在的文件名称和函数名称及堆栈块标号和内存地址。

      在实际代码调试过程中，程序运行的回溯是一个重要的方法，例如上面的代码例子，虽然现在断点断在test方法中，开发者可能需要在viewDidLoad方法中进行相关调试，例如上面viewDidLoad方法中有一个变量ab，如果想查看ab变量的值，我们就需要将当前选中调试的堆栈块选择为viewDidLoad方法所在的堆栈块，从Xcode左侧导航区可以看到，viewDidLoad方法堆栈块的标号为1，执行如下LLDB指令即可切换：

```
(lldb) frame select 1
frame #1: 0x00000001024978cb BreakPointTest`-[ViewController viewDidLoad](self=0x00007fcd5b413320, _cmd="viewDidLoad") + 91 at ViewController.m:31
   28          int a = 0;
   29          int b = 1;
   30          int c = a+b;
-> 31          NSLog(@"%d",c);
   32      }
   33      @end
```

从打印信息可以看到，现在选中的调试堆栈块已经切换到viewDidLoad方法，再使用expression指令时就可以操作这个方法中的相关变量了。

      在使用LLDB工具前，遇到这样的情况，我往往会采用打多个断点，一步步追溯代码的运行过程并检查过程中变量的值是否正确，调试起来并不十分方便，如果不小心错过了某个断点，又要重新开始，通过选择调试的frame堆栈块可以十分方便的解决这个问题。

      与frame相关的还有一个指令十分有用，下面的指令可以打印出当前堆栈块中所有对象的内容：

```
(lldb) frame variable
(ViewController *) self = 0x00007fcd5b413320
(SEL) _cmd = "test"
(int) a = 0
(int) b = 1
(int) c = 0
```

variable后面也可以添加参数名来打印特定对象的内容：

```
(lldb) frame variable a
(int) a = 0
```

#### 3.thread线程操作相关指令

      上面提到过，程序运行中会有多个激活的线程，每个线程中又有许多堆栈块，frame相关指令用于综合调试各个堆栈块，thread指令则是用于综合调试各个线程。首先Xcode左侧导航区为我们列出的线程堆栈块并不是当前线程中的所有堆栈块，使用如下命令可以打印出当前线程的所有堆栈块：

```
(lldb) thread backtrace
* thread #1: tid = 0x152f8, 0x0000000102497905 BreakPointTest`-[ViewController test](self=0x00007fcd5b413320, _cmd="test") + 37 at ViewController.m:39, queue = 'com.apple.main-thread', stop reason = breakpoint 2.1
  * frame #0: 0x0000000102497905 BreakPointTest`-[ViewController test](self=0x00007fcd5b413320, _cmd="test") + 37 at ViewController.m:39
    frame #1: 0x00000001024978cb BreakPointTest`-[ViewController viewDidLoad](self=0x00007fcd5b413320, _cmd="viewDidLoad") + 91 at ViewController.m:31
    frame #2: 0x0000000103475984 UIKit`-[UIViewController loadViewIfRequired] + 1198
    frame #3: 0x0000000103475cd3 UIKit`-[UIViewController view] + 27
    frame #4: 0x000000010334bfb4 UIKit`-[UIWindow addRootViewControllerViewIfPossible] + 61
    frame #5: 0x000000010334c69d UIKit`-[UIWindow _setHidden:forced:] + 282
    frame #6: 0x000000010335e180 UIKit`-[UIWindow makeKeyAndVisible] + 42
    frame #7: 0x00000001032d2ed9 UIKit`-[UIApplication _callInitializationDelegatesForMainScene:transitionContext:] + 4131
    frame #8: 0x00000001032d9568 UIKit`-[UIApplication _runWithMainScene:transitionContext:completion:] + 1769
    frame #9: 0x00000001032d6714 UIKit`-[UIApplication workspaceDidEndTransaction:] + 188
    frame #10: 0x0000000105d438c8 FrontBoardServices`__FBSSERIALQUEUE_IS_CALLING_OUT_TO_A_BLOCK__ + 24
    frame #11: 0x0000000105d43741 FrontBoardServices`-[FBSSerialQueue _performNext] + 178
    frame #12: 0x0000000105d43aca FrontBoardServices`-[FBSSerialQueue _performNextFromRunLoopSource] + 45
    frame #13: 0x0000000102e4a301 CoreFoundation`__CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__ + 17
    frame #14: 0x0000000102e4022c CoreFoundation`__CFRunLoopDoSources0 + 556
    frame #15: 0x0000000102e3f6e3 CoreFoundation`__CFRunLoopRun + 867
    frame #16: 0x0000000102e3f0f8 CoreFoundation`CFRunLoopRunSpecific + 488
    frame #17: 0x00000001032d5f21 UIKit`-[UIApplication _run] + 402
    frame #18: 0x00000001032daf09 UIKit`UIApplicationMain + 171
    frame #19: 0x0000000102497c3f BreakPointTest`main(argc=1, argv=0x00007fff5d768668) + 111 at main.m:14
    frame #20: 0x00000001056fe92d libdyld.dylib`start + 1
    frame #21: 0x00000001056fe92d libdyld.dylib`start + 1
```

thread list指令则可以打印出当前所有激活的线程，如下：

```
(lldb) thread list
Process 1049 stopped
* thread #1: tid = 0x152f8, 0x0000000102497905 BreakPointTest`-[ViewController test](self=0x00007fcd5b413320, _cmd="test") + 37 at ViewController.m:39, queue = 'com.apple.main-thread', stop reason = breakpoint 2.1
  thread #2: tid = 0x1531b, 0x0000000105a43ee2 libsystem_kernel.dylib`kevent64 + 10, queue = 'com.apple.libdispatch-manager'
  thread #3: tid = 0x1531c, 0x0000000105a435e2 libsystem_kernel.dylib`__workq_kernreturn + 10
  thread #4: tid = 0x15324, 0x0000000105a435e2 libsystem_kernel.dylib`__workq_kernreturn + 10
  thread #5: tid = 0x15328, 0x0000000105a435e2 libsystem_kernel.dylib`__workq_kernreturn + 10
```

thread info可以打印出当前选中调试的线程的信息：

```
(lldb) thread info
thread #1: tid = 0x152f8, 0x0000000102497905 BreakPointTest`-[ViewController test](self=0x00007fcd5b413320, _cmd="test") + 37 at ViewController.m:39, queue = 'com.apple.main-thread', stop reason = breakpoint 2.1
```

同样也可以使用thread select指令来切换选中调试的线程：

```
(lldb) thread select 2
```

thread continue指令用于继续执行当前的线程：

```
(lldb) thread continue
2016-04-24 12:29:54.562 BreakPointTest[1049:86776] 1
Resuming thread 0x152f8 in process 1049
Process 1049 resuming
```

#### 4.disassemble代码反汇编相关指令

      disassemble相关指令用于输出某段程序的汇编代码，执行disassemble指令将会反汇编当前函数：

```
BreakPointTest`-[ViewController test]:
    0x10aab7940 <+0>:  pushq  %rbp
    0x10aab7941 <+1>:  movq   %rsp, %rbp
    0x10aab7944 <+4>:  subq   $0x20, %rsp
    0x10aab7948 <+8>:  leaq   0x1711(%rip), %rax        ; @"%d"
    0x10aab794f <+15>: movq   %rdi, -0x8(%rbp)
    0x10aab7953 <+19>: movq   %rsi, -0x10(%rbp)
    0x10aab7957 <+23>: movl   $0x0, -0x14(%rbp)
    0x10aab795e <+30>: movl   $0x1, -0x18(%rbp)
->  0x10aab7965 <+37>: movl   -0x14(%rbp), %ecx
    0x10aab7968 <+40>: addl   -0x18(%rbp), %ecx
    0x10aab796b <+43>: movl   %ecx, -0x1c(%rbp)
    0x10aab796e <+46>: movl   -0x1c(%rbp), %esi
    0x10aab7971 <+49>: movq   %rax, %rdi
    0x10aab7974 <+52>: movb   $0x0, %al
    0x10aab7976 <+54>: callq  0x10aab7c80               ; symbol stub for: NSLog
    0x10aab797b <+59>: addq   $0x20, %rsp
    0x10aab797f <+63>: popq   %rbp
    0x10aab7980 <+64>: retq
```

     使用disassemble -b则会输出带字节信息的汇编代码：

```
(lldb) disassemble -b
BreakPointTest`-[ViewController test]:
    0x10aab7940 <+0>:  55                    pushq  %rbp
    0x10aab7941 <+1>:  48 89 e5              movq   %rsp, %rbp
    0x10aab7944 <+4>:  48 83 ec 20           subq   $0x20, %rsp
    0x10aab7948 <+8>:  48 8d 05 11 17 00 00  leaq   0x1711(%rip), %rax        ; @"%d"
    0x10aab794f <+15>: 48 89 7d f8           movq   %rdi, -0x8(%rbp)
    0x10aab7953 <+19>: 48 89 75 f0           movq   %rsi, -0x10(%rbp)
    0x10aab7957 <+23>: c7 45 ec 00 00 00 00  movl   $0x0, -0x14(%rbp)
    0x10aab795e <+30>: c7 45 e8 01 00 00 00  movl   $0x1, -0x18(%rbp)
->  0x10aab7965 <+37>: 8b 4d ec              movl   -0x14(%rbp), %ecx
    0x10aab7968 <+40>: 03 4d e8              addl   -0x18(%rbp), %ecx
    0x10aab796b <+43>: 89 4d e4              movl   %ecx, -0x1c(%rbp)
    0x10aab796e <+46>: 8b 75 e4              movl   -0x1c(%rbp), %esi
    0x10aab7971 <+49>: 48 89 c7              movq   %rax, %rdi
    0x10aab7974 <+52>: b0 00                 movb   $0x0, %al
    0x10aab7976 <+54>: e8 05 03 00 00        callq  0x10aab7c80               ; symbol stub for: NSLog
    0x10aab797b <+59>: 48 83 c4 20           addq   $0x20, %rsp
    0x10aab797f <+63>: 5d                    popq   %rbp
    0x10aab7980 <+64>: c3                    retq
```

      使用disassemble -c 指令可以设置输出汇编代码的行数，如下：

```
(lldb) disassemble -c 5
BreakPointTest`-[ViewController test]:
    0x10aab7940 <+0>:  pushq  %rbp
    0x10aab7941 <+1>:  movq   %rsp, %rbp
    0x10aab7944 <+4>:  subq   $0x20, %rsp
    0x10aab7948 <+8>:  leaq   0x1711(%rip), %rax        ; @"%d"
    0x10aab794f <+15>: movq   %rdi, -0x8(%rbp)
```

      使用disassemble -l只输出当前断点处汇编代码：

```
BreakPointTest`-[ViewController test] + 37 at ViewController.m:30
   29          int b = 1;
-> 30          int c = a+b;
   31          NSLog(@"%d",c);
BreakPointTest`-[ViewController test]:
->  0x10aab7965 <+37>: movl   -0x14(%rbp), %ecx
```

      使用disassemble -m混合显示汇编代码：

```
(lldb) disassemble -m
BreakPointTest`-[ViewController test] at ViewController.m:27
   26      }
   27      -(void)test{
   28          int a = 0;
BreakPointTest`-[ViewController test]:
    0x10aab7940 <+0>:  pushq  %rbp
    0x10aab7941 <+1>:  movq   %rsp, %rbp
    0x10aab7944 <+4>:  subq   $0x20, %rsp
    0x10aab7948 <+8>:  leaq   0x1711(%rip), %rax        ; @"%d"
    0x10aab794f <+15>: movq   %rdi, -0x8(%rbp)
    0x10aab7953 <+19>: movq   %rsi, -0x10(%rbp)
BreakPointTest`-[ViewController test] + 23 at ViewController.m:28
   27      -(void)test{
   28          int a = 0;
   29          int b = 1;
    0x10aab7957 <+23>: movl   $0x0, -0x14(%rbp)
BreakPointTest`-[ViewController test] + 30 at ViewController.m:29
   28          int a = 0;
   29          int b = 1;
   30          int c = a+b;
    0x10aab795e <+30>: movl   $0x1, -0x18(%rbp)
BreakPointTest`-[ViewController test] + 37 at ViewController.m:30
   29          int b = 1;
-> 30          int c = a+b;
   31          NSLog(@"%d",c);
->  0x10aab7965 <+37>: movl   -0x14(%rbp), %ecx
BreakPointTest`-[ViewController test] + 40 at ViewController.m:30
   29          int b = 1;
   30          int c = a+b;
   31          NSLog(@"%d",c);
    0x10aab7968 <+40>: addl   -0x18(%rbp), %ecx
BreakPointTest`-[ViewController test] + 43 at ViewController.m:30
   29          int b = 1;
   30          int c = a+b;
   31          NSLog(@"%d",c);
    0x10aab796b <+43>: movl   %ecx, -0x1c(%rbp)
BreakPointTest`-[ViewController test] + 46 at ViewController.m:31
   30          int c = a+b;
   31          NSLog(@"%d",c);
   32      }
    0x10aab796e <+46>: movl   -0x1c(%rbp), %esi
BreakPointTest`-[ViewController test] + 49 at ViewController.m:31
   30          int c = a+b;
   31          NSLog(@"%d",c);
   32      }
    0x10aab7971 <+49>: movq   %rax, %rdi
    0x10aab7974 <+52>: movb   $0x0, %al
    0x10aab7976 <+54>: callq  0x10aab7c80               ; symbol stub for: NSLog
BreakPointTest`-[ViewController test] + 59 at ViewController.m:32
   31          NSLog(@"%d",c);
   32      }
   33      @end
    0x10aab797b <+59>: addq   $0x20, %rsp
    0x10aab797f <+63>: popq   %rbp
    0x10aab7980 <+64>: retq
```

      使用disassemble -p进行当前行的汇编代码输出：

```
(lldb) disassemble -p
BreakPointTest`-[ViewController test]:
->  0x10aab7965 <+37>: movl   -0x14(%rbp), %ecx
    0x10aab7968 <+40>: addl   -0x18(%rbp), %ecx
    0x10aab796b <+43>: movl   %ecx, -0x1c(%rbp)
    0x10aab796e <+46>: movl   -0x1c(%rbp), %esi
```

#### 5.其他LLDB常用指令

        bt指令用于打印当前线程所有堆栈块信息：

```
(lldb) bt
* thread #1: tid = 0x19f11, 0x000000010aab7965 BreakPointTest`-[ViewController test](self=0x00007fee9c11e330, _cmd="test") + 37 at ViewController.m:30, queue = 'com.apple.main-thread', stop reason = breakpoint 2.1
  * frame #0: 0x000000010aab7965 BreakPointTest`-[ViewController test](self=0x00007fee9c11e330, _cmd="test") + 37 at ViewController.m:30
    frame #1: 0x000000010aab792b BreakPointTest`-[ViewController viewDidLoad](self=0x00007fee9c11e330, _cmd="viewDidLoad") + 91 at ViewController.m:22
    frame #2: 0x000000010ba95984 UIKit`-[UIViewController loadViewIfRequired] + 1198
    frame #3: 0x000000010ba95cd3 UIKit`-[UIViewController view] + 27
    frame #4: 0x000000010b96bfb4 UIKit`-[UIWindow addRootViewControllerViewIfPossible] + 61
    frame #5: 0x000000010b96c69d UIKit`-[UIWindow _setHidden:forced:] + 282
    frame #6: 0x000000010b97e180 UIKit`-[UIWindow makeKeyAndVisible] + 42
    frame #7: 0x000000010b8f2ed9 UIKit`-[UIApplication _callInitializationDelegatesForMainScene:transitionContext:] + 4131
    frame #8: 0x000000010b8f9568 UIKit`-[UIApplication _runWithMainScene:transitionContext:completion:] + 1769
    frame #9: 0x000000010b8f6714 UIKit`-[UIApplication workspaceDidEndTransaction:] + 188
    frame #10: 0x000000010e3638c8 FrontBoardServices`__FBSSERIALQUEUE_IS_CALLING_OUT_TO_A_BLOCK__ + 24
    frame #11: 0x000000010e363741 FrontBoardServices`-[FBSSerialQueue _performNext] + 178
    frame #12: 0x000000010e363aca FrontBoardServices`-[FBSSerialQueue _performNextFromRunLoopSource] + 45
    frame #13: 0x000000010b46a301 CoreFoundation`__CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__ + 17
    frame #14: 0x000000010b46022c CoreFoundation`__CFRunLoopDoSources0 + 556
    frame #15: 0x000000010b45f6e3 CoreFoundation`__CFRunLoopRun + 867
    frame #16: 0x000000010b45f0f8 CoreFoundation`CFRunLoopRunSpecific + 488
    frame #17: 0x000000010b8f5f21 UIKit`-[UIApplication _run] + 402
    frame #18: 0x000000010b8faf09 UIKit`UIApplicationMain + 171
    frame #19: 0x000000010aab7c5f BreakPointTest`main(argc=1, argv=0x00007fff55148668) + 111 at main.m:14
    frame #20: 0x000000010dd1e92d libdyld.dylib`start + 1
    frame #21: 0x000000010dd1e92d libdyld.dylib`start + 1
```

**  c指令****继续运行线程****和****process continue****效果一样。**

**        call指令****运行一个表达式，****和** **expression** **效果一样。**

**        detach指令****结束当前调试的线程。**

**        di指令****反汇编当前函数****与****disassemble****相同。**

**        exit指令****退出****lldb****调试器。**

**        finish指令****完成当前堆栈块的调试，****程序会继续运行。**

**        n指令****进行单步调试，****与****next****作用一样。**

**        p指令****与****expression****作用一样。**

**        print指令用于变量的打印。**

**        r指令****重新运行应用程序。**

**        quit指令结束调试。**

**        bugreport指令用于创建堆栈信息报告。**

**        command history指令用于打印LLDB调试命令记录。**

**        help指令用于查询LLDB相关调试指令的用法。**

**        apropo指令用于查询某些包含某些关键字的指令。**

**        version指令用于查询LLDB调试器的版本，如下：**

```
(lldb) version
lldb-350.0.21.3
```

        image list命令用于打印工程中所有用到的库文件。

       image相关指令还有一个十分有用的命令，image lookup --address可以查询某个内存地址的内容，如下：

```
(lldb) image lookup --address 0x000000010373e885
      Address: CoreFoundation[0x00000000000f4885] (CoreFoundation.__TEXT.__text + 996309)
      Summary: CoreFoundation`-[__NSArray0 objectAtIndex:] + 101
```

        image lookup --type用于查询某种类型中包含的属性，如下：

```
(lldb) image lookup --type UILabel
Best match found in /Users/vip/Library/Developer/Xcode/DerivedData/BreakPointTest-cearqrjqbntqcnfgiqzpxhyadewi/Build/Products/Debug-iphonesimulator/BreakPointTest.app/BreakPointTest:
id = {0x000082c1}, name = "UILabel", byte-size = 8, decl = UILabel.h:18, compiler_type = "@interface UILabel : UIView
@property ( getter = text,setter = setText:,readwrite,copy,nonatomic ) NSString * text;
@property ( getter = font,setter = setFont:,readwrite,nonatomic ) UIFont * font;
@property ( getter = textColor,setter = setTextColor:,readwrite,nonatomic ) UIColor * textColor;
@property ( getter = shadowColor,setter = setShadowColor:,readwrite,nonatomic ) UIColor * shadowColor;
@property ( getter = shadowOffset,setter = setShadowOffset:,assign,readwrite,nonatomic ) CGSize shadowOffset;
@property ( getter = textAlignment,setter = setTextAlignment:,assign,readwrite,nonatomic ) NSTextAlignment textAlignment;
@property ( getter = lineBreakMode,setter = setLineBreakMode:,assign,readwrite,nonatomic ) NSLineBreakMode lineBreakMode;
@property ( getter = attributedText,setter = setAttributedText:,readwrite,copy,nonatomic ) NSAttributedString * attributedText;
@property ( getter = highlightedTextColor,setter = setHighlightedTextColor:,readwrite,nonatomic ) UIColor * highlightedTextColor;
@property ( getter = isHighlighted,setter = setHighlighted:,assign,readwrite,nonatomic ) BOOL highlighted;
@property ( getter = isUserInteractionEnabled,setter = setUserInteractionEnabled:,assign,readwrite,nonatomic ) BOOL userInteractionEnabled;
@property ( getter = isEnabled,setter = setEnabled:,assign,readwrite,nonatomic ) BOOL enabled;
@property ( getter = numberOfLines,setter = setNumberOfLines:,assign,readwrite,nonatomic ) NSInteger numberOfLines;
@property ( getter = adjustsFontSizeToFitWidth,setter = setAdjustsFontSizeToFitWidth:,assign,readwrite,nonatomic ) BOOL adjustsFontSizeToFitWidth;
@property ( getter = baselineAdjustment,setter = setBaselineAdjustment:,assign,readwrite,nonatomic ) UIBaselineAdjustment baselineAdjustment;
@property ( getter = minimumScaleFactor,setter = setMinimumScaleFactor:,assign,readwrite,nonatomic ) CGFloat minimumScaleFactor;
@property ( getter = allowsDefaultTighteningForTruncation,setter = setAllowsDefaultTighteningForTruncation:,assign,readwrite,nonatomic ) BOOL allowsDefaultTighteningForTruncation;
@property ( getter = preferredMaxLayoutWidth,setter = setPreferredMaxLayoutWidth:,assign,readwrite,nonatomic ) CGFloat preferredMaxLayoutWidth;
@property ( getter = minimumFontSize,setter = setMinimumFontSize:,assign,readwrite,nonatomic ) CGFloat minimumFontSize;
@property ( getter = adjustsLetterSpacingToFitWidth,setter = setAdjustsLetterSpacingToFitWidth:,assign,readwrite,nonatomic ) BOOL adjustsLetterSpacingToFitWidth;
@end"
```

       x指令可以读取某段内存的二进制数据：

```
(lldb) x 0x000000010373e885
0x10373e885: 66 66 2e 0f 1f 84 00 00 00 00 00 55 48 89 e5 48  ff.........UH..H
0x10373e895: 8d 3d 6d f2 28 00 e8 c0 d9 f0 ff 48 89 05 c1 58  .=m.(......H...X
```

        LLDB的用法和技巧还有很多，它可以大大提高我们调试代码的效率，有疏漏和错误之处，还望与志同道合的朋友共同学习进步。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

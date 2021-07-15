---
title: OS X 开发：打开文件面板NSOpenPanel应用
date: 2017-07-17
categories: macOS开发
tags: []
---
## OS X 开发：打开文件面板NSOpenPanel应用

      在Mac桌面软件开发中，如果涉及到对文件的操作，无论是新建文件还是选择或读取文件，都离不开文件路径的定位，NSOpenPanel类提供了简洁的文件选择面板，其继承自NSSavePanel(一个专门用来存储文件的类)，NSOpenPanel的使用非常简单，示例如下：

```objectivec
    NSOpenPanel * panel = [NSOpenPanel openPanel];
    //设置是否解析别名
    panel.resolvesAliases = NO;
    //设置是否允许选择文件夹
    panel.canChooseDirectories = YES;
    //设置是否允许选择文件
    panel.canChooseFiles = YES;
    //设置是否允许多选
    panel.allowsMultipleSelection = YES;
    NSInteger result = [panel runModal];
    if (result==NSFileHandlingPanelOKButton) {
        NSLog(@"%@",panel.URLs);
    }
```

在使用runModel方法弹出面板后，用户可以选择面板中的文件或文件夹，如下图所示：

![](https://static.oschina.net/uploads/space/2017/0717/105822_HpXi_2340880.png)

runModel方法的返回值为NSInteger类型，其是一个枚举值，枚举的意义如下：

```objectivec
enum {
    NSFileHandlingPanelCancelButton    = NSModalResponseCancel,//用户点击了取消按钮
    NSFileHandlingPanelOKButton    = NSModalResponseOK,//用户点击了OK按钮
};
```

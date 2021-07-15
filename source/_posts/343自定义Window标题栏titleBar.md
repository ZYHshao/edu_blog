---
title: 自定义Window标题栏titleBar
date: 2017-06-20
categories: macOS开发
tags: []
---
## 自定义Window标题栏titleBar

    在进行OS X软件开发时，Window自带的标题栏十分简易，往往不能达到我们的需求，如下图：

![](https://static.oschina.net/uploads/space/2017/0620/143116_mY07_2340880.png)

在实际开发中，我们需要根据项目的需要对标题栏进行自定义。自定义标题栏主要有如下两种思路：

1.去掉系统的标题栏，使用自定义的View来做标题栏。

2.隐藏系统的标题栏，进行标题栏的透明处理。

上面两种思路中第2种要更好一些，我们可以服用系统的功能按钮，即关闭、最小化和最大化按钮。

    首先，现在Window的contentView中添加一个自定义的View，作为标题栏视图，View上可以添加图标或任意自定义的功能按钮。如下：

![](https://static.oschina.net/uploads/space/2017/0620/143532_2EsS_2340880.png)

通过如下代码来设置标题栏：

```objectivec
//将系统的标题栏设置透明    
self.window.titlebarAppearsTransparent = YES;
//将系统标题进行隐藏
self.window.titleVisibility = NSWindowTitleHidden;
//设置可以通过拖拽window背景视图进行窗口的移动
[self.window setMovableByWindowBackground:YES];
//设置window的内容部分充满整个窗口
[self.window setStyleMask:[self.window styleMask] | NSWindowStyleMaskFullSizeContentView];
//获取到windows的主视图    
NSView * themeView = self.window.contentView.superview;
//根据层级结构获取到标题栏视图
NSView * titleView = themeView.subviews[1];
titleView.autoresizesSubviews = YES;
//重新对标题栏视图的尺寸进行布局，使得系统的功能按钮出现在自定义标题中的竖直中间
[titleView mas_remakeConstraints:^(MASConstraintMaker *make) {
    make.left.equalTo(@10);
    make.width.equalTo(@70);
    make.top.equalTo(@9);
    make.height.equalTo(@22);
}];
```

需要注意，上面对标题栏的布局进行了重设，这样是为了让系统的3个功能按钮显示在自定义标题栏的中间，但是当用户使用全屏功能进行全屏与非全屏切换时，系统会对标题栏的尺寸进行重新布局，将功能按钮放回原来的位置，为了避免这样的问题，可以监听用户全屏切换事件，退出全屏时，进行重新布局。

    整体效果如下：

![](https://static.oschina.net/uploads/space/2017/0620/144429_4tMR_2340880.png)

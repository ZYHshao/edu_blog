---
title: AppleWatch开发入门五——菜单控件的使用
date: 2015-10-15
categories: Apple Watch开发手记
tags: []
---
## AppleWatch开发入门五——菜单控件的使用

### 一、简介

        菜单也是WatchOS中一个重要的交互方式，限于Watch的屏幕尺寸，若将所有用户交互控件都紧密的排列进展示的UI中，那样难免会使用户操作困难，也会影响界面布局的简洁美观。因此，WatchOS的菜单机制是一层覆盖在屏幕上的交互界面，有如下的特点：

1、菜单是内置于InterfaceController中的，不需显式处理，只需对齐菜单项进行添加设置。

2、菜单最多可以容乃四个选项按钮。

3、通过重按可以呼出和隐藏菜单。

### 二、创建菜单的两种方式

#### 1、通过storyBoard创建

        在storyBoard中，我们可以将一个菜单控件拖入到interfaceController中：

![](http://static.oschina.net/uploads/space/2015/1015/142755_E8SV_2340880.png)

在Menu中可以添加一些item，每个item都可以设置图片和文字：

![](http://static.oschina.net/uploads/space/2015/1015/142904_vpCS_2340880.png)

图片的设置分为，自定义和系统两种，我们可以使用自己的图片作为菜单的图片，也可以使用系统为我们提供的一些图片，系统的图片参数是一个枚举，值如下：

```
public enum WKMenuItemIcon : Int {
    
    case Accept // checkmark
    case Add // '+'
    case Block // circle w/ slash
    case Decline // 'x'
    case Info // 'i'
    case Maybe // '?'
    case More // '...'
    case Mute // speaker w/ slash
    case Pause // pause button
    case Play // play button
    case Repeat // looping arrows
    case Resume // circular arrow
    case Share // share icon
    case Shuffle // swapped arrows
    case Speaker // speaker icon
    case Trash // trash icon
}
```

这些枚举中提供了一些我们常用的功能图标。

菜单按钮的触发方法，我们可以通过拖拽Action的方式来添加，在Xcode7的模拟器中，我们使用command+shift+2可以切换到重按模式，模拟器效果如下：

![](http://static.oschina.net/uploads/space/2015/1015/144535_msxf_2340880.png)

#### 2、通过代码来添加菜单选项

        前面提到过，菜单是内含于InterfaceController中的一个控件，在Interface中我们可以调用一些方法来添加菜单按钮，相关方法如下：

```
    //添加一个菜单按钮，图片自定义
    public func addMenuItemWithImage(image: UIImage, title: String, action: Selector)
    public func addMenuItemWithImageNamed(imageName: String, title: String, action: Selector)
    //添加一个系统图片的按钮
    public func addMenuItemWithItemIcon(itemIcon: WKMenuItemIcon, title: String, action: Selector)
    //清除所有按钮
    public func clearAllMenuItems()
```

示例代码如下：

```
override func awakeWithContext(context: AnyObject?) {
        super.awakeWithContext(context)
      
        self.addMenuItemWithItemIcon(WKMenuItemIcon.Share, title: "分享", action:Selector("share"))
        self.addMenuItemWithItemIcon(WKMenuItemIcon.Decline, title: "取消", action: Selector("cancle"))
        self.addMenuItemWithItemIcon(WKMenuItemIcon.Add, title: "添加", action:Selector("add"))
    }

    func share(){
        print("分享")
    }
    func add(){
        print("add")
    }
    func cancle(){
        
    }
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

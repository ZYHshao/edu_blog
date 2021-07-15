---
title: Tkinter之Menu组件用法
date: 2017-11-02
categories: Python
tags: []
---
## Tkinter之Menu组件用法

    开发工具类桌面应用使用Python、Java这类语言是一种不错的选择，他们的GUI库都可以很好的支持跨平台特性。本系列博客主要总结Tkinter库中提供的UI组件，关于Java的GUI开发，感兴趣的可以在如下系列博客中找到：

[https://my.oschina.net/u/2340880/blog?catalog=5647945&temp=1509528194945](https://my.oschina.net/u/2340880/blog?catalog=5647945&temp=1509528194945)。

    Tkinter中有提供Menu菜单组件中可以添加如下几种组件:

1_动作项：简单的行为按钮，用户点击后会执行相应的方法。

2_子菜单：行为完整的子菜单项。

3_控制按钮：可有选中与非选中状态，用来做开关。

4_单选列表：一组单选按钮。

    为一个窗口添加菜单十分容易，示例代码如下：

```python
root = Tk()
rootMenu = Menu(root)
root.config(menu=rootMenu)

item = Menu(master=rootMenu,postcommand=call,tearoff=1)
rootMenu.add_cascade(menu=item,label="File")
item.add_command(label='new File')
subItem = Menu(item)
subItem.add_command(label="Open in noew window")
item.add_cascade(menu=subItem,label="Open")
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2017/1102/140508_9k3P_2340880.png)

Menu构造函数中第1个参数可以传入菜单所属的窗口或者父菜单，后面可以添加一些菜单配置，例如：

| 属性 | 意义 |
| activebackground | 活跃时的背景色 |
| activeborderwidth | 活跃时的边框宽度 |
| activeforeground | 活跃时的前景色 |
| bg 或者 background | 正常状态背景色 |
| bd 或者 borderwidth | 正常状态变宽宽度 |
| cursor | 鼠标样式 |
| disabledforeground | 无效状态的前景色 |
| font | 菜单字体 |
| fg 或者 foreground | 正常状态的前景色 |
| postcommand | 设置菜单被唤出时的回调 |
| relief | 设置菜单浮雕效果 |
| selectcolor | 设置菜单选中颜色 |
| tearoff | 可以设置为0和1，表示此菜单是否可以独立出来 |
| tearoffcommand | 菜单独立被触发时的回调 |
| title | 可设置独立菜单的标题 |

需要注意，在MacOS系统上，菜单的样式是由系统维护的，上面的大多属性都将没有效果。

    下面这些方法用来进行菜单配置：

```python
#添加一个子菜单 coption为配置选项
add_cascade(coption...)
#添加一个切换按钮 coption为配置选项
add_checkbutton(coption...)
#添加一个功能按钮 coption为配置选项
add_command(coption...)
#添加一个单选按钮 coption为配置选项
add_radiobutton(coption...)
#添加一个分割线
add_separator()
#删除index1 到 index2之间的选项
delete(index1,index2)
#获取菜单某一项的属性值
entrycget(index,coption)
#重新配置菜单中某项的属性
entryconfigure(index,coption...)
#返回参数位置对应的选项索引
index(i)
#在指定位置插入一个子菜单
insert_cascade(index,coption...)
#在指定位置插入一个切换按钮
insert_checkbutton(index,coption...)
#在指定位置插入一个功能按钮
insert_command(index,coption...)
#在指定位置插入一个单选按钮
insert_radiobutton(index,coption...)
#在指定位置插入一个分割线
insert_separator(index)
#代码手动调用一次某个选项
invoke(index)
#在窗口指定位置弹出菜单
post(x,y)
#获取个选项的类型
type(index)
#获取某个选项距离菜单顶部的偏移量
yposition(n)
#添加一个选项 可以是功能按钮，切换按钮，单选按钮或子菜单，由类型确认
#类型可选 cascade checkbutton command radiobutton separator
add(kind,coption)
```

上面列举方法中的coption用来进行一些配置项的设置，可选配置项如下：

| 属性名 | 意义 |
| accelerator | 设置快捷键 |
| activebackground | 激活状态背景色 |
| activeforeground | 激活状态前景色 |
| background | 正常状态背景色 |
| bitmap | 设置bitmap图标 |
| columnbreak | 设置分列 |
| command | 设置激活时的回调函数 |
| compound | 设置展示文本和图标是的布局方式 |
| font | 设置字体 |
| foreground | 设置正常状态的前景色 |
| hidemargin | 设置是否隐藏外边距 设置True或False |
| image | 设置图片 gif格式 |
| label   | 设置显示的文本  |
| menu | 这个选项只用在添加子菜单中 |
| offvalue | 设置checkbutton关闭时的值 |
| onvalue | 设置checkbutton开启时的值 |
| selectcolor | 设置选中状态的颜色 |
| selectimage | 设置选中状态的图像 |
| state    | 设置选项状态，DISABLED或ACTIVE |
| underline  | 设置下划线 |
| value | 选项的值 |
| variable | 用于单选按钮或切换按钮 |

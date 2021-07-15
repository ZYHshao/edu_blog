---
title: Java开发GUI之Menu菜单
date: 2017-06-13
categories: Java
tags: []
---
## Java开发GUI之Menu菜单

    在MacOS上的软件都有一个菜单栏，会浮现在屏幕顶部，Java的awt包中也提供了构建菜单功能的相关组件，示例代码如下：

```javascript
    static void MenuTest(){
        Frame frame = new Frame("Menu");
        //创建菜单栏
        MenuBar menuBar = new MenuBar();
        //设置菜单栏
        frame.setMenuBar(menuBar);
        //创建菜单
        Menu m1 = new Menu("文件", true);
        //向菜单栏中添加菜单
        menuBar.add(m1);
        //创建选项
        MenuItem menuItem1 = new MenuItem("新建");
        MenuItem menuItem2 = new MenuItem("打开");
        //向菜单中添加选项
        m1.add(menuItem1);
        m1.add(menuItem2);
        
        Menu m2 = new Menu("编辑", true);
        menuBar.add(m2);
        MenuItem menuItem3 = new MenuItem("复制");
        MenuItem menuItem4 = new MenuItem("粘贴");
        m2.add(menuItem3);
        m2.add(menuItem4);
        
        Menu m3 = new Menu("帮助", true);
        menuBar.setHelpMenu(m3);
        MenuItem menuItem5 = new MenuItem("问询");
        MenuItem menuItem6 = new MenuItem("联系我们");
        m3.add(menuItem5);
        m3.add(menuItem6);
        
        frame.pack();
        frame.show();
    }
```

效果如下：

![](https://static.oschina.net/uploads/space/2017/0613/133830_mVEP_2340880.png)

MenuBar用来创建菜单栏，其中常用方法如下：

```java
//构造方法
public MenuBar();
//获取帮助菜单
public Menu getHelpMenu();
//设置帮助菜单
public void setHelpMenu(Menu m);
//添加菜单
public Menu add(Menu m);
//根据下标移除菜单
public void remove(int index);
//移除一个菜单
public void remove(MenuComponent m);
//获取菜单栏中菜单个数
public int getMenuCount();
//获取菜单栏中菜单个数 已经弃用 使用上面的方法
public int countMenus();
//根据下标获取菜单对象
public Menu getMenu(int i);
```

Menu类为菜单对象，其中可以添加选项类MenuItem对象，Menu类中常用方法如下：

```java
//构造函数
public Menu();
//label参数设置菜单的标题
public Menu(String label);
//布尔值参数设置是否为tear-off菜单
public Menu(String label, boolean tearOff);
//获取菜单是否为tear-off菜单
public boolean isTearOff();
//获取选项个数
public int getItemCount();
//获取选项个数 已经弃用 使用上面方法
public int countItems();
//获取某个选项对象
public MenuItem getItem(int index);
//添加一个菜单选项
public MenuItem add(MenuItem mi);
//添加一个指定标题的菜单项
public void add(String label);
//插入一个菜单项
public void insert(MenuItem menuitem, int index);
//插入一个指定标题的菜单项
public void insert(String label, int index) ;
//添加分割线
public void addSeparator();
//插入分割线
public void insertSeparator(int index);
//根据下标移除一个选项
public void remove(int index);
//移除一个选项
public void remove(MenuComponent item);
//移除所有选项
public void removeAll() ;

```

下面是MenuItem类的方法解析：

```java
//构造方法
public MenuItem();
//label参数设置选项标题
public MenuItem(String label);
//MenuShortcut为设置快捷键
public MenuItem(String label, MenuShortcut s);
//获取选项标题
public String getLabel() ;
//设置选项标题
public synchronized void setLabel(String label);
//获取选项是否有效
public boolean isEnabled();
//设置选项
public synchronized void setEnabled(boolean b);
//获取快捷键
public MenuShortcut getShortcut();
//设置快捷键
public void setShortcut(MenuShortcut s);
//删除快捷键
public void deleteShortcut();
//添加事件监听
public synchronized void addActionListener(ActionListener l);
//移除事件监听
public synchronized void removeActionListener(ActionListener l);
//获取所有监听者
public synchronized ActionListener[] getActionListeners();

```

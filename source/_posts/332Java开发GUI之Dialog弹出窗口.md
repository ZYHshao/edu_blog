---
title: Java开发GUI之Dialog弹出窗口
date: 2017-06-05
categories: Java
tags: []
---
## Java开发GUI之Dialog弹出窗口

 构造方法：

```java
//创建弹出窗 owner为拥有其的窗口
public Dialog(Frame owner);
//创建弹出窗，modal设置其是否是模态的 如果是模态的 则弹出窗显示时不能操作其他窗口
public Dialog(Frame owner, boolean modal);
//创建弹出窗 title设置弹出窗标题
public Dialog(Frame owner, String title);
//同上
public Dialog(Frame owner, String title, boolean modal);
public Dialog(Frame owner, String title, boolean modal, GraphicsConfiguration gc); 
public Dialog(Dialog owner);
public Dialog(Dialog owner, String title);
public Dialog(Dialog owner, String title, boolean modal);
public Dialog(Dialog owner, String title, boolean modal, GraphicsConfiguration gc);
public Dialog(Window owner);
public Dialog(Window owner, String title);
/*
ModalityType是模式枚举
MODELESS：不覆盖任何窗口
DOCUMENT_MODAL：阻止文档内的所有窗口
APPLICATION_MODAL：阻止应用程序的所有窗口
TOOLKIT_MODAL
*/
public Dialog(Window owner, ModalityType modalityType);
public Dialog(Window owner, String title, ModalityType modalityType);
public Dialog(Window owner, String title, ModalityType modalityType, GraphicsConfiguration gc);
```

其他常用方法：

```java
//获取弹出窗是否是模态的
public boolean isModal();
//设置弹出窗是否为模态窗口
public void setModal(boolean modal);
//获取弹出窗模态类型
public ModalityType getModalityType();
//设置弹出窗模态类型
public void setModalityType(ModalityType type);
//获取弹出窗标题
public String getTitle();
//设置弹出窗标题
public void setTitle(String title);
//设置弹出窗显示或隐藏
public void setVisible(boolean b);
//显示弹出窗 已经弃用 使用setVisible方法
public void show();
//隐藏弹出窗 已经弃用 使用setVisible方法
public void hide();
//获取弹出窗是否尺寸可调整
public boolean isResizable();
//设置弹出窗尺寸是否可调整
public void setResizable(boolean resizable);
//设置弹出窗透明度
public void setOpacity(float opacity);
//设置弹出窗形状
public void setShape(Shape shape);
//设置弹出窗背景色
public void setBackground(Color bgColor);

```

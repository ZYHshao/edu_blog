---
title: Java开发GUI之Label标签
date: 2017-06-07
categories: Java
tags: []
---
## Java开发GUI之Label标签

    Label控件是awt包中最简单的几个视图控件之一，其用来显示固定的文本，示例如下：

```java
Frame frame = new Frame("Label");
Panel pannel = new Panel();
Label label = new Label("刚好遇见你", Label.LEFT);
pannel.add(label);
Label label2 = new Label("如果再相遇", Label.CENTER);
pannel.add(label2);
Label label3 = new Label("我会记得你", Label.RIGHT);
pannel.add(label3);
frame.add(pannel);
frame.pack();
frame.show();
```

Label类中定义了3个静态变量，其中LEFT表示文本左对齐，CENTER表示文本居中对齐，RIGHT表示右对齐。

```java
//初始化方法 默认左对齐
public Label();
public Label(String text);
public Label(String text, int alignment);
//获取文本对齐模式
public int getAlignment(); 
//设置文本对齐模式
public synchronized void setAlignment(int alignment);
//获取标签文本
public String getText();
//设置标签文本
public void setText(String text);

```

![](https://static.oschina.net/uploads/space/2017/0607/161113_aarC_2340880.png)

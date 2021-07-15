---
title: Java开发GUI之FlowLayout流布局
date: 2017-06-18
categories: Java
tags: []
---
## Java开发GUI之FlowLayout流布局

    FlowLayout顾名思义，即流式布局。其默认以行进行布局，可以设置对齐模式，当一行的距离不够组件进行排列时，FlowLayout会自行进行换行。

```java
    static void FlowLayoutTest(){
        Frame frame = new Frame("Flow");
        FlowLayout layout = new FlowLayout(FlowLayout.RIGHT, 30, 20);
        Panel pannel = new Panel(layout);
        pannel.add(new Button("Button1"));
        pannel.add(new Button("Button2"));
        pannel.add(new Button("Button3"));
        pannel.add(new Button("Button4"));
        pannel.add(new Button("Button5"));
        frame.add(pannel);
        frame.pack();
        frame.show();
    }
```

效果如下：

![](https://static.oschina.net/uploads/space/2017/0618/194717_ELC3_2340880.png)

FlowLayout类核心方法结局如下：

```java
//构造方法 默认居中对齐 行列间距为5
public FlowLayout();
//align设置对齐模式
/*
//左对齐
public static final int LEFT        = 0;
//居中对齐
public static final int CENTER      = 1;
//右对齐
public static final int RIGHT       = 2;
*/
public FlowLayout(int align);
//hgap设置水平间距 vgap设置竖直间距
public FlowLayout(int align, int hgap, int vgap);
//获取对齐模式
public int getAlignment();
//设置对齐模式
public void setAlignment(int align);
//获取水平间距
public int getHgap();
//设置水平间距
public void setHgap(int hgap) ;
//获取竖直间距
public int getVgap();
//设置竖直间距
public void setVgap(int vgap) ;
//设置是否基线对齐
public void setAlignOnBaseline(boolean alignOnBaseline) ;
//获取是否基线对齐
public boolean getAlignOnBaseline() ;
```

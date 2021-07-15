---
title: Java开发GUI之GridLayout网格布局
date: 2017-06-18
categories: Java
tags: []
---
## Java开发GUI之GridLayout网格布局

    GridLayout是简单的网格布局，使用其可以方便的实现多行多列的布局样式。

```java
    static void GridLayoutTest(){a
        Frame frame = new Frame("Grid");
        GridLayout layout = new GridLayout(2, 3, 10, 10);
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

![](https://static.oschina.net/uploads/space/2017/0618/205551_tFNu_2340880.png)

GridLayout类中常用方法总结如下：

```java
//构造方法
public GridLayout();
//设置行数与列数
public GridLayout(int rows, int cols);
//设置行数与列数 以及水平竖直间距
public GridLayout(int rows, int cols, int hgap, int vgap);
//获取行数
public int getRows();
//设置行数
public void setRows(int rows);
//获取列数
public int getColumns();
//设置列数
public void setColumns(int cols) ;
//获取水平间距
public int getHgap();
//设置水平间距
public void setHgap(int hgap);
//获取竖直间距
public int getVgap();
//设置竖直间距
public void setVgap(int vgap);
```

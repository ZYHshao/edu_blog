---
title: Java开发GUI之BorderLayout边界布局
date: 2017-06-16
categories: Java
tags: []
---
## Java开发GUI之BorderLayout边界布局

    前面博客中所提及的例子都是针对单独的视图组件，将组件组合并布局在合适的位置才能算是完整的界面。Java中的布局采用布局管理器模式进行，提供了跨平台性，BoaderLayout布局管理器会将其内容分成5个部分，上下左右和中心，示例代码如下：a

```java
    static void BorderLayoutTest(){
        Frame frame = new Frame("Label");
        BorderLayout layout = new BorderLayout(10,15);
        Panel pannel = new Panel(layout);
        pannel.add(BorderLayout.NORTH, new Button("北方"));
        pannel.add(BorderLayout.SOUTH, new Button("南方"));
        pannel.add(BorderLayout.EAST, new Button("东方"));
        pannel.add(BorderLayout.WEST, new Button("西方"));
        pannel.add(BorderLayout.CENTER, new Button("中心"));
        frame.add(pannel);
        frame.pack();
        frame.show();
    }
```

效果如下：

![](https://static.oschina.net/uploads/space/2017/0616/135612_huPw_2340880.png)

BorderLayout中常用方法解析：

```java
//常量 指定布局在北方位置
public static final String NORTH  = "North";
//常量 指定布局在南方位置
public static final String SOUTH  = "South";
//常量 指定布局在东方位置
public static final String EAST   = "East";
//常量 指定布局在西方位置
public static final String WEST   = "West";
//常量 指定布局在中心位置
public static final String CENTER = "Center";
//初始化方法 默认无间距
public BorderLayout();
//初始化方法 hgap设置水平间距 vgap设置垂直间距
public BorderLayout(int hgap, int vgap);
//获取水平间距
public int getHgap();
//设置水平间距
public void setHgap(int hgap);
//获取垂直间距
public int getVgap();
//设置垂直间距
public void setVgap(int vgap);

```

除了上面的方法，布局管理器相关类中都实现了添加组件的方法，这些方法一般开发者是不需要调用到的，当向容器中添加组件时，容器会用其对应的布局管理器来调用这些方法进行布局。

---
title: Java开发GUI之GridBagLayout布局
date: 2017-06-19
categories: Java
tags: []
---
## Java开发GUI之GridBagLayout布局

    GridBagLayout布局管理器是比GridLayout布局更加强大的表格布局。GridLayout进行的表格布局其中元素尺寸相同，而GridBagLayout则可以灵活配置其中元素的尺寸和位置。同样，GridBagLayout的使用也更加复杂，其布局依赖GridBagConstraints类。

    先看如下经典示例：

```java
    static void GridBagLayoutTest(){
        Frame frame = new Frame("GridBag");
        GridBagLayout layout = new GridBagLayout();
        GridBagConstraints constraints = new GridBagConstraints();
        Panel pannel = new Panel(layout);
        constraints.fill = GridBagConstraints.BOTH;
        constraints.weightx = 1.0;
        Button button1 = new Button("Button1");
        layout.setConstraints(button1, constraints);
        pannel.add(button1);
        Button button2 = new Button("Button2");
        layout.setConstraints(button2, constraints);
        pannel.add(button2);
        Button button3 = new Button("Button3");
        layout.setConstraints(button3, constraints);
        pannel.add(button3);
        constraints.gridwidth = GridBagConstraints.REMAINDER;
        Button button4 = new Button("Button4");
        layout.setConstraints(button4, constraints);
        pannel.add(button4);
        constraints.weightx=0;
        Button button5 = new Button("Button5");
        layout.setConstraints(button5, constraints);
        pannel.add(button5);
        constraints.gridwidth = GridBagConstraints.RELATIVE;
        Button button6 = new Button("Button6");
        layout.setConstraints(button6, constraints);
        pannel.add(button6);
        constraints.gridwidth = GridBagConstraints.REMAINDER;
        Button button7 = new Button("Button7");
        layout.setConstraints(button7, constraints);
        pannel.add(button7);
        constraints.gridwidth=1;
        constraints.gridheight=2;
        constraints.weighty=1.0;
        Button button8 = new Button("Button8");
        layout.setConstraints(button8, constraints);
        pannel.add(button8);
        constraints.weighty=0;
        constraints.gridwidth=GridBagConstraints.REMAINDER;
        constraints.gridheight = 1;
        Button button9 = new Button("Button9");
        layout.setConstraints(button9, constraints);
        pannel.add(button9);
        Button button10 = new Button("Button10");
        layout.setConstraints(button10, constraints);
        pannel.add(button10);
        frame.add(pannel);
        frame.pack();
        frame.show();
    }
```

上面代码布局了10个按钮，其中复用了同一个GridBagConstraints对象，效果如下：

![](https://static.oschina.net/uploads/space/2017/0619/144005_kBPa_2340880.png)

GridBagLayout类中常用方法列举如下：

```java
//构造方法
public GridBagLayout ();
//设置组件的布局
public void setConstraints(Component comp, GridBagConstraints constraints);
//获取某个组件的布局对象
public GridBagConstraints getConstraints(Component comp);
//获取布局原点
public Point getLayoutOrigin ();
```

在GridBagLayout中其着至关重要作用的类是GridBagConstraints布局类，其精确确定每个子组件的位置和尺寸信息。下面我们来一点点介绍这个类中属性的意义：

fill：这个属性确定当被布局组件尺寸小于其被指定的表格尺寸时，组件的拉伸模式，可选值定义在GridBagConstraints类中，如下：

```java
//不进行尺寸处理 默认居中
public static final int NONE = 0;
//水平和竖直均拉伸到充满
public static final int BOTH = 1;
//水平方向拉伸充满
public static final int HORIZONTAL = 2;
//竖直方向拉伸充满
public static final int VERTICAL = 3;

```

anchor：这个属性确定当被布局组件尺寸小于其被指定的表格尺寸时，组件的布局位置，可选值如下：

```java
//居中
public static final int CENTER = 10;
//布局在上方
public static final int NORTH = 11;
//布局在右上方
public static final int NORTHEAST = 12;
//布局在右方
public static final int EAST = 13;
//布局在右下方
public static final int SOUTHEAST = 14;
//布局在下方
public static final int SOUTH = 15;
//布局在左下方
public static final int SOUTHWEST = 16;
//布局在左方
public static final int WEST = 17;
//布局在左上方
public static final int NORTHWEST = 18;
```

gridwidth与gridheight：这两个属性分别设置组件的宽度与高度，他们可以设置为固定的数值，也可以设置为下面几个特殊的值来表示特殊的意义：

```java
//占据其他组件布局后余下的尺寸
public static final int RELATIVE = -1;
//暂居此行或者此列的剩下全部，后置的组件另起一行或一列
public static final int REMAINDER = 0;
```

gridx与gridy：这两个值设置组件布局左上角所在的单元格，单位为单元格，默认会排列在上一个单元格之后。

weightx与weighty：这两个值设置组件布局的水平权重和竖直权重。

insets：设置组件边距。

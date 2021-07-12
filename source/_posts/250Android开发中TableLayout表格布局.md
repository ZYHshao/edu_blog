---
title: Android开发中TableLayout表格布局
date: 2016-09-05
categories: Android小记
tags: []
---
## Android开发中TableLayout表格布局

### 一、引言

        在移动端应用程序开发中，常常会使用到表格布局，iOS和Android开发框架中都提供了独立的表格视图控件供开发者使用，例如iOS中的UITableView、UICollectionView，Android中的ListView、GridView等。除了独立的视图控件外，Android中还提供了一个布局容器类TableLayout，使用其也可以进行方便的表格布局。

        前边博客有介绍过关于LinearLayout线性布局的相关内容，LinearLayout只能进行水平或者垂直方向上的排列布局，使用LinearLayout的布局嵌套，实际上也可以实现表格布局的样式。实际上，TableLayout就是采用这样的原理，TableLayout继承于LinearLayout，其中每个视图元素作为一行，同时Android中还提供了一个TableRow类，这个类同样继承自LinearLayout，其中每个视图元素作为当前行中的一列，结合使用TableLayout与TableRow，就实现了行列的表格布局。

### 二、关于TableRow

        TableRow可以简单理解为TableLayout布局中的一行，当然，TableLayout中也可以直接添加任意的View视图，但是默认添加的View视图将独占一行。TableRow中可以添加其他视图，每个视图被作为一列处理，通过TableRow的内部类LayoutParams来设置TableRow内部视图的布局方式，其中主要可以通过设置宽高或者设置权重来定制每列视图元素的尺寸，例如：

```java
TableLayout tableLayout = new TableLayout(this);
//创建行 第一行用单个元素
TextView textView = new TextView(this);
textView.setText("1000");
textView.setTextSize(20);
textView.setTextAlignment(View.TEXT_ALIGNMENT_VIEW_END);
tableLayout.addView(textView);
//第二行使用TableRow
TableRow tableRow1 = new TableRow(this);
//设置本行中每一列的权重和
tableRow1.setWeightSum(10);
Button button11 = new Button(this);
button11.setText("AC");
//设置固定宽高
TableRow.LayoutParams layoutParams1 = new TableRow.LayoutParams(300,200);
button11.setLayoutParams(layoutParams1);
tableRow1.addView(button11);
Button button12 = new Button(this);
//通过权重设置列的宽度 占正常列宽的一半
TableRow.LayoutParams layoutParams2 = new TableRow.LayoutParams(0,200,5);
button12.setLayoutParams(layoutParams2);
button12.setText("+/-");
tableRow1.addView(button12);
Button button13 = new Button(this);
button13.setText("%");
tableRow1.addView(button13);
Button button14 = new Button(this);
button14.setText("÷");
tableRow1.addView(button14);
tableLayout.addView(tableRow1);

```

上面代码向TableRow中添加了4个视图，默认情况下会生成四列，setWeightSum()方法用于设置每列的权重和，需要注意，它作用的对象是每一列元素，而不是整行。上面的代码效果如下：

![](http://static.oschina.net/uploads/space/2016/0905/152138_VKXr_2340880.png)

默认的列宽是评分整个行宽，可以通过指定宽度或者权重来修改特定列的列宽。

        还有一点需要注意，如果一个TableLayout布局中多个TableRow，则表格的列数会以最多列的一行为准，例如在添加一行TableRow，而其中只有一列，则其依然会预留4列的位置，示例如下：

```java
TableRow tableRow2 = new TableRow(this);
Button button = new Button(this);
button.setText("跳过");
tableRow2.addView(button);
tableLayout.addView(tableRow2);
```

效果如下：

![](http://static.oschina.net/uploads/space/2016/0905/153213_3XOj_2340880.png)

也可以设置跳过某列进行布局，或者进行列的合并，示例如下：

```java
TableRow tableRow2 = new TableRow(this);
Button button = new Button(this);
button.setText("跳过");
TableRow.LayoutParams layoutParams21 = new TableRow.LayoutParams();
//从第2列开始
layoutParams21.column = 1;
//合并3列
layoutParams21.span = 3;
button.setLayoutParams(layoutParams21);
tableRow2.addView(button);
tableLayout.addView(tableRow2);
```

![](http://static.oschina.net/uploads/space/2016/0905/161251_j3bz_2340880.png)

### 三、关于TableLayout

        在向TableLayout容器中添加或者移除视图的时候，开发者可以对其进行监听，示例如下：

```java
TableLayout tableLayout = new TableLayout(this);
tableLayout.setOnHierarchyChangeListener(new ViewGroup.OnHierarchyChangeListener() {
    @Override
    public void onChildViewAdded(View parent, View child) {
        Toast.makeText(getBaseContext(),"add",Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onChildViewRemoved(View parent, View child) {
        Toast.makeText(getBaseContext(),"remove",Toast.LENGTH_SHORT).show();
    }
});
```

开发者还可以对表格中视图元素的一些尺寸自适应做一些设置，其中还有一些常用的方法列举如下：

```java
//获取表格中所有列是否是可收缩的
public boolean isShrinkAllColumns()
//设置表格中的所有列是否可收缩
public void setShrinkAllColumns()
//获取表格中的所有列是否可拉伸
public boolean isStretchAllColumns()
//设置表格中的所有列是否可拉伸
public void setStretchAllColumns()
//设置某一列是否可拉伸
public void setColumnStretchable(int columnIndex, boolean isStretchable)
//获取某一列是否可拉伸
public boolean isColumnStretchable(int columnIndex)
//设置某一列是否可收缩
public void setColumnShrinkable(int columnIndex, boolean isShrinkable)
//获取某一列是否可收缩
public boolean isColumnShrinkable(int columnIndex)
```

所谓可收缩的列，是指如果此列的内容宽度超出一定宽度，为了使后面的列内容展示出来，此列宽度会自动收缩，高度会增加，如下图所示：

![](http://static.oschina.net/uploads/space/2016/0905/171556_XW6I_2340880.png)

至于可拉伸的列，是指如果此行内容内有充满整行，此列会进行拉伸自动充满。

        下面这些方法与表格中列的隐藏有关：

```java
//设置某列是否隐藏
public void setColumnCollapsed(int columnIndex, boolean isCollapsed)
//获取某列是否被隐藏
public boolean isColumnCollapsed(int columnIndex)
```

需要注意，在TableLayout中也定义了一个LayoutParams的内部类，其用于设置其中每一行视图元素的布局，但是开发者只能设置此布局类对应的高度参数，宽度将强制设置为MATCH_PARENT。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：435043639

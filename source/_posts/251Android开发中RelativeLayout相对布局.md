---
title: Android开发中RelativeLayout相对布局
date: 2016-09-06
categories: Android小记
tags: []
---
## Android开发中RelativeLayout相对布局

        RelativeLayout布局是Android界面布局中应用最广也最强大的一种布局，其不仅十分灵活，可以解决开发中各种界面布局需求，同时也很方便了解决了多屏幕尺寸的适配问题。在iOS开发中，Autolayout技术总是被赞不绝口，RelativeLayout布局就是Andriod系统中的Autolayout，其又被称为相对布局。

        所谓相对布局，是指其坐标的确定并不是开发者写死的，而是有系统自动计算出来的，那么系统如何计算每个视图控件的位置呢？开发者需要为其添加一些规则进行约束，这些规则大致包括2类：

第1类 与父视图之间位置关系的规则：

        此类规则包括在父视图中的居中、左对齐、右对齐、上对齐、下对齐等。

第2类 平级视图之间相对位置关系的规则：

        此类规则包括同级视图间对其关系，相对位置关系，例如A在B左侧20像素位置，B与C上边缘对齐等。

使用RelativeLayout进行布局示例代码如下：

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        RelativeLayout relativeLayout = new RelativeLayout(this);
        Button button1 = new Button(this);
        button1.setText("按钮一");
        button1.setId(R.id.button1);
        RelativeLayout.LayoutParams layoutParams1 = new RelativeLayout.LayoutParams(200,200);
        //添加约束 使其靠近父视图右上角
        layoutParams1.addRule(RelativeLayout.ALIGN_PARENT_TOP);
        layoutParams1.addRule(RelativeLayout.ALIGN_PARENT_RIGHT);
        button1.setLayoutParams(layoutParams1);

        Button button2 = new Button(this);
        button2.setText("按钮二");
        button2.setId(R.id.button2);
        RelativeLayout.LayoutParams layoutParams2 = new RelativeLayout.LayoutParams(200,200);
        //添加约束 让其右侧靠近按钮一左侧 上侧靠近按钮一下侧
        layoutParams2.addRule(RelativeLayout.BELOW,R.id.button1);
        layoutParams2.addRule(RelativeLayout.LEFT_OF,R.id.button1);
        button2.setLayoutParams(layoutParams2);

        Button button3 = new Button(this);
        button3.setText("按钮三");
        button3.setId(R.id.button3);
        RelativeLayout.LayoutParams layoutParams3 = new RelativeLayout.LayoutParams(200,200);
        //添加约束 让其固定距离按钮二 下方100px 左侧边缘对其
        layoutParams3.addRule(RelativeLayout.ALIGN_LEFT,R.id.button2);
        layoutParams3.addRule(RelativeLayout.BELOW,R.id.button2);
        layoutParams3.topMargin = 100;
        button3.setLayoutParams(layoutParams3);

        relativeLayout.addView(button1);
        relativeLayout.addView(button2);
        relativeLayout.addView(button3);
        setContentView(relativeLayout);
    }
```

小提示：使用代码创建的视图，可以通过xml文件配置id，如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <item name="button1" type="id"></item>
    <item name="button2" type="id"></item>
    <item name="button3" type="id"></item>
</resources>
```

效果如下图：

![](http://static.oschina.net/uploads/space/2016/0906/154309_QJcF_2340880.png)

RelativeLayout布局中视图位置的配置主要使用其内部类LayoutParams，这个内部类LayoutParams是继承自MarginLayoutParams。其中常用方法和属性列举如下：

```java
//设置左边距
public int leftMargin;
//设置上边距
public int topMargin;
//设置右边距
public int rightMargin;
//设置下边距
public int bottomMargin;

//添加一个规则 这个方法添加的规则不需要参照视图 例如靠近父视图边缘
public void addRule(int verb)
//添加一个规则 这个方法添加的规则需要一个参照视图 例如某两个平级视图间的位置关系 anchor参数为视图id
public void addRule(int verb, int anchor) 
//移除一个布局规则
public void removeRule(int verb)
```

用于进行布局规则配置的参数如下：

```java
/*=======需要使用addRule(int verb, int anchor)方法添加的约束规则==========*/
//将当前视图约束到某个视图左边
public static final int LEFT_OF
//将当前视图约束到某个视图右边
public static final int RIGHT_OF
//将当前视图约束到某个视图上边
public static final int ABOVE
//将当前视图约束到某个视图下边
public static final int BELOW
//将当前视图约束与某个视图基线对齐
public static final int ALIGN_BASELINE
//将当前视图约束与某个视图左侧对齐
public static final int ALIGN_LEFT
//将当前视图约束与某个视图上侧对齐
public static final int ALIGN_TOP
//将当前视图约束与某个视图右侧对齐
public static final int ALIGN_RIGHT
//将当前视图约束与某个视图下侧对齐
public static final int ALIGN_BOTTOM
//将当前视图约束与某个视图起始对齐
public static final int START_OF
//当当前视图约束与某个视图末尾对齐
public static final int END_OF

/*========需要使用addRule(int verb)方法添加的约束规则====================*/
//约束当前视图与父视图左侧对齐
public static final int ALIGN_PARENT_LEFT
//约束当前视图与父视图上侧对齐
public static final int ALIGN_PARENT_TOP
//约束当前视图与父视图上侧对齐
public static final int ALIGN_PARENT_RIGHT
//约束当前视图与父视图下侧对齐
public static final int ALIGN_PARENT_BOTTOM
//约束当前视图与父视图居中对齐
public static final int CENTER_IN_PARENT
//约束当前视图与父视图水平居中
public static final int CENTER_HORIZONTAL
//约束当前视图与父视图垂直居中
public static final int CENTER_VERTICAL
//约束当前视图与父视图起始对齐
public static final int ALIGN_PARENT_START
//约束当前视图与父视图末尾对齐
public static final int ALIGN_PARENT_END
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：435043639

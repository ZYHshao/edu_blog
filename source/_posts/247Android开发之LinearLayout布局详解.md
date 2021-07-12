---
title: Android开发之LinearLayout布局详解
date: 2016-08-31
categories: Android小记
tags: []
---
## Android开发之LinearLayout布局详解

        LinaerLayout又被称为线性布局，是Android界面开发中常用的一种容器视图控件。可以使用XML布局文件配置和代码动态创建两种方式来使用LinearLayout。使用LinearLayout可以十分轻松的布局出横向或者纵向线性堆叠界面，并且，嵌套使用LinearLayout也可以方便的布局出复杂的平面组合布局，通常情况下，ScrollView会与LinearLayout进行结合使用。在iOS9中推出的UIStackView、在watchOS开发中使用和核心布局模型Group与LinearLayout的思路十分一致，可见这种线性堆叠的布局方式在一定场景下十分有优势。

        使用代码动态创建LinearLayout示例如下：

```java
 @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //创建对象
        LinearLayout linearLayout = new LinearLayout(this);
        linearLayout.setBackgroundColor(Color.WHITE);
        setContentView(linearLayout);
        //设置布局方向 
        /*
          支持水平和竖直两种
          public static final int HORIZONTAL = 0;  水平线性布局
          public static final int VERTICAL = 1;    竖直线性布局
        */
        linearLayout.setOrientation(LinearLayout.VERTICAL);
        //设置显示分割线的模式
        /*
          public static final int SHOW_DIVIDER_NONE = 0; 不显示分割线
          public static final int SHOW_DIVIDER_BEGINNING = 1;  在开始处显示分割线
          public static final int SHOW_DIVIDER_MIDDLE = 2;  在子视图之间显示分割线
          public static final int SHOW_DIVIDER_END = 4;   在结束尾部显示分割线
        */
        linearLayout.setShowDividers(LinearLayout.SHOW_DIVIDER_MIDDLE);
        //设置分割线Drawable
        linearLayout.setDividerDrawable(ResourcesCompat.getDrawable(getResources(),R.drawable.line,null));
    }
```

LinearLayout中常用属性与方法，列举如下：

```java
//获取分割线Drawable对象
Drawable getDividerDrawable ()
//获取分割线的padding值
int getDividerPadding ()
//获取子视图布局模式
int getGravity ()
//获取线性布局方向
int getOrientation ()
//获取展示分割线模式
int getShowDividers ()
//获取布局权重和
float getWeightSum ()
//设置是否允许计量最大子元素 与权重有关
boolean isMeasureWithLargestChildEnabled ()
//设置分割线Drawable
void setDividerDrawable (Drawable divider)
//设置分割线padding值
void setDividerPadding (int padding)
//设置子视图布局模式
/*
可选参数
AXIS_CLIP   //原始对齐
AXIS_PULL_AFTER  
AXIS_PULL_BEFORE
AXIS_SPECIFIED
AXIS_X_SHIFT
AXIS_Y_SHIFT
BOTTOM   //下对齐
CENTER   //居中对齐
CENTER_HORIZONTAL // 水平居中对齐
CENTER_VERTICAL   // 竖直居中对齐
CLIP_HORIZONTAL
CLIP_VERTICAL
DISPLAY_CLIP_HORIZONTAL
DISPLAY_CLIP_VERTICAL
END  //末尾对齐
FILL //充满
FILL_HORIZONTAL //水平充满
FILL_VERTICAL   //竖直充满
HORIZONTAL_GRAVITY_MASK
LEFT   //左对齐
NO_GRAVITY //空模式
RELATIVE_HORIZONTAL_GRAVITY_MASK
RELATIVE_LAYOUT_DIRECTION
RIGHT  //右对齐
START  //起始对齐
TOP   //上对齐
VERTICAL_GRAVITY_MASK
*/
void setGravity (int gravity)
//设置水平布局模式
void setHorizontalGravity (int horizontalGravity)
//设置布局方向
void setOrientation (int orientation)
//设置竖直布局模式
void setVerticalGravity (int verticalGravity)
//设置布局权重和
/*
当布局容器内子视图是通过权重来计算所占比例时 这个值表示权重总和
*/
void setWeightSum (float weightSum)
//设置子视图的触摸事件是否延迟执行
/*
这个属性用于类型ScrollView，ListView可以滑动的视图中，避免手势冲突
*/
boolean shouldDelayChildPressedState ()
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：435043639

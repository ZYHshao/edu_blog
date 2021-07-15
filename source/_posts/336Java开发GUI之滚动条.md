---
title: Java开发GUI之滚动条
date: 2017-06-14
categories: Java
tags: []
---
## Java开发GUI之滚动条

    滚动条组件可以实现用户拖动调整效果，示例代码如下：

```java
    static void ScrollBarTest(){
        Frame frame = new Frame("Label");
        Panel pannel = new Panel();
        Scrollbar scrollbar = new Scrollbar(Scrollbar.HORIZONTAL, 5, 2, 0, 20);
        scrollbar.setUnitIncrement(1);
        scrollbar.setBlockIncrement(5);
        scrollbar.addAdjustmentListener(new ScrollBarListener());
        pannel.add(scrollbar);
        frame.add(pannel);
        frame.pack();
        frame.show();
    }
```

AdjustmentListener用来监听滚动条的值变化，其实现如下：

```java
class ScrollBarListener implements AdjustmentListener{

    @Override
    public void adjustmentValueChanged(AdjustmentEvent e) {
        // TODO Auto-generated method stub
        System.out.println(e.getValue());
    }
    
}
```

ScrollBar组件中常用方法列举如下：

```java
//构造方法
public Scrollbar();
//orientation参数设置滚动条的滚动防线
/*
public static final int     HORIZONTAL = 0; //水平滚动
public static final int     VERTICAL   = 1; //竖直滚动
*/
public Scrollbar(int orientation);
/*
value:滚动条的初始值
visible:滚动条的可见值(比例，确定滑块的尺寸)
minimum:滚动条的最小值
maximum:滚动条的最大值
*/
public Scrollbar(int orientation, int value, int visible, int minimum, int maximum);
//获取滚动条方向
public int getOrientation();
//设置滚动条方向
public void setOrientation(int orientation);
//获取滚动条当前值
public int getValue();
//设置滚动条当前值
public void setValue(int newValue);
//获取滚动条最小值
public int getMinimum();
//设置滚动条最小值
public void setMinimum(int newMinimum);
//获取滚动条最大值
public int getMaximum();
//设置滚动条最大值
public void setMaximum(int newMaximum);
//获取滚动条可见值
public int getVisibleAmount();
//获取滚动条可见值 已经弃用，使用上面方法
public int getVisible();
//设置滚动条可见值
public void setVisibleAmount(int newAmount);
//设置单元增值
public void setUnitIncrement(int v);
//获取单元增值
public int getUnitIncrement();
//设置块增值
public void setBlockIncrement(int v);
//获取块增值
public int getBlockIncrement() ;
//设置 当前值 可见值 最小值和最大值
public void setValues(int value, int visible, int minimum, int maximum);
//获取值是否正在调节
public boolean getValueIsAdjusting();
//设置值是否正在调节
public void setValueIsAdjusting(boolean b);
//设置滚动条值变化的监听
public synchronized void addAdjustmentListener(AdjustmentListener l) ;
//移除监听
public synchronized void removeAdjustmentListener(AdjustmentListener l);
//获取所有监听者
public synchronized AdjustmentListener[] getAdjustmentListeners();
```

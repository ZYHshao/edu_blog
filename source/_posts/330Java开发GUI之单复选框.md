---
title: Java开发GUI之单复选框
date: 2017-06-01
categories: Java
tags: []
---
## Java开发GUI之单复选框

    单复选框在处理一些用户选项时十分方便。在Java的GUI体系中，复选框使用Checkbox类来创建，单选框实际上是将多个复选框结合成为组，同一组的复选框同时只能有一个被选中。

```java
        Frame frame = new Frame("BUTTON");
        Panel pannel = new Panel();
        frame.add(pannel);
        MyItemListener listener = new MyItemListener();
        //无参的构造方法
        Checkbox checkbox1 = new Checkbox();
        //设置显示文本
        checkbox1.setLabel("是否已退款");
        //设置选中状态
        checkbox1.setState(true);
        //添加状态变化监听
        checkbox1.addItemListener(listener);
        //使用设置文本和状态的构造方法
        Checkbox checkbox2 = new Checkbox("是否开通额外服务", false);
        checkbox2.addItemListener(listener);
        pannel.add(checkbox1);
        pannel.add(checkbox2);
        //创建组
        CheckboxGroup group = new CheckboxGroup();
        Checkbox checkbox3 = new Checkbox("男", false);
        checkbox3.addItemListener(listener);
        checkbox3.setCheckboxGroup(group);
        Checkbox checkbox4 = new Checkbox("女", false, group);
        checkbox4.addItemListener(listener);
        pannel.add(checkbox3);
        pannel.add(checkbox4);
        frame.pack();
        frame.show();
```

MyItemListener实现了ItemListener接口，如下：

```java
class MyItemListener implements ItemListener{

    @Override
    public void itemStateChanged(ItemEvent e) {
        // TODO Auto-generated method stub
        System.out.println(((Checkbox)e.getSource()).getState());
    }
    
}
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2017/0601/160630_ANP2_2340880.png)

Checkbox类常用方法汇总：

```java
//获取标题文本
public String getLabel();
//设置标题
public void setLabel(String label);
//获取当前复选框的选中状态
public boolean getState();
//设置当前复选框的选中状态
public void setState(boolean state);
//获取复选框所在的组
public CheckboxGroup getCheckboxGroup();
//设置复选框组
public void setCheckboxGroup(CheckboxGroup g);
//添加状态监听对象
public synchronized void addItemListener(ItemListener l);
//移除状态监听对象
public synchronized void removeItemListener(ItemListener l);
//获取状态监听对象
public synchronized ItemListener[] getItemListeners();
public <T extends EventListener> T[] getListeners(Class<T> listenerType);
```

CheckboxGroup常用方法汇总：

```java
//获取组中当前选中的checkbox
public Checkbox getSelectedCheckbox();
//设置当前选中的checkbox
public void setSelectedCheckbox(Checkbox box);
```

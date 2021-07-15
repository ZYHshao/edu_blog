---
title: Java开发GUI之选择列表
date: 2017-06-02
categories: Java
tags: []
---
## Java开发GUI之选择列表

    选择列表在多个选项供用户进行选择的场景中使用广泛。其使用也非常简单，Java的awt包中提供了Choice控件，示例代码如下：

```java
    public static Label label = new Label();
    static void choseTest(){
        Frame frame = new Frame("BUTTON");
        Panel pannel = new Panel();
        //创建选择列表
        Choice choice = new Choice();
        //添加选项
        choice.add("鸣人");
        choice.addItem("佐助");
        choice.insert("卡卡西", 0);
        //添加用户选择改变的监听
        choice.addItemListener(new MyItemListener());
        pannel.add(choice);
        label.setText(choice.getSelectedItem()+"一定可以成为最NB的火影！");
        pannel.add(label);
        frame.add(pannel);
        frame.pack();
        frame.show();
    }
```

MyItemListener类实现如下：

```java
class MyItemListener implements ItemListener{

    @Override
    public void itemStateChanged(ItemEvent e) {
        // TODO Auto-generated method stub
        if (e.getSource().getClass()==Choice.class) {
            ButtonTest.label.setText(e.getItem()+"一定可以成为最NB的火影！");
        }else{
            System.out.println(((Checkbox)e.getSource()).getState());
        }
    }
    
}
```

运行效果如下：

![](https://static.oschina.net/uploads/space/2017/0602/160617_LKur_2340880.png)

Choice类解析如下：

```java
//获取选项个数
public int getItemCount();
//获取某个选项
public String getItem(int index);
//追加一个选项
public void add(String item);
public void addItem(String item);
//插入一个选项
public void insert(String item, int index);
//通过标题删除一个选项
public void remove(String item);
//通过位置删除一个选项
public void remove(int position);
//删除所有选项
public void removeAll();
//获取当前选中的选项标题
public synchronized String getSelectedItem();
//获取当前选中的选项位置
public int getSelectedIndex();
//用代码选中某个位置的选项
public synchronized void select(int pos);
//用代码选中某个标题的选项
public synchronized void select(String str);
//添加用户选择监听
public synchronized void addItemListener(ItemListener l);
//移除监听
public synchronized void removeItemListener(ItemListener l);
//获取所有监听对象
public synchronized ItemListener[] getItemListeners();
public <T extends EventListener> T[] getListeners(Class<T> listenerType);
```

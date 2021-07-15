---
title: Java开发GUI之列表
date: 2017-06-09
categories: Java
tags: []
---
## Java开发GUI之列表

    awt包中的List控件可以创建一个选择列表，此列表可以支持单选，也可以支持多选。

```java
    static void ListTest(){
        Frame frame = new Frame("List");
        Panel pannel = new Panel();
        List list = new List();
        //向列表中添加选项
        list.add("鸣人");
        list.add("佐助");
        list.add("卡卡西");
        list.add("小樱");
        list.add("釉");
        list.add("大蛇丸");
        //设置允许多选
        list.setMultipleSelections(true);
        //添加列表选项切换的监听
        list.addItemListener(new ListListener());
        //添加列表行为的监听 例如双击某项
        list.addActionListener(new ListListener());
        pannel.add(list);
        frame.add(pannel);
        frame.pack();
        frame.show();
    }
```

ListListener类的简单实现如下：

```java
class ListListener implements ActionListener,ItemListener{

    @Override
    public void itemStateChanged(ItemEvent e) {
        // TODO Auto-generated method stub
        System.out.println(e.getSource());
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        // TODO Auto-generated method stub
        System.out.println(e);
        System.out.println("======");
    }
    
}
```

![](https://static.oschina.net/uploads/space/2017/0609/141954_b0OW_2340880.png)

List控件中的方法解析：

```java
//构造方法
public List();
//rows:设置行数
public List(int rows);
//设置行数与是否支持多选 mac上使用command可以多选
public List(int rows, boolean multipleMode);
//获取列表中选项个数
public int getItemCount();
//获取列表中选项个数 已经弃用 使用上面的方法
public int countItems();
//获取某个位置的选项
public String getItem(int index);
//获取所有选项
public synchronized String[] getItems();
//添加一个选项
public void add(String item);
//添加一个选项 已经弃用 使用上面的方法
public void addItem(String item);
//在指定位置添加一个选项
public void add(String item, int index);
//在指定位置添加一个选项 已经弃用 使用上面的方法
public synchronized void addItem(String item, int index);
//替换一个位置的选项
public synchronized void replaceItem(String newValue, int index);
//删除所有选项
public void removeAll();
//删除所有选项 已经弃用 使用上面的方法
public synchronized void clear();
//删除某个选项
public synchronized void remove(String item);
//移除某个位置的选项
public void remove(int position);
//删除某个位置的选项 已经弃用 使用上面方法
public void delItem(int position);
//获取选中的选项的位置
public synchronized int getSelectedIndex();
//获取所有选中的选项的位置
public synchronized int[] getSelectedIndexes();
//获取选中的选项
public synchronized String getSelectedItem();
//获取所有选中的选项
public synchronized String[] getSelectedItems();
//获取所有选中的选项
public Object[] getSelectedObjects();
//代码选中某个位置的选项
public void select(int index);
//代码取消某个选项的选中
public synchronized void deselect(int index);
//判断某个位置的选项是否选中
public boolean isIndexSelected(int index);
//判断某个位置的选项是否选中 已经弃用 使用上面方法
public boolean isSelected(int index);
//获取行数
public int getRows();
//获取是否可多选
public boolean isMultipleMode();
//获取是否允许多选 已经弃用 使用上面方法
public boolean allowsMultipleSelections();
//设置是否允许多选
public void setMultipleMode(boolean b);
//设置是否允许多选 已经弃用 使用上面方法
public synchronized void setMultipleSelections(boolean b);
//添加选项选中状态监听
public synchronized void addItemListener(ItemListener l);
//移除选项选中状态监听
public synchronized void removeItemListener(ItemListener l);
//获取监听者
public synchronized ItemListener[] getItemListeners();
//添加用户行为监听
public synchronized void addActionListener(ActionListener l);
//移除用户行为监听
public synchronized void removeActionListener(ActionListener l);
//获取监听
public synchronized ActionListener[] getActionListeners();
```

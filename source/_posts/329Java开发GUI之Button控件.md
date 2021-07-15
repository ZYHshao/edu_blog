---
title: Java开发GUI之Button控件
date: 2017-05-26
categories: Java
tags: []
---
## Java开发GUI之Button控件

    Java中的awt包提供了丰富的用户界面组件。重要的是，Java的跨平台性使用awt包可以在Windows，MacOS等平台创建桌面软件。本篇博客总结Button控件的简单使用。

```java
package App;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
public class ButtonTest {
    public static void main(String[] args ) {
        //创建面板
        Frame frame = new Frame("BUTTON");
        //创建按钮
        Button button1 = new Button();
        //设置按钮标题
        button1.setLabel("按钮");
        //设置按钮标记 用在触发方法中分区按钮
        button1.setActionCommand("tag1");
        //获取按钮标题
        System.out.println(button1.getLabel());
        //添加按钮
        frame.add(button1);
        //添加交互监听
        button1.addActionListener(new MyActionListener());
        frame.pack();
        frame.show();    
    }
}
class MyActionListener implements ActionListener{
    @Override
    public void actionPerformed(ActionEvent e) {
        // TODO Auto-generated method stub
        //这里可以取到按钮设置的actionCommand值
        System.out.println(e.getActionCommand());
    }
}
```

与addActionCommand方法对应，Button类中还有一些移除监听与获取监听者的方法，如下：

```java
//移除一个监听者
public synchronized void removeActionListener(ActionListener l);
//获取监听者列表
public synchronized ActionListener[] getActionListeners();
//获取监听者列表 泛型数组
public <T extends EventListener> T[] getListeners(Class<T> listenerType);
```

ActionEvent类中定义了交互行为的一些特征，示例如下：

```java
class MyActionListener implements ActionListener{
    @Override
    public void actionPerformed(ActionEvent e) {
        // TODO Auto-generated method stub
        //获取触发时间
        System.out.println(e.getWhen());
        //获取触发模式
        System.out.println(e.getModifiers());
        //获取触发事件的控件 此处为Button按钮
        System.out.println(e.getSource());
        //获取关联的actionCommand值
        System.out.println(e.getActionCommand());
    }
}
```

getModifiers方法获取事件的模式，返回值定义如下：

```java
//按住shift键点击按钮
public static final int SHIFT_MASK          = Event.SHIFT_MASK;
//按住ctrl键点击按钮
public static final int CTRL_MASK           = Event.CTRL_MASK;
//按住meta键点击按钮
public static final int META_MASK           = Event.META_MASK;
//按住alt键点击按钮
public static final int ALT_MASK            = Event.ALT_MASK;
```

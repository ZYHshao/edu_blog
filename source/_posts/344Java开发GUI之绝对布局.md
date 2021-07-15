---
title: Java开发GUI之绝对布局
date: 2017-06-20
categories: Java
tags: []
---
## Java开发GUI之绝对布局

    前面多篇博客介绍了Java的awt包中的布局管理类，当然也可以不使用任何布局管理类，开发者可以直接设置组件的坐标和尺寸，示例代码如下：

```java
    static void AbsoluteLayoutTest(){
        Frame frame = new Frame("Grid");
        //设置不使用任何布局管理类
        Panel pannel = new Panel(null);
        Button button1 = new Button("Button1");
        pannel.add(button1);
        button1.reshape(10, 10, 60, 20);
        Button button2 = new Button("Button2");
        pannel.add(button2);
        button2.reshape(80, 40, 60, 20);
        Button button3 = new Button("Button3");
        pannel.add(button3);
        button3.reshape(10, 70, 60, 20);
    
        frame.add(pannel);
        frame.pack();
        frame.show();
    }
```

组件的reshape方法用于重设组件的位置和尺寸，其前两个参数确定x，y坐标，后两个参数确定宽度与高度，效果如下图：

![](https://static.oschina.net/uploads/space/2017/0620/165407_CwJM_2340880.png)

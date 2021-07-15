---
title: Java开发GUI之CardLayout卡片布局
date: 2017-06-18
categories: Java
tags: []
---
## Java开发GUI之CardLayout卡片布局

    CardLayout布局允许进行多套界面的设计，通过切换界面来实现布局样式的改变。CardLayout类似与一叠卡片，默认最先添加的在前面，界面始终只展示一个卡片。示例如下：

```java
    static Panel cardPannel;
    
    static void CardLayoutTest(){
        Frame frame = new Frame("Label");
        Panel top = new Panel();
        Choice choice = new Choice();
        choice.add("BUTTON");
        choice.add("LABEL");
        choice.addItemListener(new CardLayoutChoiceListener());
        top.add(choice);
        
        CardLayout layout = new CardLayout();
        cardPannel = new Panel(layout);
        Panel p1 = new Panel();
        p1.add(new Button("one"));
        p1.add(new Button("two"));
        p1.add(new Button("three"));
        cardPannel.add("BUTTON", p1);
        Panel p2 = new Panel();
        p2.add(new Label("label"));
        p2.add(new Label("label"));
        p2.add(new Label("label"));
        cardPannel.add("LABEL", p2);
        top.add(cardPannel);
        
        frame.add(top);
        frame.pack();
        frame.show();
    }
```

Choice的监听对象类如下：

```java
class CardLayoutChoiceListener implements ItemListener{

    @Override
    public void itemStateChanged(ItemEvent e) {
        // TODO Auto-generated method stub
        ((CardLayout)APP.cardPannel.getLayout()).show(APP.cardPannel, (String) e.getItem());
    }
    
}
```

需要注意，CardLayout在进行卡片切换时，是通过卡片名来确定的，所以上面的代码将Choice的标题设置为和卡片的名称一致。

![](https://static.oschina.net/uploads/space/2017/0618/105252_RdNM_2340880.png)                  ![](https://static.oschina.net/uploads/space/2017/0618/105308_A7U6_2340880.png)

    CardLayout类中方法总结如下：

```java
//默认的构造方法
public CardLayout();
//构造方法 hgap设置卡片水平间距 vgap设置卡片竖直间距 
public CardLayout(int hgap, int vgap);
//获取水平间距
public int getHgap();
//设置水平间距
public void setHgap(int hgap);
//获取竖直间距
public int getVgap();
//设置竖直间距
public void setVgap(int vgap);
//显示第一个卡片界面 parent为父容器
public void first(Container parent);
//显示下一个卡片界面
public void next(Container parent);
//显示上一个卡片界面
public void previous(Container parent);
//显示最后一个卡片界面
public void last(Container parent);
//显示指定名称的卡片界面
public void show(Container parent, String name);
```

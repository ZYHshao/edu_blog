---
title: Java开发GUI之可编辑区域
date: 2017-06-15
categories: Java
tags: []
---
## Java开发GUI之可编辑区域

    Java的awt包中提供了单行的文本编辑组件TextField与多行的文本编辑区TextArea，这两个组件都是继承自TextComponent类。

```java
    static void TextTest(){
        Frame frame = new Frame("Label");
        Panel pannel = new Panel();
        TextField textField = new TextField("请开始你的表演",16);
        //设置密文输入
//        textField.setEchoChar('*');
        textField.addTextListener(new TextFieldListener());
        pannel.add(textField);
        TextArea textArea = new TextArea("是时候表演真正的技术了···",5,20);
        pannel.add(textArea);
        frame.add(pannel);
        frame.pack();
        frame.show();
    }
```

效果如下：

![](https://static.oschina.net/uploads/space/2017/0615/163622_Rlcc_2340880.png)

TextComponent类中提供了可编辑组件的基本方法：

```java
//设置是否支持切换输入法
public void enableInputMethods(boolean enable);
//设置文本
public synchronized void setText(String t);
//获取文本
public synchronized String getText();
//获取选中的文案
public synchronized String getSelectedText();
//获取是否可编辑
public boolean isEditable();
//设置是否可编辑
public synchronized void setEditable(boolean b);
//获取背景色
public Color getBackground();
//设置背景色
public void setBackground(Color c);
//获取选中文案的起点
public synchronized int getSelectionStart();
//设置选中文案的起点
public synchronized void setSelectionStart(int selectionStart);
//设置选中文案终点
public synchronized int getSelectionEnd();
//设置选中文案终点
public synchronized void setSelectionEnd(int selectionEnd);
//设置选中文案
public synchronized void select(int selectionStart, int selectionEnd);
//选中全部文案
public synchronized void selectAll();
//设置文案变化的监听
public synchronized void addTextListener(TextListener l);
//移除监听
public synchronized void removeTextListener(TextListener l);
//获取监听者
public synchronized TextListener[] getTextListeners();
```

TextField用于单行的文本输入，并且可以设置密文输入，对登录框十分适用：

```java
//构造方法
public TextField();
//text参数设置文本
public TextField(String text);
//columns参数设置列数 会影响宽度
public TextField(int columns);
public TextField(String text, int columns);
//获取输入文本被替换成的密文字符
public char getEchoChar();
//设置输入文本被替换成的密文字符
public void setEchoChar(char c);
//设置密文字符 已经弃用 适用上面的方法
public synchronized void setEchoCharacter(char c);
//设置文案
public void setText(String t);
//获取是否设置密文输入
public boolean echoCharIsSet();
//获取列数
public int getColumns();
//设置列数
public void setColumns(int columns) ;
//添加动作监听
public synchronized void addActionListener(ActionListener l);
//移除动作监听
public synchronized void removeActionListener(ActionListener l);
//获取监听者
public synchronized ActionListener[] getActionListeners();
```

TextArea类中的方法总结如下：

```java
//构造方法
public TextArea();
//text参数设置文本
public TextArea(String text);
//设置行数与列数
public TextArea(int rows, int columns);
public TextArea(String text, int rows, int columns);
//scrollbars设置滚动条模式
/*
public static final int SCROLLBARS_BOTH = 0;//水平和竖直都显示滚动条
public static final int SCROLLBARS_VERTICAL_ONLY = 1;//仅仅显示竖直滚动条
public static final int SCROLLBARS_HORIZONTAL_ONLY = 2;//仅仅显示水平滚动条
public static final int SCROLLBARS_NONE = 3; //不显示滚动条
*/
public TextArea(String text, int rows, int columns, int scrollbars);
//在指定位置插入字符串
public void insert(String str, int pos);
//同上 已经弃用 使用上面的方法
public synchronized void insertText(String str, int pos);
//在已有文本后追加字符串
public void append(String str);
//同上，已经弃用 使用上面方法
public synchronized void appendText(String str);
//替换某个范围内的字符串
public void replaceRange(String str, int start, int end);
//同上，已经弃用 使用上面方法
public synchronized void replaceText(String str, int start, int end);
//获取行数
public int getRows();
//设置行数
public void setRows(int rows);
//获取列数
public int getColumns();
//设置列数
public void setColumns(int columns);
//获取滚动条模式
public int getScrollbarVisibility();
```

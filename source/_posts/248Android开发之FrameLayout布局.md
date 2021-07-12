---
title: Android开发之FrameLayout布局
date: 2016-09-02
categories: Android小记
tags: []
---
## Android开发之FrameLayout布局

        在Android开发中，FrameLayout是所有布局容器中最简单的一种，在前边博客中有介绍关于Android开发中线性布局LinearLayout的应用。LinearLayout采用的是线性平铺的布局模式，FrameLayout也被称为帧布局。

LinearLayout应用介绍地址：[http://my.oschina.net/u/2340880/blog/740714](http://my.oschina.net/u/2340880/blog/740714)。

        FrameLayout简单理解，可以将布局容器理解为一个单元素栈，先放入的视图在栈底，后放入的视图在栈顶，后放入的视图会覆盖先放入的视图。并且，FrameLayout不能够设置其内视图的位置，默认都是从左上角开始布局，这个布局模式在简单的重叠界面中使用十分方便。

        使用代码进行FrameLayout布局示例如下：

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        FrameLayout frameLayout = new FrameLayout(this);
        setContentView(frameLayout);
        //添加子视图
        TextView textView1 = new TextView(this);
        textView1.setLayoutParams(new FrameLayout.LayoutParams(600,600));
        textView1.setBackgroundColor(Color.RED);
        frameLayout.addView(textView1);

        TextView textView2 = new TextView(this);
        textView2.setLayoutParams(new FrameLayout.LayoutParams(400,400));
        textView2.setBackgroundColor(Color.YELLOW);
        frameLayout.addView(textView2);

        TextView textView3 = new TextView(this);
        textView3.setLayoutParams(new FrameLayout.LayoutParams(200,200));
        textView3.setBackgroundColor(Color.BLUE);
        frameLayout.addView(textView3);

        TextView textView4 = new TextView(this);
        textView4.setLayoutParams(new FrameLayout.LayoutParams(100,100));
        textView4.setBackgroundColor(Color.GREEN);
        frameLayout.addView(textView4);
    }

```

上面示例代码在FrameLayout中放入4个TextView，后放入的视图依次减小，运行后效果如下图所示：

![](http://static.oschina.net/uploads/space/2016/0902/110004_kBeZ_2340880.png)

FrameLayout应该是开发中很少使用到的一种布局模式，在十分简单的界面需求中，使用它往往十分方便。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：435043639

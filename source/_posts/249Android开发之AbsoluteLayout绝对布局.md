---
title: Android开发之AbsoluteLayout绝对布局
date: 2016-09-02
categories: Android小记
tags: []
---
## Android开发之AbsoluteLayout绝对布局

        AbsoluteLayout绝对布局已经被弃用，但是相关API依然有效，其又被称为坐标布局，在iOS开发支持Autolayout之前，所有的布局模式都可以理解为绝对布局。但是iPhone设备的屏幕尺寸有限，使用绝对不觉并不会出现太多难以解决的问题，但是对于Android设备就不同了，Android设备的屏幕尺寸和分辨率都无规范，使用坐标绝对布局的缺陷就十分明显。

        AbsoluteLayout直接通过定位其内部视图的位置坐标点和尺寸来进行布局，后添加的视图优先级更高，如果坐标有重合，会覆盖先添加的视图，示例代码如下：

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        AbsoluteLayout absoluteLayout = new AbsoluteLayout(this);
        absoluteLayout.setLayoutParams(new ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT));
        setContentView(absoluteLayout);
        //添加4个TextView
        TextView textView1 = new TextView(this);
        textView1.setText("第1个textView");
        //需要注意 这里的LayoutParams()构造方法中的参数 前两个参数为视图的宽和高 后两个为x与y位置坐标点
        textView1.setLayoutParams(new AbsoluteLayout.LayoutParams(800,300,10,10));
        textView1.setBackgroundColor(Color.RED);
        absoluteLayout.addView(textView1);

        TextView textView2 = new TextView(this);
        textView2.setText("第2个textView");
        textView2.setLayoutParams(new AbsoluteLayout.LayoutParams(800,300,100,200));
        textView2.setBackgroundColor(Color.YELLOW);
        absoluteLayout.addView(textView2);

        TextView textView3 = new TextView(this);
        textView3.setText("第3个textView");
        textView3.setLayoutParams(new AbsoluteLayout.LayoutParams(800,300,200,400));
        textView3.setBackgroundColor(Color.BLUE);
        absoluteLayout.addView(textView3);

        TextView textView4 = new TextView(this);
        textView4.setText("第4个textView");
        textView4.setLayoutParams(new AbsoluteLayout.LayoutParams(800,300,300,600));
        textView4.setBackgroundColor(Color.GREEN);
        absoluteLayout.addView(textView4);

    }
```

布局效果如下图：

![](http://static.oschina.net/uploads/space/2016/0902/202612_nKFl_2340880.png)

        其实布局容器中子视图的布局参数主要有定义在各个布局容器类的内部类LayoutParams来设置。需要注意，在不同分辨率的屏幕上，使用AbsoluteLayout布局效果可能会难于把控。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：435043639

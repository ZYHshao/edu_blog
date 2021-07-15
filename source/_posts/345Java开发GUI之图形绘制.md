---
title: Java开发GUI之图形绘制
date: 2017-06-22
categories: Java
tags: []
---
## Java开发GUI之图形绘制

    在Java的GUI组件中，每一个视图都有一个paint方法，这个方法负责组件的绘制，其中会传入Graphics对象参数，开发者可以在paint方法中操作这个对象进行自定义图形的绘制。示例如下：

```java
class DrawPanel extends Panel{
    private static final long serialVersionUID = 1L;
    public DrawPanel() {
        super();
    }
    @Override
    public void paint(Graphics g) {
        // TODO Auto-generated method stub
        super.paint(g);
        Color bg = Color.WHITE;
        Color fg = Color.RED;
        //绘制背景
        g.setColor(bg);
        g.draw3DRect(50, 50, 699, 140, true);
        g.draw3DRect(53, 53, 692, 133, true);
        
        g.setColor(fg);
        //绘制线
        g.drawLine(60, 60, 140, 60);
        //绘制矩形
        g.drawRect(150, 60, 80, 50);
        //绘制圆角矩形
        g.drawRoundRect(240, 60, 80, 50, 25, 25);
        //绘制椭圆
        g.drawOval(330, 60, 80, 50);
        //绘制弧线
        g.drawArc(420, 60, 50, 50, 0, 90);
        //绘制闭合折线
        Polygon polygon = new Polygon();
        polygon.addPoint(510, 60);
        polygon.addPoint(550, 60);
        polygon.addPoint(550, 110);
        polygon.addPoint(590, 110);
        g.drawPolygon(polygon);
        //填充矩形
        g.fillRect(600, 60, 80, 50);
        //填充3D矩形
        g.fill3DRect(60, 120, 80, 50, true);
        //填充圆角矩形
        g.fillRoundRect(150, 120, 80, 50, 25, 25);
        //填充椭圆
        g.fillOval(240, 120, 80, 50);
        //填充弧线
        g.fillArc(330, 120, 50, 50, 0, 90);
        //填充闭合折线
        Polygon polygon2 = new Polygon();
        polygon2.addPoint(390, 120);
        polygon2.addPoint(440, 120);
        polygon2.addPoint(440, 180);
        polygon2.addPoint(490, 180);
        g.fillPolygon(polygon2);
        //绘制文字
        g.drawString("finish draw test!", 500, 150);
    }
}
```

效果如下图：

![](https://static.oschina.net/uploads/space/2017/0622/105651_dVVA_2340880.png)

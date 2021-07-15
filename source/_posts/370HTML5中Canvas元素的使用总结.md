---
title: HTML5中Canvas元素的使用总结
date: 2018-04-16
categories: 前后端
tags: []
---
## HTML5中Canvas元素的使用总结

    Canvas提供了开发者自定义绘图的接口，我们可以公国getContext()函数来获取绘图上下文进行绘制操作，这个函数中可以传入两个参数，其中第1个参数设置绘图上下文的类型，比较常用的是"2d"，我们也可以使用"webgl"来使用webOpenGL实现3D绘制。本篇博客主要总结2d绘制的相关方法。

### 1.进行简单的图形绘制

    使用Canvas进行平面图形绘制比较简单。例如使用如下函数则可以直接绘制一个矩形区域。

```javascript
        var c = document.getElementById("canvas");
        var context = c.getContext("2d");
        context.strokeRect(20,20,100,100);
```

图形效果如下：

![](https://static.oschina.net/uploads/space/2018/0415/161725_h27w_2340880.png)

与strokeRect对应，使用fillRect可以绘制填充矩形，例如：

```javascript
        var c = document.getElementById("canvas");
        var context = c.getContext("2d");
        context.fillRect(20,20,100,100);
```

效果如下：

![](https://static.oschina.net/uploads/space/2018/0415/161931_YlXm_2340880.png)

使用clearRect函数可以进行矩形区域的擦除，示例如下：

```javascript
        var c = document.getElementById("canvas");
        var context = c.getContext("2d");
        context.strokeRect(20,20,100,100);
        context.clearRect(10,10,100,100);
```

效果如下图：

![](https://static.oschina.net/uploads/space/2018/0415/163043_0ARo_2340880.png)

上面的绘制图形的方法实际上是一个复合的函数，其完成了路径的定义和绘制两步。我们也可以自定义图形路径，例如：

```javascript
        var c = document.getElementById("canvas");
        var context = c.getContext("2d");
        context.beginPath();
        context.moveTo(20,20);
        context.lineTo(20,100);
        context.lineTo(100,100);
        context.closePath();
        context.stroke();
```

效果如下：

![](https://static.oschina.net/uploads/space/2018/0415/164230_Ly6O_2340880.png)

beginPath函数用来开启一个路径，moveTo函数用于将画笔移动到某个点，lineTo函数用来定义一条线，线的起点为当前画笔所在位置，参数为终点位置。closePath函数用来关闭路径，当然，此函数并非一定要调用，如果不调用可以绘制非闭合的图形。stroke函数用来将已经定义的图形进行绘制，与其对应还有一个fill函数用来进行填充绘制。

    quadraticCurveTo与bezierCurveTo两个函数分别用来创建二次与三次贝塞尔曲线路径，示例如下：

```javascript
        context.moveTo(20,120);
        context.quadraticCurveTo(20,200,100,180);
        context.stroke();
        context.moveTo(20,200);
        context.bezierCurveTo(20,300,60,300,60,200);
        context.stroke();
```

效果如下图：

![](https://static.oschina.net/uploads/space/2018/0415/165436_dztx_2340880.png)

关于贝塞尔曲线的相关内容，可以查阅下面的博客：

[https://my.oschina.net/u/2340880/blog/1519503](https://my.oschina.net/u/2340880/blog/1519503)。

arc函数用来创建圆形弧线，例如：

```javascript
        context.moveTo(110,350);
        context.arc(60,350,50,0,2*Math.PI,false);
        context.stroke();
```

arc函数中，前两个个参数设置圆心点，第3个参数设置半径，第4个和第5个参数设置圆弧的起始点和结束点，以弧度制表示，最后一个参数为布尔值，设置是否逆向绘制。还有一个arcTo函数用来绘制两条切线间的圆弧，如下：

```javascript
        context.moveTo(20,420);
        context.arcTo(80,420,80,600,50);
        context.stroke();
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2018/0415/171257_WWed_2340880.png)

使用clip函数可以进行裁剪操作，裁剪之后，之后的绘制只能绘制在裁剪的区域内，例如：

```javascript
        context.rect(0,500,100,30);
        context.clip();
        context.fillRect(0,500,200,200);
```

效果如下：

![](https://static.oschina.net/uploads/space/2018/0415/172002_NfqK_2340880.png)

有一点需要注意，使用clip函数进行裁剪后，之后的绘制将只能在裁剪的区域内进行绘制，如果想在裁剪区域外绘制，需要使用save和restore两个函数来处理，在裁剪前，使用save函数来保存当前绘图上下文的状态，想要在裁剪区域外绘制时使用restore函数来还原绘图上下文。

### 2.绘制文本和图像

    前面示例了使用Canvas进行图形的绘制，除了图形，使用Canvas也可以轻松的绘制出图像与文本。使用drawImage函数进行图像的绘制，如下：

```javascript
        var image = document.createElement("img");
        image.src = 'img/HBuilder.png';
        image.onload = function(){
                context.drawImage(image,0,600);
        }
```

需要注意，上面创建了img元素后，设置src属性后不能立刻进行渲染，因为图片的加载是需要时间的，直接渲染会无法获取图像数据。drawImage这个函数总共可以有8个参数，drawImage(img,sx,sy,sw,sh,x,y,w,h)。其中sx，sy和sw，sh用来对原图像进行裁剪，只选择图像中的部分进行绘制，x，y，w，h设置绘制在画布上的坐标和尺寸。

    关于文本绘制，可以使用fillText或strokeText函数，分别用来绘制实心和空心文字。示例如下：

```javascript
        context.font = "20px Georgia";//设置字体
         context.textAlign = 'start';  //设置文字对齐方式
        context.fillText("Hello World",0,750,200);
        context.strokeText("Hello World",0,800,200);
```

效果如下：

![](https://static.oschina.net/uploads/space/2018/0416/143248_kpAF_2340880.png)

### 3.绘制属性的设置

    在绘制过程中，开发者可以对绘制的线条颜色，填充颜色，风格，阴影等进行设置。示例如下：

```javascript
        context.fillStyle = 'red';  //设置填充颜色
        context.strokeStyle = 'blue'; //设置线条颜色
        context.shadowColor = 'green'; //设置阴影颜色
        context.shadowBlur = 10;   //设置阴影模糊度
        context.shadowOffsetX = 10; //设置阴影X轴偏移量
        context.shadowOffsetY = 5;  //设置阴影Y轴偏移量
        context.lineCap = 'round';  //设置线帽样式
        context.lineJoin = 'round'; //设置折点样式
        context.lineWidth = 1; //设置线条宽度
```

效果如下图：

![](https://static.oschina.net/uploads/space/2018/0416/145421_6qW9_2340880.png)

关于fillStyle和strokeStyle两个属性比较特殊，从名字也可以了解其是设置填充或线条的风格，设置颜色只是一种方式，其还可以设置为一个渐变对象，用来实现渐变效果。例如：

```javascript
        var g = context.createLinearGradient(0,750,200,750);
        g.addColorStop(0,'black');
        g.addColorStop(0.5,'red');
        context.fillStyle =  g;
        context.fillText("Hello World",0,750,200);
```

效果入下图：

![](https://static.oschina.net/uploads/space/2018/0416/151055_jvWU_2340880.png)

createLinearGradient函数用来创建线性渐变层，其中4个参数设置起始点的x，y和结束点的x，y。调用addColorStop函数用来想渐变层中添加临界点和颜色值。也可以创建发散型渐变，例如：

```javascript
        var g = context.createRadialGradient(70,800,20,70,800,50);
        g.addColorStop(0,'black');
        g.addColorStop(1,'red');
        context.fillStyle =  g;
        context.fillRect(20,750,100,100);
```

效果如下：

![](https://static.oschina.net/uploads/space/2018/0416/152026_wtvs_2340880.png)

createRadiaGradient函数的前3个参数设置渐变开始处的圆弧(分别设置圆心x，y坐标和半径)，后3个参数设置渐变结束处的圆弧(分别设置圆心x，y坐标和半径)。

    fillStyle和strokeStyle也可以设置为一个模式背景，例如将图片进行重复得到的背景，示例如下：

```javascript
        image.onload = function(){
            var p = context.createPattern(image,'repeat');
            context.fillStyle =  p;
            context.fillRect(20,750,200,200);
        }
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2018/0416/153038_fHe0_2340880.png)

可选的重复模式还有：

repeat-x：只在水平方向重复。

repeat-y：只在竖直方向重复。

no-repeat：不重复，只显示一次。

### 4.进行画布转换

    画布也可以进行一些简单的变换操作，例如旋转，缩放等等。需要注意，对画布的操作不会影响到已经绘制到画布上的内容，之后绘制的内容会受到影响。使用scale(x,y)函数可以对画布进行缩放，其中两个参数x和y分别设置水平和竖直方向的缩放比例。rotate(angle)函数用来对画布进行旋转，其中的参数为旋转的角度值。translate(x,y)函数用来对画布进行平移，参数x，y分别设置水平和竖直方向的平移量。还有一个复合的transform(a,b,c,d,e,f)函数，使用这个函数可以一步设置平移，旋转和缩放属性，参数意义如下：

a：设置水平缩放比

b：设置水平倾斜

c：设置垂直倾斜

d：设置垂直缩放比

e：设置水平平移

f：设置垂直平移

需要注意，如果你多次调用transform，每次的transform变换都将在上一次的基础上进行。如果你不想保留上一次的记录，可以调用setTransform()函数来重置设置。

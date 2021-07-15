---
title: 一起玩转树莓派（3）——点亮RGB炫彩LED灯
date: 2021-07-10
categories: Linux
tags: []
---
# 一起玩转树莓派（3）——点亮RGB炫彩LED灯

在阅读本篇博客之前，如果你对树莓派的GPIO还没有基本的了解，我建议你先阅读本系列博客的上一篇，关于双色LED灯实验的。了解树莓派GPIO的基本用法是进行本篇博客实验的基础。上篇博客地址如下：

[https://my.oschina.net/u/2340880/blog/5123429](https://my.oschina.net/u/2340880/blog/5123429)

现在，如果你已经成功完成过树莓派双色LED灯实验，并且对深入树莓派玩法有更多的兴趣的话，那么我们在进一步，尝试点亮一个更加绚丽的LED灯：RGB3色LED灯。

## 一、认识RGB三原色LED灯并连线

通过前面的实验，我们已经知道，双色的LED灯内部封装了红绿两个发光二极管，其有3个引脚，其中1个引脚是共用的(共阴或共阳)，对于共阴型的双色LED灯，控制另外两个引脚的高低电平来分别点亮红灯或绿灯。对于RGB3色LED灯也类似，只是其内部封装了3个发光二极管，分别可以发出红光，绿光和蓝光。其有4个引脚，1个引脚共用和3个控制发光二极管的引脚。

本次实验，我们依然采用共阴型的RGB3色LED灯，元件如下图所示：

![](https://oscimg.oschina.net/oscnet/up-49ede8533183d038436f5fa1c6ba00a9b7e.JPEG)

共阴型的LED灯，GND引脚是其公共的阴极，接线的时候我们需要将此引脚接地，另外3个引脚分别接3个GPIO来控制亮灯。下面两种图，非常直观的演示了此LED灯的工作原理：

![](https://oscimg.oschina.net/oscnet/up-b1304b74a0ad7187e624b765bc27563c4fb.png)![](https://oscimg.oschina.net/oscnet/up-e89603ce34e6093a564a8998b4e095947ac.png)

在将LED灯连接到树莓派之前，我们需要预定几个要用的GPIO引脚，之后我们在编写代码时，依然采用物理编码，首先我们先确定要使用的GPIO引脚的BCM编码下的GPIO18，GPIO19和GPIO20，通常查看引脚编码对应图，我们可以找到其所对应的物理引脚分别为12，24和28。如果不使用扩展板，直接将原件上的对应引脚连接到树莓派的这些物理引脚上即可，如果使用的是BCM编码的扩展板，则我们在连线时无需关心这些物理引脚，直接连接即可，如下：

![](https://oscimg.oschina.net/oscnet/up-e0348c7f0ae163621b3bb6b00b44e9053ec.JPEG)

好了，现在我们已经完成了基本的连线工作。

## 二、三原色与脉冲宽度调制

三原色本指色彩中不能再分解的三种基本颜色，在光学上，红、绿、蓝为最基本的三原色。三原色经过混合后，可以组成各式各样的颜色。例如将三原色等比混合后将能组成白色，将红色和绿色组合后会生成黄色，将红色和蓝色混合后会得到紫色等等。如下图所示：

![](https://oscimg.oschina.net/oscnet/up-e6794fc6def4e70f6090ccb7464147ed4f7.png)

因此，从原理上说，我们只要可以控制RGB灯三种颜色的显示亮度，就可以让LED灯调制出各种颜色。控制LED等中各个发光二极管的亮和灭非常简单，我们只需要向其加高电平或低电平即可，那么如何控制发光二极管的亮度呢？我们需要使用到另外一种电流控制技术：PWM脉冲宽度调制。

脉冲宽度调制(PWM)是一种模拟控制方式，其通过控制脉冲电压中高电压的占空比来控制流过元件的电流大小。PWM技术中有两个非常重要的参数：频率与占空比。频率用来控制脉冲信号的周期，如果频率过低，在控制LED灯的时候，灯就会进行闪烁，当频率足够高，人眼已无法分辨出其闪烁，看上起LED灯就是常亮的。占空比指的是在输出的脉冲信号中，高电平保持的时间与该脉冲信号的周期时间之比。例如，假设设置周期为100Hz，则其周期时间为10ms，如果设置的占空比为20%，则当前周期中，高电平的占比时间为2ms。

Python的GPIO库中提供了PWM控制接口，使用也非常简单，使用如下方法可以获取某个引脚的PWM实例：

```python
p = GPIO.PWM(channel, frequency)

```

其中，channel参数为引脚编码，frequency参数为设置的PWM频率。下面方法用来开启PWM脉冲：

```python
p.start(dc)

```

其中，dc参数设置脉冲的高电平占空比，取值范围为0-100。

通过下面的方法可以对PWM脉冲频率和占空比进行修改：

```objectivec
p.ChangeFrequency(freq) 
p.ChangeDutyCycle(dc)

```

需要结束PWM脉冲调制时，可以调用如下方法：

```python
p.stop()

```

## 三、点亮炫彩的三彩LED灯

现在，我们已经做好了所有准备工作，可以开始编码了。我们要实现这样一个功能，当程序运行时，先控制LED灯的红灯，绿灯，靛色灯分别亮2秒，之后通过脉冲混合，让LED灯进行各种颜色的炫彩闪烁，完整代码如下：

```python
#coding:utf-8

# 导入GPIO控制薄块
import RPi.GPIO as GPIO
# 导入time模块
import time
# 导入系统模块
import sys

# 定义引脚(物理引脚)
R,G,B = 12,35,38
# 设置使用的引脚编码模式
GPIO.setmode(GPIO.BOARD)

# 对要使用的引脚进行初始化
GPIO.setup(R,GPIO.OUT)
GPIO.setup(G,GPIO.OUT)
GPIO.setup(B,GPIO.OUT)

# 使用PWM脉冲宽度调制
pR = GPIO.PWM(R, 60)
pG = GPIO.PWM(G, 60)
pB = GPIO.PWM(B, 60)

# 开启脉冲，默认的占空比为0，灯不亮
pR.start(0)
pG.start(0)
pB.start(0)

# 初始时，各种颜色点亮2秒
# 红灯先亮2秒
pR.ChangeDutyCycle(100)
pG.ChangeDutyCycle(0)
pB.ChangeDutyCycle(0)
time.sleep(2)

# 替换为绿灯亮2秒
pR.ChangeDutyCycle(0)
pG.ChangeDutyCycle(100)
pB.ChangeDutyCycle(0)
time.sleep(2)

# 替换为靛色灯亮2秒
pR.ChangeDutyCycle(0)
pG.ChangeDutyCycle(0)
pB.ChangeDutyCycle(100)
time.sleep(2)

# 定义要闪烁的时间 这里定义为10秒
endTime = 100
current = 0

# 开始进行炫彩闪烁
while True:
    # 通过占空比控制红色的占比
    for r in range(0, 101, 10):
        pR.ChangeDutyCycle(r)
        # 通过占空比控制绿色的占比
        for g in range(0, 101, 10):
            pG.ChangeDutyCycle(g)
            # 空通过占空比控制蓝色的占比
            for b in range(0, 101, 10):
                pB.ChangeDutyCycle(b)
                time.sleep(0.1)
                current += 1
                # 结束程序
                if (current > endTime):
                    pR.stop()
                    pG.stop()
                    pB.stop()
                    GPIO.cleanup()
                    sys.exit(0)

```

在树莓派上运行此程序，注意！小心不要被太亮的LED闪到了眼睛😝。

> 专注技术，懂的热爱，愿意分享，做个朋友
> 
> QQ：316045346

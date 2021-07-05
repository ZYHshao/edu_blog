---
title: Cocos2d-x-v3动作体系
date: 2015-08-05
categories: COCOS2D
tags: []
---
## Cocos2d-x-v3动作体系

        cocos2d-x-v3版本v2的版本有的很大的改动，最直观的是在一些函数的改动和类名的改动上，首先以CC开头的类，都不再使用CC。在我个人的理解上，原来的cocos2d-x是完全从iphone的框架cocos2d移植过来的，代码的风格和编程的思想都及类似于Object-C，除了语法是C++的外，其他就像是OC版的翻译，新的版本更好的体现了跨平台的特性，也更多的运用了C++的一些编码思想和语言特点，更易于各平台的开发者使用。这篇博客的主要内容，是总结cocos2d中行为动作的处理方法和相关函数。

### 一、瞬时动作

        这类行为只能称为动作，不能称作动画，其执行是瞬时的，没有可是化的过程。

        cocos2d中常用的瞬时动作有如下几种：

      FlipX：关于x轴做镜像变换。

      FlipY：关于y轴做镜像变换。

      Hide：隐藏。

      Show：显示。

      ToggleVisibility：切换隐藏和显示。

      Place：将对象放置在某个位置。

### 二、延时动作

       延时动作就是动画，将动作的过程展现出来，cocos2d引擎中的几种延时动作如下：

       1\. MoveTo:将对象移动到某一位置，是绝对位置，移动后不会记录对象的原始位置，动作不能进行反转。例如：

```
auto action = MoveTo::create(2, Vec2(100, 100));//2S时间移动到(100,100)
    label->runAction(action);//执行动作
```

        2.MoveBy:将对象相对现在的位置移动某个距离，这个移动是相对对象当前位置的，可以反转。

      3.JumpTo:和MoveTo类似，对象跳动到某一位置，例如：

```
label->runAction(JumpTo::create(2, Vec2(100, 100), 30, 3));//对象在2S内跳三次，每次高度为30像素，跳到(100,100)点
```

       4.JumpBy:和MoveBy类似。

        5.BezierTo:以贝塞尔曲线的方式移动到某一位置，例如：

```
ccBezierConfig config;
    config.controlPoint_1=Vec2(300, 300);
    config.controlPoint_2=Vec2(200,200);
    config.endPosition=Vec2(100, 100);//设置两个中间点和一个终点
    
    label->runAction(BezierTo::create(2, config));//2S时间通过贝塞尔曲线方式移动
```

        6.BezierBy:以贝塞尔曲线的方式进行相对移动。

      7.ScaleTo:相对原始大小缩放到某一尺度。

      8.ScaleBy:相对目前大小进行缩放。

      9.RotateTo:相对原始状态旋转到某一角度。

      10.RotateBy：相对目前转台旋转某个角度。

      11.Blink:闪烁动画。

      12.TintTo：颜色转化到某一色值

      13.TintBy:相对目前色值，颜色相对转变某一色值。

      14.FadeTo:变暗到某一透明度

      15.FadeIn:淡入动作

      16.FadeOut：淡出动作

### 三、动作的组合方式

        cocos2d中不仅为我们提供的各种动作方式，也为我们提供了相关的类用于管理这些动作：

     1.动作序列Sequence：这个类可以创建一个动作序列，按序列中动作的顺序依次执行动作，如下：

```
 Sequence * sq= Sequence::create(TintTo::create(2, Color3B(123, 123, 123)),RotateTo::create(2, 30), NULL);
    label->runAction(sq);//创建动作序列，使对象执行先变颜色，在旋转的动画
```

      2.同步动作组Spawn：这个类和Sequence类似，只是他里面的动画会同时一起执行。

    3.有限次的循环动作Repeat：这个类可以使某一动作循环执行数次，例如：

```
Repeat * re = Repeat::create(RotateBy::create(2, 30), 5);//旋转5次30度
    label->runAction(re);
```

    4.无限次循环动作RepeatForever：

```
RepeatForever * ref = RepeatForever::create(RotateBy::create(2, 30));
    label->runAction(ref);
```

    5.帧动画

cocos2d中同样提供了对帧动画的支持：

```
   //创建设置精灵
    Sprite * spr = Sprite::create( "CloseNormal.png");
    spr->setPosition(Vec2(100, 100));
    //创建两帧精灵图片
    SpriteFrame * frame1 = SpriteFrame::create("CloseNormal.png", Rect(0, 0, 50, 50));
    SpriteFrame * frame2 = SpriteFrame::create("CloseSelected.png", Rect(0, 0, 50, 50));
    Vector<SpriteFrame *>  arr;
    arr.pushBack(frame1);
    arr.pushBack(frame2);
    //创建动画体 第一个参数是帧容器，第二个是每一帧的播放时间，第三个是循环次数
    Animation * ani = Animation::createWithSpriteFrames(arr, 1, 1);
    //创建动作
    Animate *ant = Animate::create(ani);
    RepeatForever * ref = RepeatForever::create(ant);
    spr->runAction(ref);
    this->addChild(spr);
```

    6.反转动画

可以通过reverse方法获取动作的反转动作，例如：

```
auto label = Label::createWithTTF("Hello World", "fonts/arial.ttf", 24);
    MoveBy * move = MoveBy::create(3, Vec2(100, 100));
    Sequence * sq = Sequence::create(move,move->reverse(), NULL);
    label->runAction(sq);
    //label 会先相对移动(100,100)，再反移动回来
```

    7.动作的速度控制

通过一些速度相关的类，cocos2d可以很轻松的创建出各种线性与非线性的动作。例如：

```
auto label = Label::createWithTTF("Hello World", "fonts/arial.ttf", 24);
    MoveTo * move = MoveTo::create(3, Vec2(-200, -200));
    EaseIn* an = EaseIn::create(move, 5);
    label->runAction(an);
//label的运动会先慢后快，速度差为5倍
```

EaseIn:由慢变快，线性

EaseOut：由快变慢，线性

EaseInOut：由慢变快再由快变慢

EaseSineIn:由慢变快，正弦规律

EaseSineOut:由快变慢，正弦规律

EaseSineInOut：由慢变快再由快变慢，正弦规律

EaseExponentialIn:由慢变快,指数规律

EaseExponentialOut:由快变慢,指数规律

EaseExponentialInOut:由慢变快再由快变慢,指数规律

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

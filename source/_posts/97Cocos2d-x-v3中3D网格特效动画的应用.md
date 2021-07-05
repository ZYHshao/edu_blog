---
title: Cocos2d-x-v3中3D网格特效动画的应用
date: 2015-08-06
categories: COCOS2D
tags: []
---
## Cocos2d-x-v3中3D网格特效动画的应用

### 一、网格特效的使用原理

        基础的动作是对节点整体进行移动，变形等操作，网格特效的原理是将节点分割成多个尺寸相同的网格，根据改变每个网格块的属性使整体节点产生3D的效果。

### 二、网格特效的基本用法

      在cocos2d-x中，v3的版本新引入了一个类NodeGrid，专门用来包装网格的特效，示例如下：

```
    //获取屏幕尺寸
    Size visibleSize = Director::getInstance()->getVisibleSize();
    Vec2 origin = Director::getInstance()->getVisibleOrigin();
    //加载精灵
    auto sprite = Sprite::create("HelloWorld.png");
    sprite->setPosition(Vec2(visibleSize.width/2+origin.x, visibleSize.height/2+origin.y));
    //创建网格特效包装类
    NodeGrid * nodeg = NodeGrid::create();
    nodeg->setPosition(Vec2::ZERO);
    //添加播放特效的精灵
    nodeg->addChild(sprite);
    this->addChild(nodeg);
    //参数的含义 分别是 执行时间，切分的网格大小，波浪次数，波浪大小
    Waves3D * ani3d = Waves3D::create(2, Size(15, 15), 6, 4);
    //执行特效
    nodeg->runAction(ani3d);
```

### 三、系统提供的网格特效

static Waves3D* create(float duration, const Size& gridSize, unsigned int waves, float amplitude);

创建波浪3D效果，参数含义为：执行时间，网格尺寸，波浪次数，波浪大小

static FlipX3D* create(float duration);

以x为轴进行翻转

static FlipY3D* create(float duration);

以y为轴进行翻转

static Lens3D* create(float duration, const Size& gridSize, const Vec2& position, float radius);

创建镜头的3D效果，参数为：执行时间，网格大小，镜头中心，镜头半径

static Ripple3D* create(float duration, const Size& gridSize, const Vec2& position, float radius, unsigned int waves, float amplitude);

创建波纹特效，参数为：执行时间，网格大小，波纹中心，波纹半径，波纹计数，振幅

static Shaky3D* create(float initWithDuration, const Size& gridSize, int range, bool shakeZ);

创建震动特效，参数为：执行时间，网格大小，震动范围，是否波动z轴

static Liquid* create(float duration, const Size& gridSize, unsigned int waves, float amplitude);

创建液体特效，参数为：执行时间，网格尺寸，流动次数，幅度

static Waves* create(float duration, const Size& gridSize, unsigned int waves, float amplitude, bool horizontal, bool vertical);

创建平面波纹特效，参数为：执行时间，网格尺寸，波纹次数，波纹振幅，开关横向波纹，开关纵向波纹

 static Twirl* create(float duration, const Size& gridSize, Vec2 position, unsigned int twirls, float amplitude);  
创建扭曲旋转的特效，参数为：执行时间，网格尺寸，扭曲旋转中心，旋转次数，旋转幅度

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

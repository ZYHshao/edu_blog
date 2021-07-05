---
title: Cocos2d-x-v3坐标体系
date: 2015-08-03
categories: COCOS2D
tags: []
---
## Cocos2d-x坐标体系

        cocos2d引擎是一款非常优秀的扩平台的游戏开发引擎，在apple游戏榜上，有很多排名靠前的游戏都是由他创造出来的，他也有一套十分方便的坐标体系。

### 一、UI坐标体系

        UI坐标体系相对于移动开发人员来说再熟悉不过了，在iOS系统中，它就是frame体系，即坐标(0,0)点位于屏幕的左上角，向右x增大，向下y增大。

### 二、OpenGL坐标体系

        OpenGL坐标系是cocos2d中使用的坐标系，它更接近于数学上的坐标系，即(0,0)点位于屏幕的左下角，往左x增大，往上y增大。这套坐标系统也更符合物理世界的逻辑，便于游戏的开发。当然，这并不是说cocos2d中所有的坐标都是采用这个体系标准的，在手指点击事件层，接收到点击坐标点的坐标就是采用UI坐标系表示的。

### 三、世界坐标系

        简单的理解，世界坐标系就是绝对坐标系，在cocos2d中，精灵的坐标是相对于其父视图而言的，是相对的坐标，世界坐标则是统一绝对的坐标，在项目中是固定的。

### 四、相对坐标系

        最常用的坐标体系，任何类设置的坐标都是相对于其父视图原点的坐标。

### 五、坐标系的转换

        由于UI坐标系与OpenGL坐标系的差异，在开发中，我们有时需要其两个标准的相互转化，cocos2d中也未我们提供了相应的方法：

Vec2 Director::convertToGL(const Vec2& uiPoint);

        这个方法将UI坐标系转换为OpenGL坐标系。

Vec2 Director::convertToUI(const Vec2& glPoint);

        这个方法将OpenGL坐标系转换为UI坐标系。

Vec2 Node::convertToWorldSpace(const Vec2& nodePoint) const;

        这个方法将物体的相对坐标

Vec2 Node::convertToNodeSpace(const Vec2& worldPoint) const;

        这个方法将世界坐标转化为某一节点的相对坐标。

还有两个转化的方法与上面类似，只有一点不同，这两个方法参照的原点不是系统默认的，而是我们设置的节点的锚点：

Vec2 Node::convertToNodeSpaceAR(const Vec2& worldPoint) const;

Vec2 Node::convertToWorldSpaceAR(const Vec2& nodePoint) const;

### 六、锚点

    锚点的概念可以理解为参照点，其设置范围为0-1，系统默认的节点锚点为(0,0)。在UI坐标系中，(0,0)点就是节点的左上角，在OpenGL坐标系中，(0,0)点就是节点的左下角。例如，如果我将锚点设置为(0.5,0.5),则在UI和OpenGL坐标系中，(0,0)点都是节点的中心点。又如，我将锚点设置为(1,1)，则在UI坐标系中，原点为右下角，在OpenGL坐标系中，原点为右上角，锚点的用处就是更改参考点，在另一种情形下，锚点对程序也会产生很大的影响，就是当我们设置一个节点旋转或者缩放时，节点会以锚点位置为中心进行旋转或缩放。

    cocos2d中通过下面方法分别来设置和获取锚点：

void Sprite::setAnchorPoint(const Vec2& anchor);

设置锚点

const Vec2& Node::getAnchorPoint() const;

获取锚点

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

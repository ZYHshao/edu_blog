---
title: Cocos2d-x-v3场景切换
date: 2015-08-05
categories: COCOS2D
tags: []
---
## Cocos2d-x-v3场景切换

        cocos2d中场景的切换采用的是包装的思想，通过创建一个专场效果类，将需要专场的场景进行包装。代码示例如下：

```
    auto * scene = OtherScene::createScene();//创建一个场景
    Director::getInstance()->replaceScene(TransitionFlipX::create(1, scene));//进行包装切换，第一个参数为切换时间，第二个为切换的场景
```

引擎为我们封装的特效有很多，函数方法如下：

static TransitionRotoZoom* create(float t, Scene* scene);

旧的场景旋转缩小到中心点后再将新的场景旋转放大完成切换

static TransitionJumpZoom* create(float t, Scene* scene);

旧场景弹跳缩小移出，新场景弹跳方法完成切换

 static TransitionMoveInL* create(float t, Scene* scene);

新的场景从左边切入(覆盖)

 static TransitionMoveInR* create(float t, Scene* scene);

新的场景从右边切入(覆盖)

static TransitionMoveInT* create(float t, Scene* scene);

新的场景从上边切入(覆盖)

 static TransitionMoveInB* create(float t, Scene* scene);

新的场景从下边切入(覆盖)

static TransitionSlideInL* create(float t, Scene* scene);

新的场景从左边推入

static TransitionSlideInR* create(float t, Scene* scene);

新的场景从右边推入

static TransitionSlideInT* create(float t, Scene* scene);

新的场景从上边推入

static TransitionSlideInB* create(float t, Scene* scene);

新的场景从下边推入

static TransitionShrinkGrow* create(float t, Scene* scene);

新的场景从后向前进行替换

static TransitionFlipX* create(float t, Scene* s, Orientation o);

场景以X为轴进行翻转切换，第三个参数为翻转的方向

static TransitionFlipY* create(float t, Scene* s, Orientation o);

场景以Y为轴进行翻转切换，第三个参数为翻转的方向

static TransitionFlipAngular* create(float t, Scene* s, Orientation o);

场景以对角线为轴进行翻转切换，第三个参数为翻转的方向

static TransitionZoomFlipX* create(float t, Scene* s, Orientation o);

场景以X轴进行翻转，带缩放效果

static TransitionZoomFlipY* create(float t, Scene* s, Orientation o);

场景以Y轴进行翻转，带缩放效果

static TransitionZoomFlipAngular* create(float t, Scene* s, Orientation o);

场景以对角线为轴进行翻转，带缩放效果

static TransitionFade* create(float duration, Scene* scene, const Color3B& color);

场景以颜色过渡进行切换

static TransitionCrossFade* create(float t, Scene* scene);

场景淡出过渡切换

static TransitionTurnOffTiles* create(float t, Scene* scene);

场景瓦片溶解切换

static TransitionSplitCols* create(float t, Scene* scene);

场景纵向切割切换

static TransitionSplitRows* create(float t, Scene* scene);

场景横向切割切换

static TransitionFadeTR* create(float t, Scene* scene);

场景向右上角过滤切换

static TransitionFadeBL* create(float t, Scene* scene);

场景向左下角过滤切换

static TransitionFadeUp* create(float t, Scene* scene);

场景向上过滤切换

static TransitionFadeDown* create(float t, Scene* scene);

场景向下过滤切换

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

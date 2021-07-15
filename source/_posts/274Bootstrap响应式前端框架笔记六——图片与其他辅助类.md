---
title: Bootstrap响应式前端框架笔记六——图片与其他辅助类
date: 2016-12-08
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记六——图片与其他辅助类

    在页面中插入图片，Bootstrap框架中定义了3中图片的Css类样式，分别为圆角图片img-rounded类，圆形图片img-circle类和带边框的图片img-thumbnail类，示例如下：

```html
        <p>设置img-rounded类可以使图片显示圆角，img-circle类可以使图片显示圆形，img-thumbnail可以为图片加上边框</p>
        <img src="image/test.png" class="img-rounded" />
        <img src="image/test.png" class="img-circle" />
        <img src="image/test.png" class="img-thumbnail" />
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1208/153325_WQU3_2340880.png)

    text-xxx相关类定义了一些常用的字体颜色，示例如下：

```html
        <p class="text-muted">正常文字</p>
        <p class="text-primary">重要文字</p>
        <p class="text-success">成功文字</p>
        <p class="text-info">详情文字</p>
        <p class="text-warning">警告文字</p>
        <p class="text-danger">危险文字</p>
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1208/154552_DEHB_2340880.png)

与上面文字颜色的类相对应，Bootstrap中也定义了一组背景颜色类，示例如下：

```html
        <p class="bg-muted">正常背景</p>
        <p class="bg-primary">重要背景</p>
        <p class="bg-success">成功背景</p>
        <p class="bg-info">详情背景</p>
        <p class="bg-warning">警告背景</p>
        <p class="bg-danger">危险背景</p>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1208/155059_qtiI_2340880.png)

使用caret类可以方便的创建倒三角图案，示例如下：

```html
        <p>使用caret类可以创建一个倒三角图案</p>
        <span class="caret"></span>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1208/155720_BGqB_2340880.png)

使用show和hidden类可以进行标签的显示与隐藏，示例如下：

```html
        <p class="hidden">show和hidden可以进行便签的显示与隐藏</p>
```

Bootstrap中还提供了一些与响应类开发相关的类，开发者可以设置某些元素在某个尺寸的屏幕中可见或者隐藏，也可以设置某个元素在浏览器或打印机上可见或隐藏，如下：

屏幕尺寸响应式类：

![](https://static.oschina.net/uploads/space/2016/1208/160458_GInb_2340880.png)

显示设备响应式类：

![](https://static.oschina.net/uploads/space/2016/1208/160604_Asxa_2340880.png)

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/images.html](http://zyhshao.github.io/bootStrapDemo/images.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

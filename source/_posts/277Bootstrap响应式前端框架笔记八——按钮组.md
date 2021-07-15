---
title: Bootstrap响应式前端框架笔记八——按钮组
date: 2016-12-10
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记八——按钮组

    在Bootstrap定义的Css样式中，开发者可以将一组btn控件包裹在btn-group类中使其组合成按钮组控件，组合后的控件左右两侧的按钮将被圆角处理，示例代码如下：

```html
        <p>正常的按钮组</p>
        <div class="btn-group">
            <button class="btn btn-default">左按钮</button>
            <button class="btn btn-danger">中心按钮</button>
            <button class="btn btn-primary">右按钮</button>
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1210/115111_adMS_2340880.png)

    也可以将一组按钮组包裹在btn-toolbar类中，使其组合成为按钮组工具栏，示例如下：

```html
        <p>按钮组工具栏</p>
        <div class="btn-toolbar">
            <div class="btn-group">
                <button class="btn btn-default">左按钮</button>
                <button class="btn btn-danger">中心按钮</button>
                <button class="btn btn-primary">右按钮</button>
            </div>
            <div class="btn-group">
                <button class="btn btn-default">左按钮</button>
                <button class="btn btn-primary">右按钮</button>
            </div>
            <div class="btn-group">
                <button class="btn btn-primary">独立按钮</button>
            </div>
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1210/115637_IkHd_2340880.png)

按钮组也可以进行嵌套，使用按钮组嵌套的方式也可以实现下拉菜单效果，示例如下：

```html
        <div class="btn-group">
            <button class="btn btn-default">左按钮</button>
            <button class="btn btn-danger">中心按钮</button>
            <div class="btn-group">
                <button class="btn btn-info dropdown-toggle">菜单
                <span class="caret"></span>
                </button>
                <ul class="dropdown-menu">
                    <li><a>金牛</a></li>
                    <li><a>狮子</a></li>
                </ul>
            </div>
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1210/120243_C3e1_2340880.png)

    默认的按钮组是水平排列的，为其设置btn-group-vertical类可以将其设置为竖直排列的，示例如下：

```html
        <p>竖直排列的按钮组</p>
        <div class="btn-group-vertical">
            <button class="btn btn-default">左按钮</button>
            <button class="btn btn-danger">中心按钮</button>
            <button class="btn btn-primary">右按钮</button>
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1210/120535_J6lv_2340880.png)

    如果需要使按钮组填充其父容器，需要设置btn-group-justified类，并且使用a标签作为按钮，示例如下：

```html
        <p>设置按钮组宽度充满父容器</p>
        <div class="btn-group btn-group-justified">
            <a class="btn btn-default">左按钮</a>
            <a class="btn btn-danger">中心按钮</a>
            <a class="btn btn-primary">右按钮</a>
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1210/121206_E9VY_2340880.png)

    通过按钮组，可以十分方便的实现分裂式下拉菜单，示例如下：

```html
        <p>分裂式下拉菜单</p>
        <div class="btn-group">
            <button type="button" class="btn btn-danger">金牛</button>
            <button type="button" class="btn btn-danger dropdown-toggle">
                <span class="caret"></span>
                </button>
            <ul class="dropdown-menu">
                <li>
                    <a href="#">金牛</a>
                </li>
                <li>
                    <a href="#">狮子</a>
                </li>
                <li>
                    <a href="#">摩羯</a>
                </li>
                <li class="divider"></li>
                <li>
                    <a href="#">无</a>
                </li>
            </ul>
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1210/121816_7SVv_2340880.png)

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/buttonGroup.html](http://zyhshao.github.io/bootStrapDemo/buttonGroup.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

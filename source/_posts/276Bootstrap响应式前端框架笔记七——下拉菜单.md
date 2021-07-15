---
title: Bootstrap响应式前端框架笔记七——下拉菜单
date: 2016-12-09
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记七——下拉菜单

    在Bootstrap的Css框架中，下拉菜单属于组件。一个完整的下拉菜单应该有两部分组成，一个触发按钮与一个选项列表。触发按钮dropdown-toggle类来创建，选项列表有drop-menu类来创建，这两部分元素需要包裹在一个dropdown类元素中，才能正确组合，示例代码如下：

```html
        <p>正常的下拉菜单样式</p>
        <div class="dropdown">
            <button class="btn btn-default dropdown-toggle">
                下拉菜单
            <span class="caret"></span>
            </button>
            <ul class="dropdown-menu" >
                <li><a>白羊座</a></li>
                <li><a>金牛座</a></li>
                <li><a>摩羯座</a></li>
                <li><a>狮子座</a></li>
            </ul>
        </div>
```

默认创建的下拉菜单是隐藏的，为了演示方便，可以将ul的display属性重设：

```html
        <style>
            ul{
                display: block !important;
            }
        </style>
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/1209/165526_51fO_2340880.png)

使用dropdown-menu-left或者dropdown-menu-right可以实现对菜单列表的左对齐或者右对齐。

    为列表的li元素添加dropdown-header类可以将此行设置为头信息行，示例如下：

```html
        <p>可以使用dropdown-header类来进行菜单头的设置</p>
        <div class="dropdown">
            <button class="btn btn-default dropdown-toggle">
                下拉菜单
            <span class="caret"></span>
            </button>
            <ul class="dropdown-menu" >
                <li class="dropdown-header">星座</li>
                <li><a>白羊座</a></li>
                <li><a>金牛座</a></li>
                <li class="dropdown-header">属相</li>
                <li><a>猴</a></li>
            </ul>
        </div>
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1209/170740_l8so_2340880.png)

    为li标签设置divider类可以将此行设置为分割线，示例如下：

```html
        <p>可以使用divider类可以为菜单设置分割线</p>
        <div class="dropdown">
            <button class="btn btn-default dropdown-toggle">
                下拉菜单
            <span class="caret"></span>
            </button>
            <ul class="dropdown-menu">
                <li class="dropdown-header">星座</li>
                <li>
                    <a>白羊座</a>
                </li>
                <li>
                    <a>金牛座</a>
                </li>
                <li class="divider"></li>
                <li class="dropdown-header">属相</li>
                <li>
                    <a>猴</a>
                </li>
            </ul>
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1209/171649_nT6P_2340880.png)

    可以为li设置disabled类来将此行选项设置为禁用，设置禁用后，此行标签也将无法点击，示例如下：

```html
        <p>可以使用disabled类来禁用某些选项</p>
        <div class="dropdown">
            <button class="btn btn-default dropdown-toggle">
                下拉菜单
            <span class="caret"></span>
            </button>
            <ul class="dropdown-menu">
                <li class="dropdown-header">星座</li>
                <li class="disabled">
                    <a>白羊座</a>
                </li>
                <li>
                    <a>金牛座</a>
                </li>
                <li class="dropdown-header">属相</li>
                <li>
                    <a>猴</a>
                </li>
            </ul>
        </div>
```

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/dropList.html](http://zyhshao.github.io/bootStrapDemo/dropList.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

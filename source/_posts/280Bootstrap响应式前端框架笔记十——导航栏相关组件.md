---
title: Bootstrap响应式前端框架笔记十——导航栏相关组件
date: 2016-12-12
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记十——导航栏相关组件

    Bootstrap中提供的导航栏分为两种模式，使用nav-tabs类可以创建页卡模式的导航栏，使用nav-pills类可以创建胶囊模式的导航栏，示例如下：

```html
        <p>导航分为两种，页卡模式和胶囊模式</p>
        <p>页卡模式</p
        <ul class="nav nav-tabs">
            <li class="active"><a>主页</a></li>
            <li><a>活动</a></li>
            <li><a>留言</a></li>
        </ul>
        <hr />
        <p>胶囊模式</p>
        <ul class="nav nav-pills">
            <li class="active"><a>主页</a></li>
            <li><a>活动</a></li>
            <li><a>留言</a></li>
        </ul>
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/1212/155120_BuM5_2340880.png)

针对胶囊式导航，也可以设置其排列方向为堆叠，添加nav-stacked类即可，示例如下：

```html
        <p>堆叠排列的胶囊导航</p>
        <ul class="nav nav-pills nav-stacked">
            <li class="active"><a>主页</a></li>
            <li><a>活动</a></li>
            <li><a>留言</a></li>
        </ul>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1212/155423_tYg8_2340880.png)

    导航中也可以进行下拉菜单的嵌套，示例如下：

```html
        <p>导航中嵌套下拉菜单</p>
        <ul class="nav nav-pills">
            <li><a>主页</a></li>
            <li><a>活动</a></li>
            <li><a>留言</a></li>
            <li class="dropdown active">
                <a class="dropdown-toggle">
                    更多
                    <span class="caret"></span>
                </a>
                <ul class="dropdown-menu">
                    <li><a>个人中心</a></li>
                    <li><a>设置</a></li>
                    <li><a>退出</a></li>
                </ul>
            </li>
        </ul>
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1212/160748_ye0X_2340880.png)

    除了默认的导航栏组件，Bootstrap中还支持自定义导航条，使用navbar类可以创建导航条容器，其内可以布局图标，文本，按钮和表单等，示例如下：

```html
        <p>自定义导航条</p>
        <nav class="navbar navbar-default">
            <div class="container">
                <div class="navbar-header">
                    <a class="navbar-brand"><img src="image/icon.png" width="20px" /></a>
                </div>
                <form class="navbar-form navbar-left">
                    <div class="form-group">
                        <input type="text" class="form-control" placeholder="Search">
                    </div>
                    <button class="btn btn-default">Search</button>
                </form>
                <button class="btn btn-warning navbar-btn navbar-left">提示</button>
                <p class="navbar-text">导航文本</p>
            </div>
        </nav>
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/1212/162318_3CfC_2340880.png)

使用navbar-fixed-top类或者navbar-fixed-bottom类可以将导航条固定在顶部或底部，示例如下：

```html
<p>将导航栏固定在顶部</p>
        <nav class="navbar navbar-default navbar-fixed-top">
            <div class="container">
                <div class="navbar-header">
                    <a class="navbar-brand"><img src="image/icon.png" width="20px" /></a>
                </div>
                <form class="navbar-form navbar-left">
                    <div class="form-group">
                        <input type="text" class="form-control" placeholder="Search">
                    </div>
                    <button class="btn btn-default">Search</button>
                </form>
                <button class="btn btn-warning navbar-btn navbar-left">提示</button>
                <p class="navbar-text">导航文本</p>
            </div>
        </nav>
        <hr />
        <p>将导航栏固定在底部</p>
        <nav class="navbar navbar-default navbar-fixed-bottom">
            <div class="container">
                <div class="navbar-header">
                    <a class="navbar-brand"><img src="image/icon.png" width="20px" /></a>
                </div>
                <form class="navbar-form navbar-left">
                    <div class="form-group">
                        <input type="text" class="form-control" placeholder="Search">
                    </div>
                    <button class="btn btn-default">Search</button>
                </form>
                <button class="btn btn-warning navbar-btn navbar-left">提示</button>
                <p class="navbar-text">导航文本</p>
            </div>
        </nav>
```

使用navbar-inverse类可以将导航条进行反色处理，示例如下：

```html
        <p>将导航条进行反色处理</p>
        <nav class="navbar navbar-default navbar-inverse">
            <div class="container">
                <div class="navbar-header">
                    <a class="navbar-brand"><img src="image/icon.png" width="20px" /></a>
                </div>
                <form class="navbar-form navbar-left">
                    <div class="form-group">
                        <input type="text" class="form-control" placeholder="Search">
                    </div>
                    <button class="btn btn-default">Search</button>
                </form>
                <button class="btn btn-warning navbar-btn navbar-left">提示</button>
                <p class="navbar-text">导航文本</p>
            </div>
        </nav>
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/1212/163724_R0KD_2340880.png)

    Bootstrap也支持进行路径导航的创建，其需要使用有序列表配合breadcrumb类，示例如下：

```html
        <p>进行路径导航的创建</p>
        <ol class="breadcrumb">
            <li>
                <a href="#">主页</a>
            </li>
            <li>
                <a href="#">新闻列表</a>
            </li>
            <li class="active">国际新闻</li>
        </ol>
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/1212/164034_GEFc_2340880.png)

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/navigation.html](http://zyhshao.github.io/bootStrapDemo/navigation.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

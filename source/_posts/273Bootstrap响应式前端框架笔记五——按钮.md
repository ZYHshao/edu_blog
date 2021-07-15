---
title: Bootstrap响应式前端框架笔记五——按钮
date: 2016-12-08
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记五——按钮

    Bootstrap中预设了default，primary，info，warning，danger和link6种按钮风格，示例如下：

```html
        <p>Bootstrap中预设的按钮样式如下</p>
        <button type="button" class="btn btn-default">正常</button>
        <button type="button" class="btn btn-primary">重要</button>
        <button type="button" class="btn btn-success">成功</button>
        <button type="button" class="btn btn-info">信息</button>
        <button type="button" class="btn btn-warning">警告</button>
        <button type="button" class="btn btn-danger">危险</button>
        <button type="button" class="btn btn-link">链接</button> 
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1208/102533_v4Wp_2340880.png)

可以为按钮元素添加btn-lg，btn-sm，btn-xs类来进行按钮尺寸的设置，示例如下：

```html
        <p>设置按钮的大小</p>
        <button type="button" class="btn btn-default btn-lg">正常</button>
        <button type="button" class="btn btn-primary">重要</button>
        <button type="button" class="btn btn-success btn-sm">成功</button>
        <button type="button" class="btn btn-info btn-xs">信息</button>
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/1208/102954_zHKz_2340880.png)

使用btn-block类可以将按钮设置为充满整个父元素，示例如下：

```html
        <p>使用btn-block类可以将按钮设置为充满父元素</p>
        <button type="button" class="btn btn-default btn-block">正常</button>
        <button type="button" class="btn btn-primary btn-block">重要</button>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1208/103257_pnL1_2340880.png)

需要注意：当button标签被用户点击时，按钮周围会出现边框，如果不需要这个边框，可以使用a标签来创建按钮。

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/button.html](http://zyhshao.github.io/bootStrapDemo/button.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

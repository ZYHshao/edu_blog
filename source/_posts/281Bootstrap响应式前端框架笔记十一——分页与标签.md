---
title: Bootstrap响应式前端框架笔记十一——分页与标签
date: 2016-12-13
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记十一——分页与标签

    在开发搜索结果页、列表页时通常会使用到分页器控件，Bootstrap中提供了方便的类来进行分页器的创建，示例如下：

```html
        <p>标准的分页器控件</p>
        <ul class="pagination">
            <li>
                <a href="#">&laquo;</a>
            </li>
            <li>
                <a href="#">1</a>
            </li>
            <li>
                <a href="#">2</a>
            </li>
            <li>
                <a href="#">3</a>
            </li>
            <li>
                <a href="#">4</a>
            </li>
            <li>
                <a href="#">5</a>
            </li>
            <li>
                <a href="#">&raquo;</a>
            </li>
        </ul>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1213/181344_cc7A_2340880.png)

为li元素添加disabled类或者active类可以将其设置为禁用或者激活状态，示例如下：

```html
        <p>使用disabled类与active类可以将页标签设置为禁用或激活</p>
        <ul class="pagination">
            <li class="disabled">
                <a href="#">&laquo;</a>
            </li>
            <li class="active">
                <a href="#">1</a>
            </li>
            <li>
                <a href="#">2</a>
            </li>
            <li>
                <a href="#">3</a>
            </li>
            <li>
                <a href="#">4</a>
            </li>
            <li>
                <a href="#">5</a>
            </li>
            <li>
                <a href="#">&raquo;</a>
            </li>
        </ul>
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/1213/181741_ldGP_2340880.png)

    除了分页器控件，Bootstrap中还提供了一种更为简单的分页控件，示例如下：

```html
        <p>只有前进与后退的分页器</p>
        <ul class="pager">
            <li>
                <a href="#">Previous</a>
            </li>
            <li>
                <a href="#">Next</a>
            </li>
        </ul>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1213/182124_SmlG_2340880.png)

这种分页控件默认是居中的，使用previous与next类可以实现两端对齐的效果，示例如下：

```html
        <p>进行两端对齐</p>
        <ul class="pager">
            <li class="previous">
                <a href="#">Previous</a>
            </li>
            <li class="next">
                <a href="#">Next</a>
            </li>
        </ul>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1213/183101_shXb_2340880.png)

    Bootstrap中的标签是一种用于展示文本的控件，示例代码如下：

```html
        <p>标签控件演示</p>
        <span class="label label-default">标签</span>
        <span class="label label-primary">标签</span>
        <span class="label label-success">标签</span>
        <span class="label label-info">标签</span>
        <span class="label label-warning">标签</span>
        <span class="label label-danger">标签</span>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1213/195025_5os2_2340880.png)

开发者也可以使用badge类来创建气泡，示例如下：

```html
        <p>进行气泡的创建</p>
        <a href="#">链接<span class="badge">3</span></a>
        <button class="btn btn-primary" type="button">
          按钮
          <span class="badge">4</span>
        </button>
```

![](https://static.oschina.net/uploads/space/2016/1213/195507_3E01_2340880.png)

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/pageAndLabel.html](http://zyhshao.github.io/bootStrapDemo/pageAndLabel.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

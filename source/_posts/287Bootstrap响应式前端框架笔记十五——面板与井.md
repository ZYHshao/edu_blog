---
title: Bootstrap响应式前端框架笔记十五——面板与井
date: 2016-12-22
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记十五——面板与井

    Bootstrap中的面板由pannel相关类来创建，一个完整的面板分为面板头部、面板体和面板注脚，并且Bootstrap中默认定义了一些面板风格，示例如下：

```html
<p>标准样式的面板</p>
            <div class="panel panel-default">
                <div class="panel-heading">
                    面板标题
                </div>
                <div class="panel-body">
                    面板内容.........
                </div>
                <div class="panel-footer">
                    面板注脚..........
                </div>
            </div>
            <div class="panel panel-primary">
                <div class="panel-heading">
                    面板标题
                </div>
                <div class="panel-body">
                    面板内容.........
                </div>
                <div class="panel-footer">
                    面板注脚..........
                </div>
            </div>
            <div class="panel panel-success">
                <div class="panel-heading">
                    面板标题
                </div>
                <div class="panel-body">
                    面板内容.........
                </div>
                <div class="panel-footer">
                    面板注脚..........
                </div>
            </div>
            <div class="panel panel-info">
                <div class="panel-heading">
                    面板标题
                </div>
                <div class="panel-body">
                    面板内容.........
                </div>
                <div class="panel-footer">
                    面板注脚..........
                </div>
            </div>
            <div class="panel panel-warning">
                <div class="panel-heading">
                    面板标题
                </div>
                <div class="panel-body">
                    面板内容.........
                </div>
                <div class="panel-footer">
                    面板注脚..........
                </div>
            </div>
            <div class="panel panel-danger">
                <div class="panel-heading">
                    面板标题
                </div>
                <div class="panel-body">
                    面板内容.........
                </div>
                <div class="panel-footer">
                    面板注脚..........
                </div>
            </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1222/203034_A3I9_2340880.png)

面板中也可以追加列表组，是的面板更具扩展性，示例如下：

```html
        <p>在面板中追加列表组</p>
        <div class="panel panel-danger">
                <div class="panel-heading">
                    面板标题
                </div>
                <div class="panel-body">
                    面板内容.........
                </div>
                <div class="list-group">
                    <div class="list-group-item">数据</div>
                    <div class="list-group-item">数据</div>
                    <div class="list-group-item">数据</div>
                    <div class="list-group-item">数据</div>
                </div>
            </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1222/203709_Dtp2_2340880.png)

    Bootstrap中还定义了一种样式Well，其效果类似嵌入界面内，示例如下：

```html
            <p>Well效果</p>
            <div class="well">
                这里是内容！！！！！！！
            </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1222/204203_CDDE_2340880.png)

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/pannelAndWell.html](http://zyhshao.github.io/bootStrapDemo/pannelAndWell.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

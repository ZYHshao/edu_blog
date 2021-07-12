---
title: Bootstrap响应式前端框架笔记一——强大的栅格布局
date: 2016-12-01
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记一——强大的栅格布局

### 一、Bootstrap？

    Bootstrap是一款HTML，Css和JavaScript开发框架，其也支持开发者进行自定义构建，开发者也可以只打包自己需要的功能模块使用。Bootstrap的中文网址如下：

[http://v3.bootcss.com/](http://v3.bootcss.com/)。

    Bootstrap是一款响应式的编程框架，所谓响应式，是指在不同屏幕尺寸的设备上，使用Bootstrap开发的项目可以自动进行布局调整适配。其响应式布局的核心是栅格系统，栅格系统将浏览器分隔成一定数量的行和列。默认栅格系统将浏览器窗口分为12列，开发者可以为元素设置其在对应设备尺寸中所占的列数。

### 二、均分与尺寸适配

    Bootstrap将浏览器尺寸分为4个等级，分别为xs，sm，md和lg。xs是指浏览器宽度小于768时的状态，一般对应移动手机设备，sm指浏览器宽度大于768且小于992时的状态，其一般对应平板设备，md指浏览器宽度大于768且小于1200时的状态，一般对应正常的个人电脑，lg是指浏览器宽度大于1200时的状态。如下表所示：

![](https://static.oschina.net/uploads/space/2016/1201/171021_lNnQ_2340880.png)

在开发者使用栅格类对标签进行定义的时候，需要注意，如果只设置了高等级的栅格类，则在此等级以下的浏览器尺寸都将采用竖直堆叠，此等级及以上等级的浏览器尺寸中都将水平排列。例如，如果配置了两个标签的类都为为col-md-6，则在992以下尺寸的的浏览器中竖直堆叠布局，在992即以上尺寸的浏览器中都将水平均分一行。

    栅格系统的一行中被分成了12列，默认一行中也最多可以添加12个列，如下代码演示了竖直堆叠布局与水平布局在栅格系统中的应用：

```html
    <body class="container">
        <p>将md以上尺寸窗口宽度分为12份，md一下尺寸的窗口将竖直堆叠排列</p>
        <div class="row">
            <div class="col-md-1">.col-md-1</div>
            <div class="col-md-1">.col-md-1</div>
            <div class="col-md-1">.col-md-1</div>
            <div class="col-md-1">.col-md-1</div>
            <div class="col-md-1">.col-md-1</div>
            <div class="col-md-1">.col-md-1</div>
            <div class="col-md-1">.col-md-1</div>
            <div class="col-md-1">.col-md-1</div>
            <div class="col-md-1">.col-md-1</div>
            <div class="col-md-1">.col-md-1</div>
            <div class="col-md-1">.col-md-1</div>
            <div class="col-md-1">.col-md-1</div>
        </div>
        <hr />
        <p>将md以上尺寸的窗口宽度进行2等分，md一下尺寸的窗口将竖直堆叠排列</p>
        <div class="row">
            <div class="col-md-6">.col-md-6</div>
            <div class="col-md-6">.col-md-6</div>
        </div>
        <hr />
        <p>将md以上尺寸的窗口宽度进行2:1等分，md一下尺寸的窗口将竖直堆叠排列</p>
        <div class="row">
            <div class="col-md-8">.col-md-8</div>
            <div class="col-md-4">.col-md-4</div>
        </div>
        <hr />
        <p>将xs以上尺寸的窗口宽度进行6等分，xs为最小等级</p>
        <div class="row">
            <div class="col-xs-2">.col-xs-2</div>
            <div class="col-xs-2">.col-xs-2</div>
            <div class="col-xs-2">.col-xs-2</div>
            <div class="col-xs-2">.col-xs-2</div>
            <div class="col-xs-2">.col-xs-2</div>
            <div class="col-xs-2">.col-xs-2</div>
        </div>
    </body>
```

上面代码在窗口尺寸大于等于992时的效果如下所示：

![](https://static.oschina.net/uploads/space/2016/1201/175411_g7zz_2340880.png)

将浏览器窗口缩小，可以看到，除了第4行可以继续保持6等分外，其它行等变成了竖直堆叠布局：

![](https://static.oschina.net/uploads/space/2016/1201/175610_8LXe_2340880.png)

如果需要对移动设备和桌面是被进行布局的区别化，可以为某个标签配置多套不同等级下的栅格类，示例如下：

```html
        <p>在md及以上尺寸窗口中进行4等分，在md以下尺寸sm以上尺寸窗口进行2等分布局，在sm以下储存窗口进行竖直堆叠布局</p>
        <div class="row">
            <div class="col-md-3 col-sm-6">.col-md-3 .col-sm-6</div>
            <div class="col-md-3 col-sm-6">.col-md-3 .col-sm-6</div>
            <div class="col-md-3 col-sm-6">.col-md-3 .col-sm-6</div>
            <div class="col-md-3 col-sm-6">.col-md-3 .col-sm-6</div>
        </div>
```

需要注意，默认Bootstrap中一行最多可以包含12列，如果列数超出12，将另起一行进行布局，示例如下：

```html
        <p>Bootstrap最多一行可以分配12列，超出将另起一行，例如下面三个div，宽度分别为8，3，4，第3个div将另起一行布局</p>
        <div class="row">
            <div class="col-md-8">.col-md-8</div>
            <div class="col-md-3">.col-md-3</div>
            <div class="col-md-4 ">.col-md-4</div>
        </div>
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/1201/180807_Zgks_2340880.png)

### 三、列的调整

    很多场景下，每列元素的高度并不一定均等，在列高度不均等情况下的栅格布局，很可能会出现开发者意想不到的布局差错，示例代码如下：

```html
        <p>列高度不均等造成的布局错乱</p>
        <div class="row">
            <div class="col-md-6">.col-md-4 内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容</div>
            <div class="col-md-6">.col-md-4</div>
            <div class="col-md-4 ">.col-md-4</div>
        </div>
```

上面代码的栅格布局效果如下：

![](https://static.oschina.net/uploads/space/2016/1201/182336_aSc2_2340880.png)

如图所示，开发者本意是将第3个div另起一行进行布局，由于前两个div高度的不均等，导致第3个div直接布局在了第2个div下面，可以通过visible-md-block等类来进行强制另起一行，示例如下：

```html

        <p>解决列高度不均等造成的布局错乱</p>
        <div class="row">
            <div class="col-md-6">.col-md-4 内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容</div>
            <div class="col-md-6">.col-md-4</div>
            <div class="clearfix visible-md-block"></div>
            <div class="col-md-4 ">.col-md-4</div>
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1201/182904_vluP_2340880.png)

在使用栅格布局时，开发者也不需要将每一行中的12列都占满，可以通过列偏移设置来进行列的定位，示例如下：

```html
        <p>进行列偏移操作</p>
        <p>将占1/3行的一列向右便宜1/3行 使其固定在中间</p>
        <div class="row">
            <div class="col-md-4 col-md-offset-4">.col-md-4 .col-md-offset-4</div>
        </div>
        <div class="row">
            <div class="col-md-2">.col-md-2</div>
            <div class="col-md-2 col-md-offset-8">.col-md-2 .col-md-offset-8</div>
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1201/184732_CDUs_2340880.png)

Bootstrap的栅格系统也支持进行列的嵌套，需要注意，在嵌套中也不可以超过12列，示例如下：

```html
        <p>进行列的嵌套</p>
        <div class="row">
            <div class="col-md-8">.col-md-8
                <div class="row">
                    <div class="col-md-6">.col-md-6</div>
                    <div class="col-md-6">.col-md-6</div>
                </div>
            </div>
        </div>
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1201/185935_9JVV_2340880.png)

.col-md-push-* 和 .col-md-pull-*两个类可以方便的实现对列的移动，示例如下：

```html
        <p>进行列的移动</p>
        <div class="row">
            <div class="col-md-2 col-md-push-8">.col-md-8 向右移动8格</div>
            <div class="col-md-2 col-md-pull-2">.col-md-2 向左移动2格</div>
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1201/191556_CtgZ_2340880.png)

    另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/gridSystem.html](http://zyhshao.github.io/bootStrapDemo/gridSystem.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

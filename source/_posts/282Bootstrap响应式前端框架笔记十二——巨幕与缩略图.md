---
title: Bootstrap响应式前端框架笔记十二——巨幕与缩略图
date: 2016-12-14
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记十二——巨幕与缩略图

    巨幕用于创建一块区域，此区域可以用来展示网页页头或者需要重点提示的地方，使用jumbotron类来创建巨幕，示例如下：

```html
        <p>巨幕演示</p>
        <div class="jumbotron">
            <h1>勿忘国耻！九一八！</h1>
            <p>九一八事变（又称奉天事变、柳条湖事件）是日本在中国东北蓄意制造并发动的一场侵华战争，是日本帝国主义侵华的开端。1931年9月18日夜，在日本关东军安排下，铁道“守备队”炸毁沈阳柳条湖附近日本修筑的南满铁路路轨，并栽赃嫁祸于中国军队。日军以此为借口，炮轰沈阳北大营，是为“九一八事变”。</p>
            <p><a class="btn btn-primary btn-lg">查看详情</a></p>
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1214/154139_jlKt_2340880.png)

除了使用巨幕，开发者也可以使用page-header类来创建页头，示例如下：

```html
        <p>页头演示</p>
        <div class="page-header">
            <h1>前事不忘，后事之师！<small>祭奠南京大屠杀中遇难的三十万同胞！</small></h1>
        </div>
        <p>南京大屠杀指抗日战争期间，中国当时的首都南京于1937年12月13日沦陷后，日军在南京及附近地区进行长达四十多天的大屠杀[1]  。日军在南京城内对大量平民及战俘无恶不作。南京大屠杀的死亡人数超过30万。</p>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1214/160426_EPYP_2340880.png)

    Bootstrap中的缩略图也十分容易创建，使用thumbnail类可以将图片元素创建为缩略图样式，如下:

```html
        <p>缩略图</p>
        <div class="row">
            <div class="col-xs-6 col-md-3">
                <a href="#" class="thumbnail">
                    <img src="image/test.png">
                </a>
            </div>
        </div>
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1214/162008_JdQ9_2340880.png)

缩略图组件中也可以添加其他附件，例如标题，段落，按钮等，示例如下：

```html
        <p>缩略图也可以添加一些附件</p>
        <div class="row">
            <div class="col-sm-6 col-md-4">
                <div class="thumbnail">
                    <img src="image/nanjing.png">
                    <div class="caption">
                        <h3>国家公祭</h3>
                        <p>南京大屠杀指抗日战争期间，中国当时的首都南京于1937年12月13日沦陷后，日军在南京及附近地区进行长达四十多天的大屠杀[1] 。日军在南京城内对大量平民及战俘进行屠杀、抢掠、强奸、无恶不作。南京大屠杀的死亡人数超过30万。</p>
                        <p>
                            <a href="#" class="btn btn-primary" role="button">网上献花</a>
                            <a href="#" class="btn btn-default" role="button">更多史料</a>
                        </p>
                    </div>
                </div>
            </div>
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1214/162612_ik7W_2340880.png)

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/screen.html](http://zyhshao.github.io/bootStrapDemo/screen.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

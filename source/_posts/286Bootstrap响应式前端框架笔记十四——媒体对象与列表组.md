---
title: Bootstrap响应式前端框架笔记十四——媒体对象与列表组
date: 2016-12-20
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记十四——媒体对象与列表组

    在移动开发中经常会使用到列表，使用媒体对象可以方便的创建列表中每一行元素，常规的媒体对象实例如下：

```html
        <p>常规的媒体对象</p>
        <div class="media">
            <div class="media-left">
                <img src="image/icon.png" />
            </div>
            <div class="media-body">
                <h4 class="media-heading">社科院：经济不会发生硬着陆 警惕高房价对消费挤出效应</h4>
                从2010年开始中国经济增速逐年下滑，到2015年经济增速为6.9%，这是从1990年以来最低的增速，也是从改革开放以来首次连续六年经济增速下滑。由于今年前三季度经济增速为6.7%，北京大学国家发展研究院名誉院长、国务院参事林毅夫在18日举行的第一届国家发展论坛上表示， 2016年的经济增速还会继续下行，低于6.9%。
            </div>
         </div>
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1220/152155_QXCl_2340880.png)

使用media-middle类与media-bottom类可以设置媒体对象居中或者底部对齐，示例如下：

```html
        <p>媒体对象居中显示</p>
        <div class="media">
            <div class="media-left media-middle">
                <img src="image/icon.png" />
            </div>
            <div class="media-body">
                <h4 class="media-heading">社科院：经济不会发生硬着陆 警惕高房价对消费挤出效应</h4> 从2010年开始中国经济增速逐年下滑，到2015年经济增速为6.9%，这是从1990年以来最低的增速，也是从改革开放以来首次连续六年经济增速下滑。由于今年前三季度经济增速为6.7%，北京大学国家发展研究院名誉院长、国务院参事林毅夫在18日举行的第一届国家发展论坛上表示， 2016年的经济增速还会继续下行，低于6.9%。
            </div>
        </div>
        <br />
        <p>媒体对象底部对齐</p>
        <div class="media">
            <div class="media-left media-bottom">
                <img src="image/icon.png" />
            </div>
            <div class="media-body">
                <h4 class="media-heading">社科院：经济不会发生硬着陆 警惕高房价对消费挤出效应</h4> 从2010年开始中国经济增速逐年下滑，到2015年经济增速为6.9%，这是从1990年以来最低的增速，也是从改革开放以来首次连续六年经济增速下滑。由于今年前三季度经济增速为6.7%，北京大学国家发展研究院名誉院长、国务院参事林毅夫在18日举行的第一届国家发展论坛上表示， 2016年的经济增速还会继续下行，低于6.9%。
            </div>
        </div>
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1220/152847_VpIG_2340880.png)

    在实际开发中，列表组的应用也十分广泛，Bootstrap中定义的列表组样式十分灵活，开发者可以灵活的对其进行自定义操作，示例如下：

```html
        <p>列表组示例</p>
        <ul class="list-group">
            <li class="list-group-item list-group-item-info">
                未读消息
                <span class="badge">2</span>
            </li>
            <li class="list-group-item list-group-item-success">
                个人中心
            </li>
            <li class="list-group-item list-group-item-danger">
                退出登录
            </li>
            <li class="list-group-item list-group-item-warning">
                <div class="list-group-item-heading">温馨提示</div>
                <small>更多功能请进个人中心</small>
            </li>
        </ul>
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1220/153806_NrmF_2340880.png)

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/medioAndListGroup.html](http://zyhshao.github.io/bootStrapDemo/medioAndListGroup.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

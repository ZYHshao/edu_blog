---
title: Bootstrap响应式前端框架笔记十三——警告框与进度条
date: 2016-12-19
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记十三——警告框与进度条

    在Bootstrap中，使用alert相关类可以实现简洁的警告框控件，示例如下：

```html
        <p>alert相关类可以实现简洁的警告框样式</p>
        <div class="alert alert-success">成功风格的警告框</div>
        <div class="alert alert-info">详情风格的警告框</div>
        <div class="alert alert-warning">警告风格的警告框</div>
        <div class="alert alert-danger">危险风格的警告框</div>
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1219/142649_W5Hf_2340880.png)

警告框上面也可以添加有一个关闭按钮，示例如下：

```html
        <p>带关闭按钮的警告框</p>
        <div class="alert alert-warning alert-dismissible">可关闭的警告框
        <button type="button" class="close">
            <span aria-hidden="true">&times;</span>
        </button>
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1219/143641_9T0W_2340880.png)

警告框中也可以添加跳转链接，示例如下：

```html
        <p>带链接的警告框</p>
        <div class="alert alert-danger">
            您输入的用户名或密码有误
            <a class="alert-link" href="#">找回密码</a>
        </div>
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1219/144054_crv5_2340880.png)

    关于进度条组件，Bootstrap中使用progress类来创建，示例如下：

```html
        <p>默认的进度条组件</p>
        <div class="progress">
            <div class="progress-bar" style="width: 60%;">
            </div>
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1219/144555_Os4j_2340880.png)

进度条组件也支持多种样式，示例如下：

```html
        <p>各种风格的进度条组件</p>
        <div class="progress">
            <div class="progress-bar progress-bar-danger" style="width: 60%;">
                60%
            </div>
        </div>
        <div class="progress">
            <div class="progress-bar progress-bar-info" style="width: 60%;">
                60%
            </div>
        </div>
        <div class="progress">
            <div class="progress-bar progress-bar-success" style="width: 60%;">
                60%
            </div>
        </div>
        <div class="progress">
            <div class="progress-bar progress-bar-success" style="width: 60%;">
                60%
            </div>
        </div>
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/1219/145112_vPpO_2340880.png)

进度条也支持条纹模式，使用progress-bar-striped类可以创建条纹进度条，添加active类可以展现条纹动画，示例如下：

```html
        <p>带条纹的进度条</p>
        <div class="progress">
            <div class="progress-bar progress-bar-success progress-bar-striped active" style="width: 60%;">
                60%
            </div>
        </div>
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/1219/145505_oYys_2340880.png)

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/alertAndProgress.html](http://zyhshao.github.io/bootStrapDemo/alertAndProgress.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

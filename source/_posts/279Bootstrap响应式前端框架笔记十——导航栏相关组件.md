---
title: Bootstrap响应式前端框架笔记九——输入框组
date: 2016-12-11
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记九——输入框组

    将input标签与input-group-addon类组合使用可以为输入框添加前后挂件，需要注意，Bootstrap不支持在输入框控件一侧添加多个挂件，示例如下：

```html
        <p>输入框的前后可以添加额外的标题元素</p>
        <div class="input-group form-group">
            <span class="input-group-addon">邮箱</span>
            <input type="text" class="form-control" placeholder="邮箱">
        </div>

        <div class="input-group form-group">
            <input type="text" class="form-control">
            <span class="input-group-addon">平米</span>
        </div>

        <div class="input-group form-group">
            <span class="input-group-addon">余额</span>
            <input type="text" class="form-control">
            <span class="input-group-addon">.00</span>
        </div>

```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1211/163758_gTjU_2340880.png)

    也可以将输入框组合为选择控件，示例如下：

```html
        <p>将输入框组合为选择组件</p>
        <div class="input-group form-group">
            <span class="input-group-addon">
                <input type="checkbox">
            </span>
            <input type="text" class="form-control">
        </div>
        <div class="input-group form-group">
            <span class="input-group-addon">
                <input type="radio">
            </span>
            <input type="text" class="form-control">
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1211/164142_0y1i_2340880.png)

    在输入框的前后，也可以添加功能按钮，示例如下：

```html
        <p>为输入框添加功能按钮</p>
        <div class="input-group form-group">
            <span class="input-group-btn">
                <button class="btn btn-default" type="button">星座</button>
            </span>
            <input type="text" class="form-control">
        </div>
        <div class="input-group form-group">
            <input type="text" class="form-control">
            <span class="input-group-btn">
                <button class="btn btn-primary" type="button">前往</button>
            </span>
        </div>
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/1211/164842_hEG6_2340880.png)

    在输入框组件中，也可以与下拉菜单进行嵌套使用，示例如下：

```html
        <p>在输入框组件中嵌套下拉菜单组件</p>
        <div class="input-group">
            <div class="input-group-btn">
                <button type="button" class="btn btn-default dropdown-toggle">星座 <span class="caret"></span></button>
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
            <input type="text" class="form-control">
        </div>
        <div class="input-group">
            <input type="text" class="form-control">
            <div class="input-group-btn">
                <button type="button" class="btn btn-default">金牛</button>
                <button class="btn btn-default dropdown-toggle"><span class="caret"></span></button>
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
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1211/170228_XeUG_2340880.png)

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/inputGroup.html](http://zyhshao.github.io/bootStrapDemo/inputGroup.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

---
title: Bootstrap响应式前端框架笔记十七——下拉列表交互
date: 2016-12-27
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记十七——下拉列表交互

    为dropdown-toggle类添加data-toggle="dropdown"属性可以实现其下拉列表功能的绑定，示例如下：

```html
<div class="dropdown col-md-2" id="drop">
    <button class="btn dropdown-toggle btn-primary" data-toggle="dropdown">
           下拉列表
           <span class="caret"></span>
      </button>
    <ul class="dropdown-menu">
        <li>
            <a>金牛座</a>
        </li>
        <li>
            <a>摩羯座</a>
        </li>
        <li>
            <a>狮子座</a>
        </li>
        <li>
            <a>白羊座</a>
        </li>
    </ul>
</div>
```

点击此按钮后，可以自动实现下拉列表的显示或隐藏。

    Bootstrap中的下拉列表控件也定义了一些方法回调，其会在drop-menu的父标签上进行绑定，示例如下：

```javascript
$('#myDropMenu').on('show.bs.dropdown',function(){
    console.log("下拉框将要展示");
});
$('#myDropMenu').on('shown.bs.dropdown',function(){
    console.log("下拉框已经展示");
});
$('#myDropMenu').on('hide.bs.dropdown',function(){
    console.log("下拉框将要隐藏");
});
$('#myDropMenu').on('hidden.bs.dropdown',function(){
    console.log("下拉框已经展示");
});

```

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/dropdownJS.html](http://zyhshao.github.io/bootStrapDemo/dropdownJS.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

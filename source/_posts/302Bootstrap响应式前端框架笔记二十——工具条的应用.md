---
title: Bootstrap响应式前端框架笔记二十——工具条的应用
date: 2017-01-05
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记二十——工具条的应用

    工具条用于向用户进行某个操作的提示，在Bootstrap框架中，为按钮添加工具条十分简单，只需如下代码：

```html
<button class="btn btn-primary" data-toggle="tooltip" data-placement="top" id="btn">工具条</button>
```

需要注意，要使工具条显现，必须先进行工具条对象的构造，示例如下：

```javascript
$("#btn").tooltip({
    animation:false,
    delay:1000,
    placement:'top',
    title:'标题！！！！',
    trigger:'click'
});
```

这个方法中可以传入一个对象参数，其中animation属性设置工具条的显隐是否有动画效果；delay属性设置触发后延迟多久进行工具条的显隐操作；placement属性设置工具条出现的位置，可选top，bottom，left，right，auto选项；title属性设置工具栏标题；trigger属性设置触发方式，可选click，hover，focus和manual。

工具条效果如下：

![](https://static.oschina.net/uploads/space/2017/0105/155105_nkGW_2340880.png)

    开发者也可以手动调用方法来控制工具条的显示隐藏，示例如下：

```javascript
//显示工具栏
$('#show').on('click',function(){
    $('#btn').tooltip('show');
});
//隐藏工具栏
$('#hide').on('click',function(){
    $('#btn').tooltip('hide');
});
//切换显隐状态
$('#toggle').on('click',function(){
    $('#btn').tooltip('toggle');
});
```

Bootstrap中还对工具条提供了一些状态监听方法，示例如下：

```javascript
$('#btn').on('show.bs.tooltip',function(){
    console.log("工具条将要开启");
});
$('#btn').on('shown.bs.tooltip',function(){
    console.log("工具条已经开启");
});
$('#btn').on('hide.bs.tooltip',function(){
    console.log("工具条将要关闭");
});
$('#btn').on('hidden.bs.tooltip',function(){
    console.log("工具条已经关闭");
});
```

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/toolJS.html](http://zyhshao.github.io/bootStrapDemo/toolJS.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

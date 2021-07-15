---
title: Bootstrap响应式前端框架笔记十六——模态框交互
date: 2016-12-26
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记十六——模态框交互

    模态框也可以称为弹出窗，其作用是当用户点击某个功能按钮后，在网页上弹出一个内容窗口。在Bootstrap中，创建模态框十分简单。首先模态框组件通过modal类来创建，其默认隐藏，开发者可以使用data相关属性来将其唤出。简单示例如下：

```html
<button type="button" class="btn btn-primary btn-lg" data-toggle="modal" data-target="#myModal">
正常模态框
</button>
<!--这里设置的id用于绑定在按钮事件上-->
<div class="modal fade" id="myModal" tabindex="-1">
    <!--modal-dialog为悬浮框模式的模态框-->
    <div class="modal-dialog">
        <!--模态框组件内容部分-->
        <div class="modal-content">
            <!--头部内容-->
            <div class="modal-header">
                <!--为按钮绑定data-dismiss="modal"可以实现点击取消模态框-->
                <button type="button" class="close" data-dismiss="modal"><span>&times;</span>
                </button>
                <h4 class="modal-title" id="myModalLabel">模态框标题</h4>
            </div>
            <!--模态框主体内容-->
            <div class="modal-body">
            模态框内容
            </div>
            <!--模态框尾部内容-->
            <div class="modal-footer">
                <button type="button" class="btn btn-default" data-dismiss="modal">关闭</button>
                <button type="button" class="btn btn-primary" data-dismiss="modal">保存</button>
            </div>
        </div>
    </div>
</div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1226/185232_Xecz_2340880.png)

可以为model-dialog类增加modal-lg或者modal-sm可以创建大号的模态框或者小号的模态框。需要注意，模态框的弹出时有渐入动画的，如果不需要动画效果，只需要将fade类去掉即可。

    对于定义为modal模块的div，开发者也可以通过设置data属性的方式来对模态框进行设置，列举如下：

| data-backdrop  | 布尔"true"或"false"  |  如果设置为true，则显示灰色背景，否则不显示灰色背景 |
|     data-keyboard | 布尔值 | 设置点击键盘esc键是否隐藏模态框，注意，必须设置tabindex属性，这个值才有效 |
| data-show | 布尔值 | 模态框初始化后是否立即展示 |
| data-remote | 路径     | 如果配置了url，会将内容加载进modal-content中 |

modal模块也支持通过js代码来进行相关控制，支持的方法如下：

```javascript
    
$('#open').on("click",function(){
    //展示模态框
    $('#myModal').modal('show');
});
$('#close').on("click",function(){
    //隐藏模态框
    $('#myModal').modal('hide');
});
$('#exchange').on("click",function(){
    //显示或隐藏进行切换
    $('#myModal').modal('toggle');
});
$('#setting').on("click",function(){
    //对模态框的属性进行设置 传入对象
    $('#myModal').modal({
        keyboard:false
    });
});
```

模态框也可以添加一些特有的事件回调，示例如下：

```javascript
$('#myModal').on('show.bs.modal',function(e){
    console.log("模态框将要展示触发")
});
$('#myModal').on('shown.bs.modal',function(e){
    console.log("模态框已经展示后触发")
});
$('#myModal').on('hide.bs.modal',function(e){
    console.log("模态框将要消失触发")
});
$('#myModal').on('hidden.bs.modal',function(e){
    console.log("模态框已经消失后触发")
});
$('#myModal').on('loaded.bs.modal',function(e){
    console.log("从远端数据源加载数据完成")
});
```

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/modelJS.html](http://zyhshao.github.io/bootStrapDemo/modelJS.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

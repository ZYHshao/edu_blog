---
title: Bootstrap响应式前端框架笔记十九——标签页的使用
date: 2017-01-02
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记十九——标签页的使用

    Bootstrap中通过为导航标签增加data-toggle="tab"，配合类或id来进行标签页的关联，示例如下：

```html
        <ul class="nav nav-tabs">
            <li class="active">
                <a href="#one" data-toggle="tab" id="aone">第一页</a>
            </li>
            <li>
                <a href="#two" data-toggle="tab" id="atwo">第二页</a>
            </li>
            <li>
                <a href="#three" data-toggle="tab" id="athree">第三页</a>
            </li>
            <li>
                <a href="#four" data-toggle="tab" id="afour">第四页</a>
            </li>
        </ul>

        <!-- Tab panes -->
        <div class="tab-content">
            <div class="tab-pane active fade in" id="one">On iPhone 7 and iPhone 7 Plus, haptics provide additional ways to physically engage users with tactile feedback that gets attention and reinforces actions. Some system-provided interface elements, such as pickers, switches, and sliders, automatically provide haptic feedback as users interact with them</div>
            <div class="tab-pane" id="two">Using one of the concrete subclasses, you ask the system to generate haptics for a specific scenario and iOS manages the strength and behavior of the feedback based on the scenario you choose. In addition, you can call the prepare method of UIFeedbackGenerator to inform the system that haptic feedback is about to be required and to minimize latency. To learn how to use haptics to provide the best user experience in your app, see “Haptic Feedback” in iOS Human Interface Guidelines.

</div>
            <div class="tab-pane" id="three">When the user makes a request involving your service, SiriKit sends your extension an intent object, which describes the user’s request and provides any data related to that request. You use the intent object to provide an appropriate response object, which includes details of how you can handle the user’s request. Siri typically handles all user interactions, but you can use an extension to provide custom UI that incorporates branding or additional information from your app.</div>
            <div class="tab-pane" id="four">To learn how to support SiriKit and give users new ways to access your services, read SiriKit Programming Guide. When you’re ready to implement the app extensions that handle various intents, see Intents Framework Reference and Intents UI Framework Reference.

</div>
        </div>
```

效果如图：

![](https://static.oschina.net/uploads/space/2017/0102/172314_cqur_2340880.png)

为tab-pane类增加fade in类可以使标签切换时带渐入动画。

    Bootstrap中的标签页JS组件提供了一个tab函数，使用这个方法可以实现代码控制标签的切换，示例如下：

```html
        <button class="btn btn-primary" id="cone">第一页</button>
        <button class="btn btn-primary" id="ctwo">第二页</button>
        <button class="btn btn-primary" id="cthree">第三页</button>
        <button class="btn btn-primary" id="cfour">第四页</button>
        <script>
            $("#cone").on("click",function(){
                $("#aone").tab('show');
            });
            $("#ctwo").on("click",function(){
                $("#atwo").tab('show');
            });
            $("#cthree").on("click",function(){
                $("#athree").tab('show');
            });
            $("#cfour").on("click",function(){
                $("#afour").tab('show');
            });
        </script>
```

Bootstrap中还提供了一些监听事件，开发者可以向导航标签中添加这些监听事件来监听标签页的状态，示例如下：

```javascript
            $("#aone").on("show.bs.tab",function(){
                console.log("此标签页将要显示");
            });
            $("#aone").on("shown.bs.tab",function(){
                console.log("此标签页已经显示");
            });
            $("#aone").on("hide.bs.tab",function(){
                console.log("此标签页将要隐藏");
            });
            $("#aone").on("hidden.bs.tab",function(){
                console.log("此标签页已经隐藏");
            });
```

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/PageJS.html](http://zyhshao.github.io/bootStrapDemo/PageJS.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

---
title: Bootstrap响应式前端框架笔记十八——导航滚动监听
date: 2016-12-29
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记十八——导航滚动监听

    Bootstrap框架中提供了十分方便的方法来使用导航关联内容快，并且开发者可以监听滚动进行导航按钮的切换，示例如下：

```html
<!DOCTYPE html>
<html>

    <head>
        <meta charset="UTF-8">
        <link rel="stylesheet" href="../bower_components/bootstrap/dist/css/bootstrap.css" />
        <script type="text/javascript" src="../bower_components/jquery/dist/jquery.js"></script>
        <script type="text/javascript" src="../bower_components/bootstrap/dist/js/bootstrap.js"></script>
        <style>
            .scrollspy-example {
                position: relative;
                height: 200px;
                margin-top: 10px;
                overflow: auto
            }
        </style>
        <title>滚动监听</title>
    </head>

    <body class="container">
        <br />
        <br />
        <!--导航栏-->
        <nav id="navbar" class="navbar navbar-default navbar-static">
            <div class="navbar-header">
                <button class="navbar-toggle" type="button" data-toggle="collapse" data-target=".navigationShow">
                    <span class="icon-bar"></span>
                        <span class="icon-bar"></span>
                        <span class="icon-bar"></span>
                </button>
                <a class="navbar-brand" href="#">列表</a>
            </div>
            <!--导航容器-->
            <div class="navbar-collapse navigationShow">
                <!--导航-->
                <ul class="nav navbar-nav">
                    <li class="active">
                        <a href="#One">第一项</a>
                    </li>
                    <li class="">
                        <a href="#Two">第二项</a>
                    </li>
                    <li class="dropdown">
                        <a href="#" id="navbarDrop1" class="dropdown-toggle" data-toggle="dropdown">菜单 <span class="caret"></span></a>
                        <ul class="dropdown-menu">
                            <li class="">
                                <a href="#one">菜单1</a>
                            </li>
                            <li class="">
                                <a href="#two">菜单2</a>
                            </li>
                            <li class="divider"></li>
                            <li class="">
                                <a href="#three">菜单3</a>
                            </li>
                        </ul>
                    </li>
                </ul>
            </div>
        </nav>
        <!--内容div 使用data-spy="scroll"来进行滚动监听-->
        <div data-spy="scroll" data-target="#navbar" class="scrollspy-example">
            <!--id需要对应-->
            <h4 id="One">第一项</h4>
            <p>Ad leggings keytar, brunch id art party dolor labore. Pitchfork yr enim lo-fi before they sold out qui. Tumblr farm-to-table bicycle rights whatever. Anim keffiyeh carles cardigan. Velit seitan mcsweeney's photo booth 3 wolf moon irure. Cosby sweater lomo jean shorts, williamsburg hoodie minim qui you probably haven't heard of them et cardigan trust fund culpa biodiesel wes anderson aesthetic. Nihil tattooed accusamus, cred irony biodiesel keffiyeh artisan ullamco consequat.</p>
            <h4 id="Two">第二项</h4>
            <p>Veniam marfa mustache skateboard, adipisicing fugiat velit pitchfork beard. Freegan beard aliqua cupidatat mcsweeney's vero. Cupidatat four loko nisi, ea helvetica nulla carles. Tattooed cosby sweater food truck, mcsweeney's quis non freegan vinyl. Lo-fi wes anderson +1 sartorial. Carles non aesthetic exercitation quis gentrify. Brooklyn adipisicing craft beer vice keytar deserunt.</p>
            <h4 id="one">one</h4>
            <p>Occaecat commodo aliqua delectus. Fap craft beer deserunt skateboard ea. Lomo bicycle rights adipisicing banh mi, velit ea sunt next level locavore single-origin coffee in magna veniam. High life id vinyl, echo park consequat quis aliquip banh mi pitchfork. Vero VHS est adipisicing. Consectetur nisi DIY minim messenger bag. Cred ex in, sustainable delectus consectetur fanny pack iphone.</p>
            <h4 id="two">two</h4>
            <p>In incididunt echo park, officia deserunt mcsweeney's proident master cleanse thundercats sapiente veniam. Excepteur VHS elit, proident shoreditch +1 biodiesel laborum craft beer. Single-origin coffee wayfarers irure four loko, cupidatat terry richardson master cleanse. Assumenda you probably haven't heard of them art party fanny pack, tattooed nulla cardigan tempor ad. Proident wolf nesciunt sartorial keffiyeh eu banh mi sustainable. Elit wolf voluptate, lo-fi ea portland before they sold out four loko. Locavore enim nostrud mlkshk brooklyn nesciunt.</p>
            <h4 id="three">three</h4>
            <p>Ad leggings keytar, brunch id art party dolor labore. Pitchfork yr enim lo-fi before they sold out qui. Tumblr farm-to-table bicycle rights whatever. Anim keffiyeh carles cardigan. Velit seitan mcsweeney's photo booth 3 wolf moon irure. Cosby sweater lomo jean shorts, williamsburg hoodie minim qui you probably haven't heard of them et cardigan trust fund culpa biodiesel wes anderson aesthetic. Nihil tattooed accusamus, cred irony biodiesel keffiyeh artisan ullamco consequat.</p>
            <p>Keytar twee blog, culpa messenger bag marfa whatever delectus food truck. Sapiente synth id assumenda. Locavore sed helvetica cliche irony, thundercats you probably haven't heard of them consequat hoodie gluten-free lo-fi fap aliquip. Labore elit placeat before they sold out, terry richardson proident brunch nesciunt quis cosby sweater pariatur keffiyeh ut helvetica artisan. Cardigan craft beer seitan readymade velit. VHS chambray laboris tempor veniam. Anim mollit minim commodo ullamco thundercats.
            </p>
        </div>
    </body>
</html>
```

效果如下所示：

![](https://static.oschina.net/uploads/space/2016/1229/150915_p659_2340880.png)

    开发者也可以对导航标签的切换事件进行监听，方法如下：

```javascript

$('#navbar').on('activate.bs.scrollspy',function(e){
        console.log("滚动导航改变");
})
```

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/scrollJS.html](http://zyhshao.github.io/bootStrapDemo/scrollJS.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

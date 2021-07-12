---
title: Bootstrap响应式前端框架笔记三——代码与表格
date: 2016-12-05
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记三——代码与表格

### 一、代码

    在技术博客文章类页面的开发中，常常需要在文本总插入说明代码，使用code便签可以创建这种效果，示例如下：

```html
        <p>code标签用于在文本中插入代码</p>
        <div>定义变量a:<code>int a = 3; </code></div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1205/105926_Awh4_2340880.png)

kbd标签可以用来提示进行键盘输入，示例如下：

```html
        <p>kbd标签可以创建用户键盘输入的效果</p>
        <div>使用键盘上的<kbd>control</kbd>+<kbd>v</kbd>来进行文本的粘贴</div>
```

效果：

![](https://static.oschina.net/uploads/space/2016/1205/110240_Vc7X_2340880.png)

可以使用pre标签来进行成段代码的插入，同时可以使用pre-scrollable类来将代码块修饰为可滚动的，示例如下：

```html
        <pre class="pre-scrollable">
        &lt;head&gt;
            &lt;meta charset="UTF-8"&gt;
            &lt;link rel="stylesheet" href="../bower_components/bootstrap/dist/css/bootstrap.min.css" /&gt;
            &lt;title&lt;代码与表格&lt;/title&gt;
        &lt;/head&gt;
        </pre>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1205/111350_mzyu_2340880.png)

除了上面描述的标签和类外，一般情况下，程序中的变量会以斜体来显示，也可以使用var标签来包裹，程序输出结果可以使用samp标签来包裹。

### 二、表格

    为H5标签table添加table类可以使用Bootstrap定义的表格样式，示例如下:

```html
        <p>使用table标签添加table类可以进行表格的创建</p>
        <table class="table">
            <thead>学生表</thead>
            <tr>
                <th>班级</th>
                <th>姓名</th>
                <th>年龄</th>
            </tr>
            <tr>
                <th>3年1班</th>
                <th>jaki</th>
                <th>24</th>
            </tr>
        </table>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1205/113021_ViVS_2340880.png)

为表格添加tabke-striped类可以实现斑马纹样式的表格，示例如下：

```html
        <p>使用table-striped类可以为表格添加斑马纹</p>
        <table class="table table-striped">
            <thead>学生表</thead>
            <tr>
                <th>班级</th>
                <th>姓名</th>
                <th>年龄</th>
            </tr>
            <tr>
                <th>3年1班</th>
                <th>jaki</th>
                <th>24</th>
            </tr>
            <tr>
                <th>3年2班</th>
                <th>Annay</th>
                <th>22</th>
            </tr>
        </table>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1205/113356_0h6A_2340880.png)

Bootstrap默认的列表样式是不带边框的，可以使用table-bordered类来为列表添加边框，示例如下：

```html
        <p>使用table-boardered类可以为表格添加边框</p>
        <table class="table table-striped table-bordered">
            <thead>学生表</thead>
            <tr>
                <th>班级</th>
                <th>姓名</th>
                <th>年龄</th>
            </tr>
            <tr>
                <th>3年1班</th>
                <th>jaki</th>
                <th>24</th>
            </tr>
            <tr>
                <th>3年2班</th>
                <th>Annay</th>
                <th>22</th>
            </tr>
        </table>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1205/134908_YWRu_2340880.png)

使用table-hover类修饰的列表，当鼠标悬停时会有高亮效果，示例如下：

```html
        <p>使用table-hover类修饰的列表，当鼠标悬停时 会有高亮效果</p>
        <table class="table table-hover">
            <thead>学生表</thead>
            <tr>
                <th>班级</th>
                <th>姓名</th>
                <th>年龄</th>
            </tr>
            <tr>
                <th>3年1班</th>
                <th>jaki</th>
                <th>24</th>
            </tr>
            <tr>
                <th>3年2班</th>
                <th>Annay</th>
                <th>22</th>
            </tr>
        </table>
```

使用.table-condensed类可以是默认的列表padding减半。

    对于行标签tr与列表前th，开发者也可以使用如下类来修饰，为其指定状态：

.active类：将此行或者此列标记为高亮状态。

.success类：将此行或者此列标记为成功状态。

.info类：将此行或者此列标记为详情状态。

.warning类：将此行或者此列标记为警告状态。

.danger类：将此行或者此列标记为危险状态。

示例代码如下：

```html
        <p>为列表设置状态</p>
        <table class="table table-hover table-condensed">
            <thead>学生表</thead>
            <tr>
                <th>班级</th>
                <th>姓名</th>
                <th>年龄</th>
            </tr>
            <tr class="active">
                <th>3年1班</th>
                <th>jaki</th>
                <th>24</th>
            </tr>
            <tr class="success">
                <th>3年2班</th>
                <th>Annay</th>
                <th>22</th>
            </tr>
            <tr class="info">
                <th>3年1班</th>
                <th>CJ</th>
                <th>19</th>
            </tr>
            <tr class="warning">
                <th>3年1班</th>
                <th>jaki</th>
                <th>24</th>
            </tr>
            <tr>
                <th>3年2班</th>
                <th class="danger">Annay</th>
                <th>22</th>
            </tr>
        </table>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1205/140451_E0CQ_2340880.png)

列表元素也可以包裹在table-responsive类内，此时列表会变成响应式列表，当屏幕尺寸小于768px时，会自动出现水平滚动条。

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/codeAndGroup.html](http://zyhshao.github.io/bootStrapDemo/codeAndGroup.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

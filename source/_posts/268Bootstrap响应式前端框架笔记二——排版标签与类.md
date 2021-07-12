---
title: Bootstrap响应式前端框架笔记二——排版标签与类
date: 2016-12-03
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记二——排版标签与类

    Bootstrap中对h标签的字体和字号进行了微调，开发者除了可以直接使用这些标签进行标题的修饰外，还可以使用.h1到.h6类来将其他元素的字体进行修饰，示例如下：

```html
        <p>h1 到 h6 标签的样式</p>
        <h1>h1. Bootstrap heading</h1>
        <h2>h2. Bootstrap heading</h2>
        <h3>h3. Bootstrap heading</h3>
        <h4>h4. Bootstrap heading</h4>
        <h5>h5. Bootstrap heading</h5>
        <h6>h6. Bootstrap heading</h6>
        <hr />
        <p>.h1 到 .h6 类的样式</p>
        <div class="h1">h1. Bootstrap heading</div>
        <div class="h2">h1. Bootstrap heading</div>
        <div class="h3">h1. Bootstrap heading</div>
        <div class="h4">h1. Bootstrap heading</div>
        <div class="h5">h1. Bootstrap heading</div>
        <div class="h6">h1. Bootstrap heading</div>
```

    在标题或者其他标签中使用small标签或者small类可以添加内部副标题，副标题除了字号会进行缩小调整外，还会修改文字的颜色，示例如下：

```html
        <p>可以使用small标签或者.samll类来向标题中添加副标题</p>
        <h3>h3标题 <small>small标签副标题</small></h3>
        <span class="h3">h3Class类 <span class="small">small类副标题</span></span>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1203/105928_G2I1_2340880.png)

    使用.lead可以实现段落的强调显示，示例如下：

```html
        <p>这是一个普通段落</p>
        <p class="lead">这是一个强调段落</p>
        <p>这是一个普通段落</p>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1203/110521_ayZ0_2340880.png)

    使用mark标签或者mark类可以进行特殊文本的标记，如下：

```html
        <p>使用mark标签可以实现部分文本进行标记</p>
        <div class="mark">进行<mark>特殊文字</mark>的标记</div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1203/110917_ILGy_2340880.png)

    使用del标签或者s标签可以实现对文本添加删除线效果，如下：

```html
        <p>使用del标签或者s标签可以实现文本的删除效果</p>
        <del>del标签的删除效果</del>
        <br />
        <s>s标签的删除效果</s>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1203/111332_FHEw_2340880.png)

    使用ins标签或者u标签可以实现为本文添加下划线效果，示例如下：

```html
        <p>使用ins标签或者u标签可以实现文本添加下划线</p>
        <ins>ins标签的下划线效果</ins>
        <br />
        <u>u标签的下划线效果</u>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1203/111635_w52X_2340880.png)

    使用strong标签可以对特殊本文进行着重标记，如下:

```html
        <p>使用strong标签可以实现对特殊文本进行着重标记</p>
        <div>进行文本的<strong>着重</strong>标记</div>
```

效果如下图

![](https://static.oschina.net/uploads/space/2016/1203/111955_Nc24_2340880.png)

    使用em标签可以进行特殊文本的斜体处理，如下：

```html
        <p>使用em标签可以进行文本的斜体处理</p>
        <p>进行<em>特殊文本</em>的斜体处理</p>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1203/112219_v22z_2340880.png)

    使用text-left类可以实现文本的左对齐布局，与之对应text-center将文本进行中心对齐布局，text-right类来将文本进行右对齐布局，text-justufy类设置文本进行自适应对齐，text-nowarp类将设置文本不换行的进行布局，示例如下：

```html
        <mark>text-left类进行左对齐布局</mark>
        <p class="text-left ">文本左对齐排版。文本左对齐排版。文本左对齐排版。文本左对齐排版。文本左对齐排版。文本左对齐排版。文本左对齐排版。文本左对齐排版。文本左对齐排版。文本左对齐排版。文本左对齐排版。文本左对齐排版。</p>
        <mark>text-center类进行中心对齐布局</mark>
        <p class="text-center ">文本居中对齐。文本居中对齐。文本居中对齐。文本居中对齐。文本居中对齐。文本居中对齐。文本居中对齐。文本居中对齐。文本居中对齐。文本居中对齐。文本居中对齐。文本居中对齐。</p>
        <mark>text-right类进行右对齐布局</mark>
        <p class="text-right ">文本右对齐。文本右对齐。文本右对齐。文本右对齐。文本右对齐。文本右对齐。文本右对齐。文本右对齐。文本右对齐。文本右对齐。文本右对齐。文本右对齐。</p>
        <mark>text-justify类进行自适应布局</mark>
        <p class="text-justify ">正常方向布局。正常方向布局。正常方向布局。正常方向布局。正常方向布局。正常方向布局。正常方向布局。正常方向布局。正常方向布局。正常方向布局。</p>
        <mark>text-nowarp类进行不换行布局</mark>
        <p class="text-nowrap">不换行布局。不换行布局。不换行布局。不换行布局。不换行布局。不换行布局。不换行布局。不换行布局。不换行布局。不换行布局。</p>
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/1203/113744_IA6Q_2340880.png)

        text-lowercase类可以将所有修饰的文本转换成小写，与之对应text-uppercase类可以将所有修饰的文本转换成大写，text-capitalize类则只会处理每个单词的首字母，将其转换为大写。示例如下：

```html
        <mark>将所有字母转换成小写字母</mark>
        <p class="text-lowercase">My name is Jaki.</p>
        <mark>将所有字母转换成大写字母</mark>
        <p class="text-uppercase">My name is Jaki.</p>
        <mark>将所有单词首字母字母转换成大写字母</mark>
        <p class="text-capitalize">My name is Jaki.</p>
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/1203/114359_ktqf_2340880.png)

    使用abbr标签可以进行某些内容的缩略显示，示例如下：

```html
        使用abbr标签可以将某些文本进行缩略设置，当鼠标放置在对应文本上时，会显示标签中title所设置的内容
        <abbr title="这个是详细信息">信息</abbr>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1203/114757_esE3_2340880.png)

    如果要在页面中进行内容的引用，可以使用blockquote标签进行包裹，在blockquote标签中可以继续嵌套footer标签来进行引用的标注，如下：

```html
        使用blockquote标签可以进行内容的引用，其中可以嵌套fooer标签进行标注
        <blockquote>
            <p>冰冻三尺，非一日之寒。</p>
            <footer>俗语</footer>
        </blockquote>
```

    效果如下图所示：

![](https://static.oschina.net/uploads/space/2016/1203/121122_6E1k_2340880.png)

    .blockquote-reverse类可以将blockquote中的内容进行右对齐，示例如下：

```html
        <hr /> 使用blockquote标签可以进行内容的引用，其中可以嵌套fooer标签进行标注
        <blockquote class="blockquote-reverse">
            <p>冰冻三尺，非一日之寒。</p>
            <footer>俗语</footer>
        </blockquote>
```

![](https://static.oschina.net/uploads/space/2016/1203/121853_Z4CY_2340880.png)

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/typeset.html](http://zyhshao.github.io/bootStrapDemo/typeset.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

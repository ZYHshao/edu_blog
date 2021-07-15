---
title: 配置Sublime Text工具运行JavaScript代码
date: 2016-12-27
categories: 小码工具
tags: []
---
## 配置Sublime Text工具运行JavaScript代码

    SublimeText是一款及强大的跨平台编辑器，其丰富的插件可以帮助开发者编写各种语言的代码。并且其自带控制台，开发者实现简单的配置即可在SublimeText控制台中进行代码的调试运行。

    在SublimeText中运行JavaScript代码十分简单，实现运行JavaScript代码需要借助node.js环境，首先需要安装node.js环境，node.js环境可以在如下网址进行下载安装：

[https://nodejs.org/en/](https://nodejs.org/en/)。

    安装完成node环境后，在终端输入如下命令获取node的路径：

```
which node
```

示例如下：

![](https://static.oschina.net/uploads/space/2016/1227/170131_XEwj_2340880.png)

打开SublimeText编辑器，在菜单中的Tools->Build System->New Build System，如下图：

![](https://static.oschina.net/uploads/space/2016/1227/170248_4xuM_2340880.png)

需要注意，图中的JavaScript是我配置完成后增加的，默认是无法运行JavaScript代码的，Build System中也不会有这一项。

在新建的文件中写入如下信息后进行保存：

```json
{  
    "cmd": ["/usr/local/bin/node", "$file"],  
    "selector": "source.js"  
}  
```

注意cmd数组中的第一个对象是刚才从终端查到的node路径，selector对应要编译的文件后缀。

进行保存，将其文件名命名为JavaScript(其实什么都可以)。

    新建一个SublimeText文件，将其保存为js文件，在其中编写JavaScript代码，使用command+B(Mac)即可进行JavaScript代码的运行，效果如下：

![](https://static.oschina.net/uploads/space/2016/1227/171142_IZvy_2340880.png)

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

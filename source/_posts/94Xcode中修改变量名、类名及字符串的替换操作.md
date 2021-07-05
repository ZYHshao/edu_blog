---
title: Xcode中修改变量名、类名及字符串的替换操作
date: 2015-08-05
categories: 代码优化
tags: []
---
## Xcode中修改变量名、类名及字符串的替换操作

        在做iOS开发代码优化的工作时，优化代码结构之前，我们应该先整理好工程的外貌，将文件和类的命名进行规范，在Xcode中为我们提供了方便而强大的名称修改功能。

第一步：修改类名

        将鼠标点击放在类的名称上，选择Xcode工具栏中的edit->refactor->rename：

![](http://static.oschina.net/uploads/space/2015/0805/092503_pdu5_2340880.png)

之后，将类名更改为我们需要的模式点击preview，记得将下面的关联文件勾选：

![](http://static.oschina.net/uploads/space/2015/0805/092803_OrUV_2340880.png)

Xcode会为我们检测出需要更改的地方，浏览无误后点击save。

第二步 修改相关字符串：

        通过第一步，我们的类的文件名，类名都已经更改，但并不全面，因为某些注释，字符串动态创建类对象以及类函数创建类对象时的类名并没有更改，我们需要做这一步，将更改前的类名在Xcode左侧的搜索栏中搜索：

        ![](http://static.oschina.net/uploads/space/2015/0805/093235_ZPfC_2340880.png)

        将Find改选为Replace：

![](http://static.oschina.net/uploads/space/2015/0805/093401_ilVH_2340880.png)

        这里面有四个选项，意义如下：

        Containing:检索出包涵检索条件的对象

        Matching:检索出等于检索条件的对象

        Start With：检索出以检索条件开头的对象

        Ending with:检索出以检索条件结尾的对象

我们选择Matching，进行检索，将检索出来的地方进行Replace替换，通过这一步，我们可以替换代码中的注释，字符串，类方法以及xib和StoryBoard文件中关联的id，cell复用符等。

第三步：修改文件中变量名

        在文件中，我们也可以通过command+F换出搜索框，将Find改选为Replace检索进行我们想要的变量替换。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

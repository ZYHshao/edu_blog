---
title: 一个移动开发者的Mock数据之路
date: 2017-08-20
categories: 日常技巧
tags: []
---
## 一个移动开发者的Mock数据之路

### 一、始由

    在前端开发中，很大一部分工作都是将后台数据获取到后展示在前端界面上。如果接口是现成的，这个过程还相对容易一些，但是如果接口的开发和前端开发是同时进行的，在仅仅有接口文档并无测试环境的情况下，前端开发者就要痛苦了，所得非所见的盲写方式不但效率低下，也有很大的遗漏风险。如果我们有办法自己根据接口文档模拟这些数据，那开发过程中的体验就会好很多了。幸运的是，通过node.js，express和mock.js，我们可以非常容易的进行数据Mock。

### 二、准备工作

####     1.node.js

    首先你需要安装node.js，node.js是一个JavaScript运行环境，在其官网可以十分方便的进行下载安装：http://nodejs.cn/。

####     2.express

    express是一个基于Node平台的Web开发框架，使用它可以十分方便的搭建本地的web服务，用来部署我们的Mock数据，express可以通过npm来进行安装，官网如下：http://www.expressjs.com.cn/。其中的详细的安装方法。

####     3.mock.js

    mock.js是一个模拟数据结构，生成随机数据的js库。其有一套语法规则用来模拟结构和生成数据。其官网如下，安装过程也十分简单：http://mockjs.com/。

### 三、Mock.js语法规则及范例

    对于Mock数据，最重要的是掌握和灵活运用Mock.js的语法规则，能够熟练的编写出Mock数据结构必备的技能。在Mock.js中，语法规则主要分为两块：数据模板和数据占位符。

#### 1.数据模板

    数据版本主要的作用是用来生成数据结构。数据模板的组成由如下三部分：属性名，生成规则和属性值。在语法上的结构如下：

属性名|生成规则:属性值

    最简单的数据模板是不使用生成规则，直接用字面量来表示，代码如下：

```javascript
{
name:'珲少'
}
```

生成的mock数据如下所示：

![](https://static.oschina.net/uploads/space/2017/0817/163728_QGCR_2340880.png)

对于模拟字符串类型的数据，有两种模板可以定义：

模板1：'属性名|min-max':属性值 

通过重复一个字符串min到max次之间来生成数据。示例：

```javascript
{
'name|1-10':'珲少'
}
```

生成的mock数据如下：

![](https://static.oschina.net/uploads/space/2017/0817/164653_46jt_2340880.png)

模板2：'属性名|count':属性值

通过重复一个字符串count次来生成数据。示例：

```javascript
{
'name|1-10':'珲少',
'moreName|10':'珲少'
}
```

生成数据如下图所示：

![](https://static.oschina.net/uploads/space/2017/0817/164902_phKe_2340880.png)

    对于模拟数值类型的数据，有3种模板可以定义：

模板1：'属性名|+1':属性值

属性值自动自增，示例如下：

```javascript
{
'array|1-10':[
    {
        'name|1-10':'珲少',
        'moreName|10':'珲少',
        'id|+1':0
    }
]
}

```

生成数据如下图所示：

![](https://static.oschina.net/uploads/space/2017/0817/170516_Adxk_2340880.png)

模板2：'属性名|min-max':属性值

生成在min到max之间的整数，代码如下：

```javascript
{
'array|1-5':[
    {
        'name|1-10':'珲少',
        'moreName|10':'珲少',
        'id|+1':0,
        'age|20-25':20
    }
]
}
```

效果如下图所示：

![](https://static.oschina.net/uploads/space/2017/0817/170823_GOaI_2340880.png)

模板3：'属性名|min-max.dmin-dmax':属性值

生成浮点数，整数部分在min到max之间，小数部分保留dmin到dmax位，示例如下：

```javascript
{
'array|1-5':[
    {
        'name|1-10':'珲少',
        'moreName|10':'珲少',
        'id|+1':0,
        'age|20-25':20,
        'weight|60-70.1-4':60
    }
]
}
```

生成数据如下：

![](https://static.oschina.net/uploads/space/2017/0817/171207_uCZ3_2340880.png)

    对于模拟布尔类型的数据，有两种模板可以定义：

模板1：'属性名|1':属性值

随机生成一个布尔值，例如：

```javascript
{
'array|1-5':[
    {
        'name|1-10':'珲少',
        'moreName|10':'珲少',
        'id|+1':0,
        'age|20-25':20,
        'weight|60-70.1-4':60,
        'isWiner|1':true
    }
]
}
```

生成的数据如下：

![](https://static.oschina.net/uploads/space/2017/0817/171501_7dQ3_2340880.png)

模板2：'属性名|min-max':属性值

随机生成一个布尔值，值和属性值相同的概率为min/(min+max)，值与属性值不同的概率为max/(min+max)。

    对于模拟对象类型的数据，有两种模板可以定义：

模板1：'属性名|count':属性值

最终生成的对象的属性为从属性值中随机取count个属性，例如：

```javascript
{
'array|1-5':[
    {
        'name|1-10':'珲少',
        'moreName|10':'珲少',
        'id|+1':0,
        'age|20-25':20,
        'weight|60-70.1-4':60,
        'isWiner|1':true,
        'job|3':{
              num:1234,
              address:'xxxxx',
              phone:12321,
              name:'cjj'
        }
    }
]
}
```

生成的数据如下：

![](https://static.oschina.net/uploads/space/2017/0818/153414_GNVa_2340880.png)

模板2：'属性名|min-max':属性值

从属性值的属性中随机取min到max个作为最终生成的对象属性。

    对于模拟数组类型的数据，有4种模板可以定义：

模板1：'属性名|1':属性值

从属性值数组中随机取1个值作为最终值。

模板2：'属性名|+1':属性值

从属性值数组中依次取1个值作为最终值。

模板3：'属性名|min-max':属性值

通过重复min到max此属性值生成一个数组。

模板4：'属性名|count':属性值

通过重复count此属性值生成数组。

    除了上面列举的创建模板的方式外，还可以使用函数值和正则表达式值作为模板，如果是函数，则生成的值为函数的返回值，如果是正则表达式，则生成的值为可匹配的字符串。

#### 2.数据占位符

    数据占位符实际上就是指定生成的随机数据，它和Mock.Random库中的生成随机数据方法一一对应，其可以模拟邮箱地址，电话号，姓名，行段等各种数据。数据占位符格式如下：

@方法名 或 @方法名(参数)

模拟布尔类型数据：

1.无参：boolean随机返回一个布尔值，示例如下：

```javascript
{
'array|1-5':[
    {
        'name|1-10':'珲少',
        'moreName|10':'珲少',
        'id|+1':0,
        'age|20-25':20,
        'weight|60-70.1-4':60,
        'isWiner|1':'@boolean',
        'job|3':{
              num:1234,
              address:'xxxxx',
              phone:12321,
              name:'cjj'
        }
    }
]
}
```

2.有参：boolean(min,max,current)，指定current出现的概率。

模拟随机自然数：

1.无参：natural随机返回一个大于等于0的整数，示例如下：

```javascript
{
'array|1-5':[
    {
        'name|1-10':'珲少',
        'moreName|10':'珲少',
        'id|+1':0,
        'age|20-25':20,
        'weight|60-70.1-4':60,
        'isWiner|1':'@boolean',
        'job|3':{
              num:'@natural',
              address:'xxxxx',
              phone:12321,
              name:'cjj'
        }
    }
]
}
```

生成数据如下：

![](https://static.oschina.net/uploads/space/2017/0818/162135_nKFc_2340880.png)

2.有参数：natural(nim,max)，随机生成一个在min与max之间的自然数。

模拟随机整数：

1.无参：integer随机生成一个整数。

2.有参：interger(min,max)，随机生成一个在min到max之间的整数。

模拟随机浮点数：

1.无参：float随机生成一个浮点数。

2.有参：float(min,max,dmin,dmax)，随机生成一个整数部分在min到max之间，小数位数为dmin到dmax之间的浮点数。

模拟随机字符：

1.无参：character随机生成一个字符。

2.有参：character(cs),cs为一个字符串，生成的字符从cs字符串中任取一个，如果传入的cs字符串为一下其中之一，则表示从内置字符集中选：

lower：小写字母

upper：大写字符

number：数值字符

symbol：系统字符

模拟随机字符串：

1.无参：string随机生成一个字符串。

2.有参：

格式1：string(length)生成指定长度的字符串。

格式2：string(cs,length)从cs字符池中生成指定长度字符串。

格式3：string(min,max)生成长度在min到max之间的字符串。

格式4：string(cs,min,max)从cs字符集中生成长度在min到max之间的字符串。

模拟整型数组：

有参：

格式1：range(stop)生成一个整型数组，stop为数组中的数值结束边界。

格式2：range(start,stop)start为数值的起始边界，stop为数组中的数值结束边界。

格式3：range(start,stop,step)start为数值的起始边界，stop为数值的结束边界，step为步长。

示例：

```javascript
{
'array|1-5':[
    {
        'name|1-10':'珲少',
        'moreName|10':'珲少',
        'id|+1':0,
        'age|20-25':20,
        'weight|60-70.1-4':60,
        'isWiner|1':'@boolean',
        'job|3':{
              num:'@natural',
              address:'xxxxx',
              phone:12321,
              name:'@string(3)'
        },
        node:'@range(3,5,1)'
    }
]
}
```

模拟日期字符串：

1.无参：date随机生成一个日期字符串。

2.有参：date(format)format用来设置如期字符串的格式，例如：

```javascript
{
    'data|3-5':[
     {
        time:'@date',
        ctime:'@date(yyyy-MM-dd HH-mm-ss A)'
     }
    ]
}
```

生成数据如下：

![](https://static.oschina.net/uploads/space/2017/0818/165854_rNDV_2340880.png)

模拟时间字符串：

1.无参：time直接生成一个时间字符串。

2.有参：time(format)生成格式化的时间字符串。

模拟日期时间字符串：

1.无参：detetime生成默认格式的日期时间字符串。

2.有参：datetime(format)生成指定格式的日期时间字符串。

模拟当前日期字符串：

1.无参：now生成当前日期时间字符串。

2.有参：

格式1：now(unit,format)，unit设置时间单位，format设置格式化方式。时间单位可选：year，month，week，day，hour，minute，second。

格式2：now(format)

格式3：now（unit）

模拟图片素材：

1.无参：iamge随机生成一个尺寸的图片地址，此地址可以直接请求到图像。例如：

```javascript
{
    'data|3-5':[
     {
        time:'@date',
        expriseTime:'@datetime',
        image:'@image' 
    }
    ]
}
```

生成数据如下：

![](https://static.oschina.net/uploads/space/2017/0820/150239_6U6i_2340880.png)

2.有参：

格式1：image(size,background,foreground,format,text)，size指定图片的尺寸，格式为300x250，background指定图片的背景色，foreground指定图片的前景色，format指定图片的格式，可选png、gif、jpg，text参数指定图片上的文字。

格式2：image(size)

格式3：image(size,background)

格式4：image(size,background,foreground)

格式5：image(size,background,foreground,text)

模拟图片二进制素材数据：

1.无参：dataImage生成随机的base64图像编码。

2.有参：

格式1：dataImage(size,text)，size参数这是图片尺寸，text参数设置图片上的文字。

格式2：dataImage(size)

模拟颜色字符串的相关占位符：

1.color：随机生成格式为“#rrggbb”的颜色。

2.hex：随机生成格式为“#rrggbb”的颜色值。

3.rgb：随机生成格式为“rgb(r,g,b)”的颜色值。

4.rgba：随机生成格式为“rgba(r,g,b,a)”的颜色值。

5.hsl：随机生成格式为“hsl(h,s,l)”的颜色值。

模拟英文文本段落：

1.无参：paragraph随机生成一段文本。

2.有参：

格式1：paragraph(min,max)随机生成一段文本，句子个数在min于max之间。

格式2：paragraph(len)随机生成一段文本，len设置句子个数，示例如下：

```javascript
{
    'data|3-5':[
     {
        time:'@date',
        expriseTime:'@datetime',
        image:'@image("300x250","#222312","#f1f2f3","jpg","image")' ,
        content:'@paragraph(1,3)'
    }
    ]
}
```

生成的随机数据如下：

![](https://static.oschina.net/uploads/space/2017/0820/152353_6EcF_2340880.png)

模拟中文文本段落：

1.无参：cparagraph随机生成中文段落

2.有参：

格式1：cparagraph(min,max)

格式2：cparagraph(len)

模拟英文句子：

1.无参：sentence随机生成一句文本，首字母会大写。

2.有参：

格式1：sentence(min,max)，随机生成一句文本，文本中单词个数为min到max之间。

格式2：sentence(len)，随机生成一句文本，文本中单词个数为len。

模拟中文句子：

1.无参：csentence随机生成一句中文文本。

2.有参：

格式1：csentence(min,max)

格式2：csentence(len)

模拟英文单词：

1.无参：word随机生成一个单词。

2.有参：

格式1：word(min,max)，生成单词中字符个数为min到max之间。

格式2：word(len)，生成单词中字符个数为len。

模拟中文词：

1.无参：cword随机生成一个汉字。

2.有参：

格式1：cword(pool)，pool为汉字字符串，从pool字符池中选取一个汉字。

格式2：cword(length)，随机生成一个词，汉字个数为length。

格式3：cword(min,max)，随机生成一个词，汉字个数为min到max之间。

格式4：cword(pool,length)

格式5：cword(pool,min,max)

模拟标题：

1.无参：title随机生成标题。

2.有参：

格式1：title(len)生成单词个数为len的标题。

格式2：title(min,max)生成单词个数为min到max之间的标题。

模拟中文标题：

1.无参：ctitle

2.有参：

格式1：ctitle(len)

格式2：ctitle(min,max)

模拟姓名相关的占位符：

1.first随机生成常见的英文名。

2.last随机生成常见的英文姓。

3.name(middle)，随机生成一个英文姓名，middle为布尔参数，设置是否生成中间名。

4.cfirst随机生成一个中文姓。

5.clast随机生成一个中文名。

6.cname随机生成一个中文姓名。

模拟网址相关占位符：

1.url(protocol,host)随机生成一个url，protocol指定协议，host指定主机，也可以无参。

2.protocol随机生成一个url协议，例如http。

3.domain随机生成一个域名。

4.tld随机生成一个顶级域名。

5.email(domain)随机生成一个email地址，domain指定域名，也可以无参。

6.ip随机生成一个ip地址。

模拟地址相关占位符：

1.region随机生成一个中国区域，例如华北。

2.province随机生成一个中国省份。

3.city(pro)随机生成一个中国城市，pro为布尔值，指定是否生成其所在的省份，也可以无参。

4.county(all)随机生成一个中国县，all为布尔值，指定是否生成其所在的省市。也可以无参。

5.zip随机生成一个邮编。

模拟id相关占位符：

1.guid随机生成一个GUID。

2.id随机生成一个18位身份证号。

### 四、一个用express搭建Mock服务的示例

```javascript
var express = require('express');
var app = express();
var Mock = require('mockjs');

app.get('/mock', function(req, res){
    var data = Mock.mock({
        "name":'@cname', 
        "content":'@string(0, 5)',
    });
    response = {
        "data":data
    }
    res.end(JSON.stringify(response));
});


var server = app.listen(8082, function(){
    var host = server.address().address
    var port = server.address().port
})
```

上面代码开启了一个get请求，如果正确安装了node，express，mock.js，使用node运行此文件后直接在浏览器通过127.0.0.1:8082/mock地址进行访问即可看到生成的mock数据。

    Mock数据的初衷是在前端开发中进行接口的模拟使用，在接口结构和访问url都已经确定，只是没开发完成是，可以使用Charles结合Mock数据来仿真接口返回。Charles工具可以将某个请求映射到另外一个地址上，在Charles抓到的请求上邮件，弹出的菜单中选择Map Remote选项。

![](https://static.oschina.net/uploads/space/2017/0820/164126_1mXd_2340880.png)

在弹出的窗口中将映射到的主机设置为127.0.0.1，端口设置为8082，地址设置为mock即可访问上面文件生成的模拟数据。

### 五、一个方便易用的Java模拟数据客户端

    有时候我们需要mock的接口有很多，如果能够方便的对这些mock文件进行管理不用每次都通过终端来操作就更方便了。这里有我写的一个JAR小工具，可以在Mac或Windows上扩平台进行使用。下载地址如下：

[http://zyhshao.github.io/EasyMock/welcome.html](http://zyhshao.github.io/EasyMock/welcome.html)。

这个工具就是一个简单的JAR包，在其中封装了操作终端的命名，只需要在左右列表中创建相应的请求路径，在右侧直接编写Mock.js模拟数据对象后，开启服务即可，开启服务后会将左右列表中所有的接口都开启。如下图：

![](https://static.oschina.net/uploads/space/2017/0820/165259_s8O7_2340880.png)

还需要注意，这个工具不十分完善，如果有产生错误会被捕获但并没有任何提示，如果你没正确安装node或者express或者mock.js，再或者你的mock.js代码有问题，服务都不能正确启动。

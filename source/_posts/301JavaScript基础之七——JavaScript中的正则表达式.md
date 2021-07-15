---
title: JavaScript基础之七——JavaScript中的正则表达式
date: 2017-01-05
categories: 前后端
tags: []
---
## JavaScript基础之七——JavaScript中的正则表达式

    正则表达式在前端开发中应用十分广泛，从表单验证到内容替换，正则多发挥着十分重要的作用。JavaScript中提供了专门的正则对象。

    在JavaScript中，有两种方式创建正则表达式，分别可以通过直接量语法来创建和RegExp对象来创建，示例如下：

```javascript
var string = "Hello World123454321,{name:jaki,age:25}Hello,HELLO";
//使用直接量语法创建正则
console.log(string.match(/ello/g));//[ 'ello', 'ello' ]
//使用对象构造正则
var reg1 = new RegExp("ello","g");
console.log(string.match(reg1));//[ 'ello', 'ello' ]
```

使用直接量语法创建正则的模板如下:

/param/attri

其中param是正则表达式，attri为修饰参数，可以选择的有，i、g、m3个。i代表忽略大小写，g代表全局检索，m代表多行检索。

同样，使用RegExp对象的构造方法来构造正则对象也需要两个参数，第1个参数为正则表达式，第2个参数为修饰参数。

    正则表达式可以使用括号来进行范围查找，示例如下：

```javascript
//i 表示忽略大小写 g表示全局搜索 m表示多行搜索
var reg2 = new RegExp("ello","igm");
console.log(string.match(reg2));//[ 'ello', 'ello', 'ELLO' ]
//[]内表示匹配范围内的某个字符
var reg3 = new RegExp("[abcd]","g");
console.log(string.match(reg3));
//前面加上^表示取反 匹配除了[]内的所有其他字符
var reg4 = new RegExp("[^abcd]","g");
console.log(string.match(reg4));
//进行范围匹配 如下只匹配数字
var reg5 = new RegExp("[0-9]","g");
console.log(string.match(reg5));
//进行范围匹配 如下只匹配英文字符
var reg6 = new RegExp("[a-z]","ig");
console.log(string.match(reg6));
//进行指定串的匹配
var reg7 = new RegExp("(123|hell)","ig");//[ 'Hell', '123', 'Hell', 'HELL' ]
console.log(string.match(reg7));
```

    在构造正则表达式时，也可以灵活的使用许多元字符，示例如下：

```javascript
//元字符
//.元字符会比配任何字符 除了换行和行结束符
var reg8 = new RegExp("e.l","g");
console.log(string.match(reg8));
//x+用于匹配一个或多个x字符
var reg9 = new RegExp('l+',"g");
console.log(string.match(reg9));
//x*匹配0个或多个x字符
var reg10 = new RegExp('He*',"g");
console.log(string.match(reg10));
//x?匹配0个或1个x字符
var reg11 = new RegExp('He?',"g");
console.log(string.match(reg11));
//x{n}匹配有n个x字符
var reg12 = new RegExp('l{2}',"g");
console.log(string.match(reg12));
//x{n,m}匹配有n-m间个x字符
var reg13 = new RegExp('l{1,5}',"g");
console.log(string.match(reg13));
//x{n,}匹配至少有n个x字符
var reg14 = new RegExp('l{1,}',"g");
console.log(string.match(reg14));
//x$匹配以x结尾
var reg15 = new RegExp('O$',"g");
console.log(string.match(reg15));
//^x匹配以x开头
var reg15 = new RegExp('^H',"g");
console.log(string.match(reg15));
```

    关于RegExp对象，其中也封装了一些属性和方法，示例如下：

```javascript
//获取是否有相应标志位
console.log(reg15.global);
console.log(reg15.ignoreCase);
console.log(reg15.multiline);
//检索字符串中正则匹配
console.log(reg15.exec(string));
//检测字符串是否匹配 会返回true或者false
console.log(reg15.test(string));
```

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

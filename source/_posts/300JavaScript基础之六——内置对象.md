---
title: JavaScript基础之六——内置对象
date: 2017-01-04
categories: 前后端
tags: []
---
## JavaScript基础之六——内置对象

### 一、构造对象

    JavaScript中的一些数据都是对象，对象实际上是属性与方法的包装。并不像其他类似Swift/OC/Java类的面向对象语言，在目前JavaScript的实现中并没有类的概念，开发者有如下两种方式来进行对象的构造：

```javascript
//创建对象的方式有两种 可以直接创建对象的实例
var obj = new Object();
obj.name = "jaki";
obj.age = 25;
console.log(obj);
//也可以使用大括号才创建对象
var obj2 = {
    name:"jaki",
    age:25,
    show:function(){
        console.log(this.name+"--"+this.age);
    }
};
obj2.show();
```

回顾Swift语言的对象创建方法，是通过类调用构造方法，因此，在JavaScript中，也可以通过函数来模拟类的功能，此类的函数可以称为构造函数，示例如下：

```javascript
//模拟类 构造方法
function Person(name,age){
    this.name = name;
    this.age = age;
    this.show = function(){
        console.log(this.name+"--"+this.age);
    };
}
var p = new Person("jaki",25);
p.show();
```

JavaScript中的对象呗构造出来后，依然可以为其动态添加属性或方法，示例如下：

```javascript
//模拟类 构造方法
function Person(name,age){
    this.name = name;
    this.age = age;
    this.show = function(){
        console.log(this.name+"--"+this.age);
    };
}
var p = new Person("jaki",25);
p.show();
//已经构造的对象 也可以增加属性
p.phone = "1111"
console.log(p.phone);
```

### 二、数值对象Number

    在JavaScript中，数值是一种基本数据类型，但是Number是数值对象，可以理解为对数值的包装。并且JavaScript中的数值只有一种类型，可以创建整数，也可以创建小数，如下：

```javascript
//数字对象
//JS中只有一种数字对象Number 
//可以描述整数  也可以描述小数
var c = 10;
var c1 = 3.14;
```

也支持使用科学计数法来对数值进行描述，示例如下：

```javascript
//也可以使用科学计数法来计数
var c2 = 1.2e5;
var c3 = 123e-5;
console.log(c2,c3);
```

在JavaScript中，使用前缀0来描述八进制数值，使用前缀0x来描述十六进行的数值，示例如下：

```javascript
//使用0为前缀 约定为8进制
var c4 = 017;
console.log(c4);  //十进制15
//使用0x前缀约定为十六进制
var c5 = 0x11;
console.log(c5); //十进制17
```

需要注意：和其他语言不同，JavaScript中不能随意的数值前面加0，否则会被认为是八进制计数。

    关于Number，如果使用new来进行构造，会返回一直数值对象，其中可以穿入一个参数作为数值对象的原始值，如果将Number()作为函数来使用，则会直接返回一个具体的数值，示例如下：

```javascript
//Number可以作为构造方法来使用 传入的参数为要构造的Number值
var objc = new Number(15);
console.log(objc);
//Number也可以作为函数来使用 这时他将返回数值或者NaN
var objc2 = Number(2);
console.log(objc2); //2
var objc3 = Number("c");
console.log(objc3);//NaN
```

Number中内置了一些常用的属性与方法，示例如下：

```javascript
//常用内置属性
//返回可表示的最大值
console.log(Number.MAX_VALUE);
//返回可表示的最小值
console.log(Number.MIN_VALUE);
//返回非数字值
console.log(Number.NaN);
//表示负无穷大 发生溢出时 会返回
console.log(Number.NEGATIVE_INFINITY);
//表示正无穷大 发生溢出时 会返回
console.log(Number.POSITIVE_INFINITY);
```

对象实例的常用方法：

```javascript
//常用内置方法
var c6 = 100;
//将数据转换成字符串
console.log(c6.toString());
//toString()方法中可以传入一个参数 默认为10进制 可选2-36
console.log(c6.toString(36));
//toLocaleString()方法返回本地环境格式的字符串  一般10进制
console.log(c6.toLocaleString());

var c7 = 3.1415926;
//toFixed()方法用于将数字转换成字符串 可以指定小数位数 会四舍五入
console.log(c7.toFixed(3));
//将数值转换成指数计数法
console.log(c7.toExponential(1));
//toPrecision()方法可以将数值转换为指定位数
console.log(c7.toPrecision(4));
//返回原始数值
console.log(c7.valueOf());
```

需要注意，对于进制转换，toString()方法支持2-36进制，这很好理解，数字0-9加上26个英文字母，最多可以表达36个数字。

### 三、字符串对象String

    JavaScript语言中的字符串对象封装了大量的操作方法，需要注意，JavaScript中的String对象是不可变的，所有对字符串的操作都是返回一个新的字符串。String对象中最常用的一个属性为length属性，这个属性可以获取字符串的长度，示例如下：

```javascript
//字符串对象
var str1 = new String("HelloWorld");
var str2 = String("HELLO");
//length属性可以获取字符串的长度
console.log(str1.length);  //10
```

String对象中封装的一些常用方法示例如下：

```javascript
//静态方法
//通过Unicode值创建字符
var str3 = String.fromCharCode(100,102,103,104);//dfgh
console.log(str3);

//创建html锚点
console.log(str1.anchor("label")); //<a name="label">HelloWorld</a>
//嵌入big标签中
console.log(str1.big());//<big>HelloWorld</big>
//嵌入small标签
console.log(str1.small());//<small>HelloWorld</small>
//嵌入blick标签中
console.log(str1.blink());//<blink>HelloWorld</blink>
//嵌入b标签中
console.log(str1.bold());//<b>HelloWorld</b>
//获取指定位置的字符 从0开始
console.log(str1.charAt(1));//e
//获取指定位置字符的unicode码
console.log(str1.charCodeAt(1));//101
//字符串拼接
console.log(str1.concat("1","2","3"));//HelloWorld123
//嵌入打印模式
console.log(str1.fixed());//<tt>HelloWorld</tt>
//指定HTML字体颜色
console.log(str1.fontcolor('red'));//<font color="red">HelloWorld</font>
//指定HTML字体大小
console.log(str1.fontsize(24));//<font size="24">HelloWorld</font>
//indexOf()方法用于检查某个子串在父串中第一次出现的位置，其中第1个参数为要检索的子串，第2个参数为从某个位置开始检索
console.log(str1.indexOf('l',3));//3  注意，如果没有检索到 会返回-1
//嵌入i标签中
console.log(str1.italics());//<i>HelloWorld</i>
//从后往前检索 同indexOf()方法
console.log(str1.lastIndexOf("l",10));//8
//嵌入a标签中
console.log(str1.link("www.baidu.com"));//<a href="www.baidu.com">HelloWorld</a>
//进行字符串比较 如果str1<参数 则返回-1 如果大于 则返回1 相等 则返回0
console.log(str1.localeCompare("Z"));//-1
//match()方法用于字符串检索 其将返回一个检索结果对象
//match()方法的参数可以为
//字符串:要检索的字符串
//正则表达式:要检索的正则表达式
console.log(str1.match("ell"));//[ 'ell', index: 1, input: 'HelloWorld' ]
console.log(str1.match(/ell/));
//进行字符串替换 第1个参数为要被替换的字符或者正则 第2个参数为替换成的字符串
console.log(str1.replace("Hello","hahaha"));//hahahaWorld
//进行子串匹配 可以是字符串参数也可以是正则 这个方法不能指定检索起点
console.log(str1.search("World"));//5
//进行字符串截取
console.log(str1.slice(0,5));//Hello
//进行字符串分隔操作
//第一个参数：分隔标准 可以会字符串或者正则
//第二个参数：返回数组中元素的最大个数
console.log(str1.split('o',10));//[ 'Hell', 'W', 'rld' ]
//嵌入strike标签中
console.log(str1.strike());//<strike>HelloWorld</strike>
//嵌入sub标签
console.log(str1.sub());//<sub>HelloWorld</sub>
//进行字符串截取 第1个参数为起始位置 第2个参数为需要截取的长度
console.log(str1.substr(0,5));//Hello
//进行字符串截取 获取两个索引之间的字符
console.log(str1.substring(5,10));//World
//嵌入sup标签中
console.log(str1.sup());//<sup>HelloWorld</sup>
//全部转换为小写字母
console.log(str1.toLocaleLowerCase());
console.log(str1.toLowerCase());
//全部转换为大写字母
console.log(str1.toLocaleUpperCase());
console.log(str1.toUpperCase());
//返回字符串对象的原始值
console.log(str1.valueOf());
```

### 四、日期对象Date

    JavaScript中提供丰富的日期处理方法，示例如下：

```javascript
//日期对象date
//创建当前时间对象
var date1 = new Date();
console.log(date1);//2017-01-04T02:00:05.617Z
//返回当前的日期
var date2 = Date();
console.log(date2);//Wed Jan 04 2017 10:04:56 GMT+0800 (CST)
//获取时间对象是一个月中的第几天 1-31之间 必须是Date对象调用
console.log(date1.getDate()); //4
//获取时间对象是一周中的第几天 周日为0
console.log(date1.getDay());  //3
//返回月份 注意 这个会返回0-11之间
console.log(date1.getMonth()); //0
//获取完整的年
console.log(date1.getFullYear()); //2017
//获取小时0-23
console.log(date1.getHours());
//获取分钟0-59
console.log(date1.getMinutes());
//获取秒0-59
console.log(date1.getSeconds());
//获取毫秒0-999
console.log(date1.getMilliseconds());
//获取时间戳 从1970年1月1日 起
console.log(date1.getTime());
//返回本地时间与格林威治时间的分钟差
console.log(date1.getTimezoneOffset());//-480
//返回世界时间中 一个月中的第几天 1-31
console.log(date1.getUTCDate());
//返回世界时间中 一周中的第几天  周日为0
console.log(date1.getUTCDay());
//返回世界时间中的月份 0-11
console.log(date1.getUTCMonth());
//返回世界时间中的年
console.log(date1.getUTCFullYear());
//返回世界时间中小时
console.log(date1.getUTCHours());
//返回世界时间中的分钟
console.log(date1.getUTCMinutes());
//返回世界时间中的秒
console.log(date1.getUTCSeconds());
//返回世界时间中的毫秒
console.log(date1.getUTCMilliseconds());
//静态方法 返回从1970.1.1到指定日期间的毫秒
console.log(Date.parse(date1));
//设置日期 1个月中的某一天 1-31
date1.setDate(1);
//设置月份
date1.setMonth(2);
//设置年份
date1.setFullYear(2011);
//设置时
date1.setHours(10);
//设置分
date1.setMinutes(20);
//设置秒
date1.setSeconds(23);
//设置毫秒
date1.setMilliseconds(100);
console.log(date1);
//通过时间戳设置date对象
date1.setTime(1483599203000);
console.log(date1);
//下面这些方法设置时间时间
date1.setUTCDate(3);
date1.setUTCMonth(10);
date1.setUTCFullYear(2020);
date1.setUTCHours(22);
date1.setUTCMinutes(22);
date1.setUTCSeconds(34);
date1.setUTCMilliseconds(112);
console.log(date1);
//转换为本地时间字符串
console.log(date1.toString());//Wed Nov 04 2020 06:22:34 GMT+0800 (CST)
//转换时间部分
console.log(date1.toTimeString());//06:22:34 GMT+0800 (CST)
//转换日期部分
console.log(date1.toDateString());//Wed Nov 04 2020
//转换为世界时间字符串
console.log(date1.toUTCString());//Tue, 03 Nov 2020 22:22:34 GMT
//转换成本地格式的字符串
console.log(date1.toLocaleString());//11/4/2020, 6:22:34 AM
//只转换本地格式的时间
console.log(date1.toLocaleTimeString());//6:22:34 AM
//只转换日期
console.log(date1.toLocaleDateString());//11/4/2020
//返回从1970.1.1到指定时间的时间戳 
//参数含义为 年 月 日 时 分 秒 毫秒
console.log(Date.UTC(2012,1,1,1,1,1,1));
```

### 五、数组对象Array

    数组对象用于存放一组数据，JavaScript语言并不像Swift语言那样强调类型，因此数组中存放的元素类型十分自由，JavaScript中数组的相关方法示例如下：

```javascript
//有三种方式进行数组的构造
//构造空数组
var array1 = new Array();
//构造指定个数元素的数组
var array2 = new Array(5);
//构造初始数组
var array3 = new Array(1,2,3,4)

//length属性可以获取数组元素个数
console.log(array3.length);//4
//进行数组连接
console.log(array3.concat([5,6],['q','w']));
//将数组中所有元素以指定分隔符进行拼接为字符串
console.log(array3.join("-"));
//删除数组中的最后一个元素
array3.pop();
console.log(array3);
//向数组中添加一个元素
array3.push(5);
console.log(array3);
//进行数组颠倒
array3.reverse();
console.log(array3);
//删除数组中的第一个参数
array3.shift();
console.log(array3);
//向数组开头插入一些元素
array3.unshift(1,1,1);
console.log(array3);
//对数组元素进行排序 
array3.sort(function(a,b){
    if (a>b) {
        return true;
    }else{
        return false;
    }
});
console.log(array3);
//删除数组中的元素 并插入其他元素
//第1个参数为参数元素的位置
//第2个参数为删除元素的个数
//之后可以有任意个参数，作为插入元素
array3.splice(0,2,'c',5);
console.log(array3);
//将数组转换为字符串 使用逗号拼接
console.log(array3.toString());
console.log(array3.toLocaleString());
```

    需要注意，数组的排序方法sort()中需要传入一个排序函数，这个函数中会传入两个参数，分别描述数组中相邻的两个元素，如果需要交换位置，返回true即可，否则返回false即可。另外，数组的toString()方法与join()方法作用相似，不同的是join()方法更加自由，开发者可以通过参数决定进行拼接的方式，如果不传参数，则默认也会以逗号进行分割拼接。

### 六、关于Boolean对象

    Boolean对象用来描述逻辑值，JavaScript中的Boolean对象可以理解为对布尔值的一种包装，当使用构造函数来进行Boolean对象的创建时，如果不传参数，默认会构造false值的对象包装，如下：

```javascript
var b1 = new Boolean();
console.log(b1);//[Boolean: false]
```

如果将Boolean()当做函数来使用，将会返回一个基本布尔值，如下：

```javascript
var b2 = Boolean();
console.log(b2);//false
```

在创建布尔值时，下面这些传参都将创建包装false的布尔对象：

```javascript
var myBoolean=new Boolean();
var myBoolean=new Boolean(0);
var myBoolean=new Boolean(null);
var myBoolean=new Boolean("");
var myBoolean=new Boolean(false);
var myBoolean=new Boolean(NaN);
var myBoolean=new Boolean(undefined);
```

除了上面所列举的参数情况外，其余的都将构造true包装对象。

### 七、JavaScript中的数学对象及方法

    JavaScript中还内置了一个Math数学对象，这个对象中封装了许多数学中常用的常数和算术方法，示例如下：

```javascript
//Math对象
//自然对数e
console.log(Math.E);//2.718281828459045
//2的自然对数
console.log(Math.LN2);//0.6931471805599453
//10的自然对数
console.log(Math.LN10);//2.302585092994046
//以2为底e的对数
console.log(Math.LOG2E);//1.4426950408889634
//以10为底e的对数
console.log(Math.LOG10E);//0.4342944819032518
//圆周率
console.log(Math.PI);//3.141592653589793
//根号2的倒数
console.log(Math.SQRT1_2);//0.7071067811865476
//根号2
console.log(Math.SQRT2);//1.4142135623730951

//求绝对值函数
console.log(Math.abs(-5));//5
//求反余弦函数
console.log(Math.acos(0.5));
//求反正弦函数
console.log(Math.asin(0.5));
//求反正切函数
console.log(Math.atan(1));
//求到(x,y)点的角度
console.log(Math.atan2(1,0));
//进行向上舍入
console.log(Math.ceil(1.3));//2
//进行向下舍入
console.log(Math.floor(1.3));//1
//求自然对数
console.log(Math.log(10));//2.302585092994046
//返回最大值
console.log(Math.max(1,2));//2
//返回最小值
console.log(Math.min(1,2));//1
//求幂函数
console.log(Math.pow(10,2));//100
//取0-1之间的随机数
console.log(Math.random());
//进行四舍五入
console.log(Math.round(1.3));//1
//求正弦值
console.log(Math.sin(2));
//求余弦值
console.log(Math.cos(2));
//求正切值
console.log(Math.tan(1));
```

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

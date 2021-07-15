---
title: JavaScript基础之八——全局函数的应用
date: 2017-01-06
categories: 前后端
tags: []
---
## JavaScript基础之八——全局函数的应用

    JavaScript中提供了一些常用的全局函数，开发者可以直接对其进行调用，示例如下：

```javascript
var url = "jaki.io/v3/珲少";
//对字符串进行url编码 这个方法不会对ascll码进行编码
var enUrl = encodeURI(url);
console.log(enUrl);//jaki.io/v3/%E7%8F%B2%E5%B0%91
//对字符串进行url解码
console.log(decodeURI(enUrl));//jaki.io/v3/珲少
//进行uri全编码
var enCompUrl = encodeURIComponent(url);
console.log(enCompUrl);//jaki.io%2Fv3%2F%E7%8F%B2%E5%B0%91
//记性URI全解码
console.log(decodeURIComponent(enCompUrl));//jaki.io/v3/珲少
//eval()方法可以将某个字符串解释成JS代码进行执行
eval("console.log('eval')");
//检查某个值是否为有限数字
console.log(isFinite(Infinity));
//检查某个值是否为非数字
console.log(isNaN("s"));
//把对象的值转换为数字
console.log(Number("222"));
//将一个字符串解析成浮点数
console.log(parseFloat("3.14"));
//将一个字符串解析成整数
console.log(parseInt("123"));
//把对象的值转换成字符串
console.log(new Date());
```

需要注意，encodeURI()与encodeURIComponent()方法都是用来对URI进行编码，不同的是，encodeURI()方法不会对ascll字符进行编码，在进行有中文字符的url编码时，需要使用这个方法。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

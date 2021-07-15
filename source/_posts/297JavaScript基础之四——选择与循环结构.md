---
title: JavaScript基础之四——选择与循环结构
date: 2016-12-31
categories: 前后端
tags: []
---
## JavaScript基础之四——选择与循环结构

    选择结构与循环结构是编程中处理逻辑的核心结构，JavaScript中支持if-else和switch-case选择结构，支持for，for-in,do-while,while循环结构。并且可以使用break与continue语句进行循环的跳出，简单的条件选择if语句示例如下：

```javascript
//if条件语句
if (true) {
    console.log("条件语句");
};
if (false) {

} else {
    console.log("if-else语句");
};
var a = 10;
if (a < 10) {
    console.log("a<10");
} else if (a == 10) {
    console.log("a=10");
} else {
    console.log("a>10");
};
```

switch-case选择结构用于多分支条件的选择，示例如下：

```javascript
//选择语句
var b = "hi";
switch (b) {
    case "hello":
        {
            console.log("Hello world");
        }
        break;
    case "hi":
        {
            console.log("Hi world");
        }
        break;
    default:
        {
            console.log("都没匹配上");
        }
}
```

需要注意，每个case结构后面原则上都需要使用break进行中断匹配，如果不添加此break，则匹配到一个case语句后switch结构并不会结束，会继续尝试匹配后面的case条件。

    for循环结构用于处理大量重复的逻辑，示例如下：

```javascript
for (var i = 0; i < 10; i++) {
    console.log("循环" + i);
}
for (var i = 0; i < 10; i++) {
    console.log("循环" + i);
    if (i == 2) {
        //使用break可以提前中断循环
        break;
    };
}
```

JavaScript还有一种更高效的循环模式，for-in结构，这种结构专门用来遍历对象，其可以将对象的属性遍历出来，示例如下：

```javascript
var obj1 = {
    name: "jaki",
    age: 25
};
var obj2 = [1, 2, 3, 4, 5, 6, 7, 8];
for (var x in obj1) {
    //跳过本次循环 并不是跳出循环
    if (x == "name") continue;
    console.log(x + ":" + obj1[x]);

}
for (var x in obj2) {
    console.log(x + ":" + obj2[x]);

}
```

需要注意，对于数组，其遍历出来的是数组的下标，并不是其中的值，这和C/OC,Swift等语言有所差异，也证明了数组在JavaScript中其实就是一种特殊的对象。

    while循环和do-while循环的差异在于whlie结构是先进行循环条件的判断，再进入循环体，而do-while结构则是先进入循环体，在进行循环条件的判断，示例如下：

```javascript
var c = 1;
while (c < 10) {
    console.log(c);
    c++;
}
do {
    console.log(c);
    c--;
} while (c > 1);
```

    前面提到过break和continue语句，break语句用于中断switch-case匹配或者跳出最近的循环，跳出循环的意思是指执行到break后，无论后面循环次数还有多少次，直接跳出，执行循环结构之后的代码。continue语句的作用则是跳出最近的本次循环，接着进行循环条件的判断，如果满足会继续进行循环，并且如果有多层循环嵌套，break和continue也可以通过label标签指定具体跳出那层循环，示例如下：

```javascript
LAB: for (var i = 0; i < 5; i++) {
    for (var j = 0; j < 5; j++) {
        if (j == 2) {
            break LAB
        };
        console.log(i + '==' + j);
    };
};
```

上面的代码，如果不使用LAB标签，则外层循环不会被中断。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

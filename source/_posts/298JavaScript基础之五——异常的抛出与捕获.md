---
title: JavaScript基础之五——异常的抛出与捕获
date: 2017-01-02
categories: 前后端
tags: []
---
## JavaScript基础之五——异常的抛出与捕获

    任何程序在运行过程中都会产生开发者意想不到的异常，因此对异常的处理逻辑是一种编程必备的能力。在JavaScript语言中，使用try-catch块来完成对异常的捕获与处理。

    正常情况下，当JavaScript程序运行到有异常的地方时，程序会自动中断，例如开发者使用了一种未定义的变量或函数、由于手误造成的错字、由于用户输入非法造成的意想不到的错误等。但是开发者可以使用try-catch结构对可能抛出异常的代码进行异常捕获，如果捕获到异常，开发者可以选择处理或不处理，如果异常被捕获，程序就不会中断，示例代码如下：

```javascript
//异常的抛出与捕获
try{
    consele.log("异常");
}catch(error){
    console.log(error);
}
```

    除了某些系统抛出的异常外，开发者也可以定义和抛出自己的异常，使用throw关键字可以抛出异常，示例如下：

```javascript
//使用throw关键字用于异常的抛出
var func = function(){
    throw "My Error"
}
try{
    func();
}catch(error){
    console.log(error);
}
```

需要注意，抛出的异常可以是自定的异常对象，可以是字符串，可以使任意JavaScript对象。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

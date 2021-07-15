---
title: 深入理解JavaScript函数
date: 2017-02-15
categories: 前后端
tags: []
---
## 深入理解JavaScript函数

### 一、引言

    从功能上理解，函数是一组可以随时运行的语句，是一段代码块，也是所谓的子程序。在JavaScript中，函数实质上也是一种对象，是Function对象。函数通常会有参数与返回值(不是必须)，在JavaScript中，函数的应用十分灵活，也有多种定义函数的方法。

### 二、几种定义函数的方式

    在JavaScript中，可以通过函数语句来声明和定义函数、可以通过函数表达式来将创建函数，也可以使用Function构造方法来创建函数对象。

#### 1、使用函数语句来定义函数

    JavaScript中有一种特殊的语法来直接定义函数，示例如下：

```javascript
//使用函数语句定义函数
function outputName(){
    console.log("Jaki");
}
```

函数语句有这样的结构：function name(param...){}。其中function为函数语句的关键字，name为自定义的函数名称，小括号中定义函数的形参，大括号中可以进行代码的编写。

如果仅仅对上面的代码进行运行，你会发现程序并没有执行任何行为，函数必须在调用时才会被执行，调用函数示例如下：

```javascript
//使用函数语句定义函数
function outputName(){
    console.log("Jaki");
}
outputName();//将打印Jaki
```

上面的定义的函数是最简单的函数形式，函数也可以通过传入参数的差异来做不同的功能，例如修改上面的函数，使其传入姓名并且进行打印输出：

```javascript
//使用函数语句定义函数
function outputName(name){
    console.log(name);
}
outputName("张三");//将打印张三
```

函数也可以提供一个返回值，例如一个简单的加法运算函数，如下：

```javascript
function add(a,b){
    return a+b;
}
var res = add(3,4);
console.log(res);//7
```

在JavaScript中，函数的返回值类型不需要特殊指定，直接使用return语句返回即可。需要注意，实际上任何函数都是有返回值的，如果函数体中没有使用return语句显式的进行返回，则默认会返回undefined。

    JavaScript中函数的定义与使用不一定非要按照顺序进行，实际上也可以将函数的定义写在使用之后(解释器会在与处理阶段解释定义的函数)，如下：

```javascript
var res = add(3,4);
function add(a,b){
    return a+b;
}
console.log(res);//7
```

### 2.使用函数表达式来定义函数

    使用函数表达式来定义函数与函数语句的语法十分相似，示例代码如下：

```objectivec
//使用函数表达式来定义函数
var addFunc = function(a,b){
    return a+b;    
};
var res = addFunc(3,3);
console.log(res);//6
```

这种方式定义的函数实际上是创建了一个函数对象，之后将这个对象的引用赋值给了一个变量，通过变量开发者可以访问和调用函数。需要注意，函数表达式与函数语句的最大区别在于其可以省略函数名，即可以定义匿名函数，但是这种方式定义的函数在函数定义之前是不能够被调用的，这也很好理解，JavaScript解释器在预处理期间只是解析除了addFunc变量，但是并没有对其表达式进行运算，这时函数并没有被定义，例如下面的代码将报错：

```javascript
var res = addFunc(3,3);
//使用函数表达式来定义函数
var addFunc = function(a,b){
    return a+b;    
};
```

其实函数语句与函数表达式定义的函数还有一点重大区别，函数语句定义后函数名实际上是无法修改的，而使用函数表达式定义的函数是将引用赋值给了变量，变量可以重新赋值，也可以将两个变量赋值为同一个函数对象的引用，示例如下：

```javascript
//使用函数表达式来定义函数
var addFunc = function(a,b){
    return a+b;    
};
//将addFunc的值赋值给另一个变量
var addFunc2 = addFunc;
//将addFunc变量修改为整型值
addFunc = 3;
var res = addFunc2(3,3);
console.log(res);//6
```

如果需要在函数内部调用函数本身，即需要定义递归函数时，函数表达式也可以进行函数命名，示例如下：

```javascript
//定义一个递归阶乘函数
var mathFunc = function mathF(a){
    var res = a;
    a--;
    if (a>0) {
        res *= mathFunc(a);
    }
    return res;
};
var mathRes = mathFunc(5);
console.log(mathRes);//120
```

需要注意，函数表达式定义的函数名只能作用于函数内部。

### 3.使用Function构造函数

    前面有提到，函数在JavaScript中实际上也是一种对象，因此可以使用new关键字来进行函数对象的构造，示例如下：

```javascript
//Function构造器
var myFunc = new Function("name","console.log(name)");
myFunc("Jaki");//Jaki
```

Function构造其的格式如下：Function(param,param...,funcbody)。其中参数个数不固定，最后一个参数为创建的函数的函数体字符串，前面的参数全部都将作为函数的形参传入函数体内。

    实际上，无论通过哪种方式创建的函数，其实质上都是Function对象，Function对象中也有一些内置的属性，其中最长用的为arguments属性。arguments属性是实际传入函数的参数数组，其length属性为函数所传入的参数个数，也就是说，开发者在定义函数的时候实际上可以不定义形参，在调用函数的时候直接进行实参的传入，函数内部还是可以获取到参数，JavaScript函数的对象特性使得其十分灵活，形参并不能决定函数的参数个数与类型，实参、函数体和返回值这些都可以在运行时决定！示例如下：

```javascript
function testFunc(){
    console.log(arguments.length);
    console.log(arguments);
}
/*
3
{ '0': '1', '1': 2, '2': 3 }
*/
testFunc('1',2,3);
```

函数是功能完整的对象，实际上函数对象本身也有一个length属性，这个属性将获取到函数预期形参的个数，即可以方便的获取到函数预期需要传入的参数的个数，例如：

```javascript
function testFunc(){
    console.log(arguments.length);
    console.log(arguments);
}
function textFunc2(name){
    
}
console.log(testFunc.length); //0
console.log(textFunc2.length); //1
```

### 三、理解函数语句与函数表达式

    函数语句是定义函数的一种语法，而函数表达式实际上就是返回一个函数对象，因此函数表达式可以直接进行调用，请看如下代码：

```javascript
var addFuncS = function(a,b){
    console.log(a+b);    
}(1,2);//3
```

上面的代码中的addFuncS已经不再是一个函数对象的引用，而是数值3。在函数表达式后面可以直接跟小括号传参进行函数的调用，但是如下的写法将会报错：

```javascript
function addFuncE(){
    console.log("hello");
}();
```

在JavaScript中函数表达式十分容易被误认为是函数定义的语法，区分了上面两种情况，你会发现这是在编程过程中十分危险，JavaScript中的函数语法有如下两条规则：

1.当函数语法成为表达式的一部分时，其会被转换成为函数表达式。

2.内嵌于非函数的其他代码块中时，函数语法会被转换成函数表达式。

例子如下：

```javascript
//函数语法被转换成函数表达式
//1.函数语法成为表达式的一部分
(function f1(){console.log("f1")})();
//2.函数语法内嵌于非函数的代码块汇总
if (true) {
    (function f2(){
        console.log("f2");
    })();
}
```

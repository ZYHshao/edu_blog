---
title: JavaScript基础之三——基本运算符
date: 2016-12-30
categories: 前后端
tags: []
---
## JavaScript基础之三——基本运算符

    大多数语言支持的基本运算符都差别不大。其中最常用的莫属赋值运算符，编程初学者总是会将赋值运算符与相等运算符混淆，需要注意，赋值运算符用于将等号右侧的值赋值给等号左侧的变量，示例如下：

```javascript
//赋值运算符
var v = 10;
```

    基本的算术运算符在JavaScript中都是支持的，示例如下：

```javascript
//+加法运算符
var sum = 4+5;
console.log(sum);
//加法运算符也可以用于字符串之间的拼接
var str = "Hello" + "World";
//如果把数字与字符串进行相加 结果为字符串
str = str + sum;
console.log(str);
//-减法运算符
var sub = 10-2;
console.log(sub);
//*乘法运算符
var mul = 5*2;
console.log(mul);
//\除法运算符
var dev = 30/4;
console.log(dev);
//%取余运算符
var rem = 10.5%2.5;
console.log(rem);
rem = 10%3;
console.log(rem);
```

JavaScript语言中的取余运算符十分强大，其不仅可以用于整数间的取余运算，也可以用于小数间的取余运算(Swift2.2以前的版本也可以支持浮点数取余运算，后面的版本将这个特性删掉了)。

    除了前面列举的算术运算符外，JavaScript也支持递增与递减运算符，和C中的此类运算符用法一致，其可以放在操作数前也可以放在操作数后。通俗的理解，当运算符放在操作数前表示先进行递增或递减，再将结果返回；当运算符放在操作符后表示先将操作数的值返回，再进行递增或递减操作，演示如下：

```javascript
//累加
var t1 = 5;
console.log(t1++);
console.log(t1);
console.log(++t1);
console.log(t1);
//递减
var t2 = 5;
console.log(t2--);
console.log(t2);
console.log(--t2);
console.log(t2);
```

    如果将赋值运算符与算术运算符结合起来，就组成了复合赋值运算符，复合运算符将接收的变量本身进行算术运算后返回，示例如下：

```javascript
//复合运算符
var t3 = 5;
t3+=1;
t3-=1;
t3*=2;
t3/=2;
t3%=2;
```

    在条件与循环结构中，逻辑表达式十分重要，逻辑运算符是构成逻辑表达式的基础，在编程的世界中逻辑值只有两个，非真即假。比较运算符会返回一个逻辑值，JavaScript中支持的比较运算符如下：

```javascript
//比较运算符
//比较值是否相等 false
console.log(3==4);
//当数字和字符串进行比较时  只对值是否相等进行比较 true
console.log(3=='3');
//全等比较 值和类型都相等才返回true  下面示例为fasle
console.log(3==='3');
//是否不等 只比较值
console.log(3!='3');
//是否不全等 值和类型都比较
console.log(3!=='3');
//小于比较
console.log(3<4);
//大于比较
console.log(3>4);
//不大于比较
console.log(3<=4);
//不小于比较
console.log(3>=4);
```

上面列举的比较运算符中的“==”与“===”需要注意，前者是对值进行比较，并不比较类型，后者除了比较值之外，还会对类型进行比较。   

    JavaScript中支持的逻辑运算符有与运算符，或运算符和非运算符，示例如下：

```javascript
//进行与运算 有1个为false则为false  都为true才为true
console.log(false&&true);
console.log(true&&true);
//进行或运算 有1个为true则为true 都为false才为false
console.log(false||true);
console.log(false&&false);
//进行非运算
console.log(!false);
console.log(!true);
```

     JavaScript中还有一个运算符十分常用，条件运算符(问号冒号运算符)通常可以用来代替简单的条件语句，示例如下：

```javascript
//条件运算符
var a;
a = true?"aaa":"bbb";
console.log(a);
var b;
b = false?"aaa":"bbb";
console.log(b);
```

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536

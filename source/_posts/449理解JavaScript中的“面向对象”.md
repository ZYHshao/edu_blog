---
title: 理解JavaScript中的“面向对象”
date: 2022-08-05
categories: 前后端
tags: []
---
# 理解JavaScript中的“面向对象”

## 一 引子

面向对象，是程序开发者再熟悉不过的一个概念。一说到它，你首先会想到的是什么？类？继承？方法与属性？不同技术栈的开发者或许有不同的第一反应。面向对象本身只是一种编程方式，支持面向对象的语言很多，但其实现原理却并不都一样。大多数语言的面向对象特性都是基于“类”来实现的，例如C++，Objective-C，Java，Python等。在这些语言中，类是面向对象的基础，面向对象的继承，多态，封装也是通过类来实现的。

本篇文章，我们要讲的是编程语言中的一个异类：JavaScript。JavaScript不仅是面向对象语言，而且是实实在在的面向对象，除了基础类型，你所用到的一些数据实例都是对象，包括函数。但是，JavaScript中的面向对象却并不是基于类来实现的，这对熟悉其他编程语言的开发者来说，是有些反直觉的。完全了解JavaScript中的面向对象， 能够帮助你更好的理解JavaScript这门语言的工作原理。现在，TypeScript越来越热门，越来越多的应用框架开始以TypeScript作为首选语言，TypeScript只是JavaScript基于编译层面的封装，因此了解了JavaScript的原理，也能让你更好更高效的使用TypeScript。

## 二 “面向对象”的编程方式是如何演进而来的

什么是面向对象编程？我们可以从其3个特点说起：

![](https://oscimg.oschina.net/oscnet/up-be2fc50c1a290270c0b030944d87b4e24ae.png)

> **封装**
> 
> 封装是将数据和逻辑捆绑到一起，对象的某些数据和逻辑方法可以是私有的，不能被外界访问，以此实现对数据和逻辑方法不同级别的访问权限。防止了程序相互依赖性而带来的变动影响，面向对象的封装比传统语言的封装更为清晰、更为有力。有效实现了两个目标：对数据和行为的包装和信息隐藏。

> **继承**
> 
> 继承简单地说就是一种层次模型，这种层次模型能够被重用。层次结构的上层具有通用性，但是下层结构则具有特殊性。在继承的过程中类则可以从最顶层的部分继承一些方法和变量。类除了可以继承以外同时还能够进行修改或者添加。通过这样的方式能够有效提高工作效率。

> **多态**
> 
> 多态性是指相同的操作或函数、过程可作用于多种类型的对象上并获得不同的结果。不同的对象，收到同一消息可以产生不同的结果，这种现象称为多态性。
> 
> 封装，继承，多态组成了面向对象编程的基本特性，各种面向对象语言的设计也是围绕这3个特性展开的。

我们可以通过一个简单的例子来理解。例如我们需要编写程序来计算一个圆的面积，我们知道，计算圆的面积需要两个参数，圆周率与半径。按照面向过程的思路，最原始的代码可能是这样的：

```javascript
let radius = 2;
let pi = 3.14;
function area(){
    return pi * radius * radius;
}

console.log(area());
```

此时，数据和逻辑非常分散且独立，为了更好的实现编程的封装性，我们可以将计算圆面积所使用的变量和方法都聚合到一个对象中，通过对象来整合数据与调用方法，如下：

```javascript
let circle = {
    radius:2,
    pi:3.14,
    s:function(){
        return this.pi * this.radius * this.radius
    }
}

console.log(circle.s());
```

这样，我们其实就已经将面向过程的编程方式转换成了面向对象的编程方式。封装本身也是一种对现实生活事物的抽象，如上代码中所示，circle对象从意义上已经能够描述图形圆这样一种事物，它拥有半径，占有面积。从变量和函数到属性和方法，编程方式已经发生了转变。但是，直接创建对象的方式实现封装依然非常低级，思考一下，现实生活中会有各种各样的圆，半径不同的圆不可胜数，同样在程序中我们也会使用到各种各样的“圆”对象，每次都如此创建对象非常繁琐，且如果逻辑有了修改，所有的对象都要修改，代码将逐渐变得不可维护。因此，我们想到使用模板方法来创建对象“圆”，使得此对象的创建逻辑更加聚合，修改和扩展都将更方便。优化上面的代码如下：

```javascript
function Circle(radius) {
    let circle = {};
    circle.radius = radius;
    circle.pi = 3.14;
    circle.s = function() {
        return this.radius * this.radius * this.pi;
    }
    return circle;
}

var c1 = Circle(1);
var c2 = Circle(2);
console.log(c1.s());
console.log(c2.s());
```

上面示例代码中对圆对象的创建进行了封装，这就好像Circle是一个模板，使用此模板我们可快速的创建出各种半径不同的圆对象，从另一个角度理解，Circle也可以理解为是一种类型，通过此类型生成的对象都是“圆”对象。引入了“类”的思想，编程便能更好的模拟现实事物了。到此，你会发现，在JavaScript中，“类”是可以通过函数的方式来实现的。那么现在，我们再来更深入的讨论基于“类”的更多优化技巧。

通过函数实现“类”的编程方式已经进步了很多，但是还不够优秀。我们知道，无论是数据还是函数，都是要在内存中占据空间的，对象中有包装属性和方法，其实就是存储数据和函数，对于数据来说，每个对象中存储的可能不同，例如上面的圆半径，也可能所有对象存储的都一样，例如上面的圆周率。对于方法来说，应该是所有对象所共享的，就像所有圆的计算面积的方法都是一样的。如果每个对象中都存储完整的函数方法，则这不仅非常多余，更是增加了无谓的内存消耗。如何优化这一点呢，我们可以想到，设计模式中的原型模式正是为处理此类问题而生的。

将方法放入原型对象中，通过对原型对象的引用来让所有实例对象共享原型对象中封装的方法。修改代码如下：

```javascript
var prototype = {
    s:function () {
        return this.radius * this.radius * this.pi;
    }
}

function Circle(radius) {
    let circle = {};
    circle.radius = radius;
    circle.pi = 3.14;
    circle.s = prototype.s
    return circle;
}

var c1 = Circle(1);
var c2 = Circle(2);
console.log(c1.s());
console.log(c2.s());
// true
console.log(c1.s === c2.s);
```

修改后的代码，无论生成多少对象，方法都是共享的。其他语言中如果要共享属性，通常提供的有类属性或静态属性的语法，核心原理是将共享的属性绑定到类上而不是对象上，JavaScript中也可以这么做，如下：

```javascript
var prototype = {
    s:function () {
        return this.radius * this.radius * Circle.pi;
    }
}

function Circle(radius) {
    let circle = {};
    circle.radius = radius;
    Circle.pi = 3.14;
    circle.s = prototype.s
    return circle;
}
```

最后，我们还要想办法实现继承和多态，这样才算比较完善的面向对象。多态比较简单，我们只要确定父子类方法的优先级关系，即可通过方法覆写实现多态，继承也很好理解，通过一种方式将父类的属性和方法关联到子类中，即可实现继承。我们可以编写如下代码：

```javascript
function RedUnitCircle() {
    let circle = {};
    circle.parent = Circle(1);
    circle.radius = 1;
    circle.color = "red";
    return circle;
}

var c3 = RedUnitCircle();
if (c3.s === undefined) {
    console.log(c3.parent.s());
} else {
    console.log(c3.s());
}
```

红色单位圆类是对圆类的一种派生，其内部增加了color属性，同样它也有计算面积的方法，我们使用parent属性来承接，实际上，在调用方法时，首先需要在当前对象上找，如果不存在，在向继承链上一层一层的递归寻找，直到最上层的基类。事实上，在JavaScript中，是通过原型链来实现继承关系的，我们下面将会介绍。

## 三 this究竟指向哪里

在使用JavaScript编程中，我们经常会使用到this，在JavaScript函数调用时，函数内部会默认被传入一个隐含的this参数，此this参数指向一个对象，我们可以称之为上下文对象，那么如何确定this指向的对象究竟是谁呢？可以分为如下几种情况来讨论。

1\. 当函数被直接调用时，this指向golbal全局对象（node.js中），如果是在浏览器环境运行，则指向window对象。

例如：

```javascript
function testFunc() {
    console.log(this);
}
// global对象
testFunc();
```

2\. 当函数作为方法被调用时，this指向调用方法的对象。

例如：

```javascript
var obj = {
    name:"name",
    log:function(){
        console.log(this.name);
    }
}
// name
obj.log();
```

3\. 当被作为构造函数被执行时，this指向新创建的对象。

例如：

```javascript
function CreateObj() {
    this.name = "name";
}
var obj = new CreateObj();
// name
console.log(obj.name);
```

4\. 使用bind，apply和call函数，能够显式的指定函数中的this指向。

例如：

```javascript
var obj = {
    name:"name"
}

function log(p) {
    console.log(this.name, p);
}
function log2(p) {
    console.log(this.name, p);
}
function log3(p) {
    console.log(this.name, p);
}
// name Param
log.call(obj,'Param')
// name Param
log2.apply(obj,['Param']);
var newLog3 = log3.bind(obj);
// name Param
newLog3('Param');
```

其中，call和apply函数的作用基本一致，只是其传参的类型不同，bind函数用来生成一个新的函数，此函数中的this是被绑定到对应的对象上的。

由于JavaScript的这种特性，我们在函数中使用this时，常常会收到运行时的调用环境影响。在ES6中提供了箭头函数的支持，在箭头函数中，this会被绑定为函数定义时的this指向，例如：

```javascript
function create(){
    return function() {
        console.log(this);
    }
}
function create2(){
    return () => {
        console.log(this);
    }
}
var f = {name:"name", log:create(),log2:create2()};
//{name: 'name', log: ƒ, log2: ƒ}
f.log();
// global
f.log2();
```

TypeScript或其他支持EX6的编译器也会将箭头函数编译成如下样式，非常清楚的描述了this在箭头函数中的指向：

```javascript
function create2() {
    var _this = this;
    return function () {
        console.log(_this);
    };
}
```

## 四 理解JavaScript中的面向对象

前面说了这么多，我们终于要涉及到JavaScript是如何实现面向对象的特性了。前面有提到，我们可以使用函数来模拟类，构建出对象，继承和多态特性则是通过原型设计模式实现。JavaScript本身其实就有这样的一套能力，任何一个函数，当我们使用new关键字进行调用时，其都会被当成构造函数。构造函数的执行过程可以分为如下3步：

1\. 创建一个空的对象，并让构造函数内的this指向此对象。

2\. 将空对象的\_\_proto\_\_指向构造函数的prototype对象。

3\. 执行构造函数，如果构造函数结果返回的是基本数据类型（包括无返回语句），则会被忽略，直接返回创建的对象。如果返回的是对象类型，则忽略之前创建的对象。

在使用对象访问属性或方法时，会先从当前对象中查找，如果没有找到，会再向原型\_\_proto\_\_中进行查找，一层层向上，直到找到对应的属性方法或没有更上层的原型对象。例如：

```javascript
function People(name) {
    this.name = name;
}

People.prototype.sayHi = function(){
    console.log("Hello I am " + this.name);
}

var jaki = new People("Jaki");
// Hello I am Jaki
jaki.sayHi();
```

要实现继承，可以通过修改原型来实现，如下：

```javascript
function People(name) {
    this.name = name;
}

People.prototype.sayHi = function(){
    console.log("Hello I am " + this.name);
}

function Teacher(name, subject) {
    People.call(this, name);
    this.subject = subject;
}
// 修改子类原生指向
function TeacherPrototype(){
    // 重写父类的方法
    this.sayHi = function() {
        // 调用父类方法
        TeacherPrototype.prototype.sayHi.call(this);
        // 子类自己的实现
        console.log("教学："+this.subject);
    }
}
// 子类原型设置
TeacherPrototype.prototype = People.prototype
Teacher.prototype = new TeacherPrototype()

var teacher = new Teacher("Jaki", "JavaScript");
//Hello I am Jaki
//教学：JavaScript
teacher.sayHi();

```

从效果可以看到，JavaScript虽然没有类，但依然完成了面向对象的核心特性功能。

## 五 再来深入理解下原型链

当我们创建一个空对象时，实际上是调用了Object构造方法，因此空对象的\_\_proto\_\_原型是指向Object函数的prototype的，如下：

```javascript
var a = {};
var b = new Object();
// {} {}
console.log(a, b);
// true
console.log(a.__proto__ === Object.prototype);
```

在JavaScript中，函数本身也是对象，函数对象的原型是Function的prototype，如下：

```javascript
var f = function(){
}
var f2 = new Function();
console.log(f, f2);
// true
console.log(f.__proto__ === Function.prototype);
```

Object本身也是一个函数，因此Object的\_\_proto\_\_原型指向Function的prototype，同理，Function也是一种对象，因此Function的\_\_proto\_\_指向Object的prototype，在向上Object的prototype对象的\_\_proto\_\_就是null了，例如：

```javascript
// true
console.log(Object.__proto__ === Function.prototype);
// true
console.log(Function.__proto__ === Function.prototype);
// true
console.log(Function.prototype.__proto__ === Object.prototype);
// null
console.log(Object.prototype.__proto__);
```

原型链看上去有些复杂，借用网上的一张描述图，可以清晰的看出其间关系：

![](https://oscimg.oschina.net/oscnet/up-c857a3778607dca5e7df785ad7d1601ea95.gif)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> —— 珲少 QQ：316045346
> 
> 同时，如果本篇文章让你觉得有用，欢迎分享给更多朋友，请标明出处。
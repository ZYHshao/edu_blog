---
title: Swift专题讲解十八——异常处理
date: 2016-05-26
categories: Swift语法专题
tags: []
---
## Swift专题讲解十八——异常处理

### 一、异常的抛出与传递

        代码的运行很多时候并不会完全按照程序员的设想进行，编写代码时进行可控的异常处理机制是十分必要的。通常，对于一个特定的操作，程序员可以定义一个继承自ErrorType的枚举来进行异常类型的描述，使用throw关键字来进行异常的抛出，示例代码如下：

```javascript
//定义一个自定义的错误类型
enum MyError:ErrorType {
    case DesTroyError
    case NormalError
    case SimpleError
}
//进行异常的抛出
throw MyError.NormalError
```

函数可以进行错误的传递，需要使用throws关键字来声明这个函数可能会抛出错误，如果不如此声明，则函数内部抛出的错误只能在函数内部解决，throws关键字标记的函数内部抛出的错误会被传递到调用函数的地方，开发者可以在调用函数的地方捕获到错误描述来做相应处理，示例如下：

```javascript
func MyFunc()throws ->  Void {
    throw MyError.NormalError
}
```

对于可能抛出异常的函数调用，开发者要么在调用函数的地方捕获处理这些异常，要么使用try关键字将异常继续抛出去，等待下一层捕获者处理。异常的处理后面会介绍，继续抛出异常示例如下：

```javascript
try MyFunc()
```

### 二、异常的处理

        除了将错误继续向上抛出之外，Swift还提供了3种处理异常的方式。

#### 1.使用do-catch语句来捕获异常

        开发者可以使用do-catch语句来捕获异常，通过异常类型的判断来分别做处理，示例代码如下:

```javascript

do{
   try MyFunc()
}catch MyError.DesTroyError{//将打印error1
    print("error1")
}catch MyError.NormalError{
     print("error2")
}catch MyError.SimpleError{
     print("error3")
}catch{//如果上面所有的catch都没有捕获 会走这个异常捕获判断
    print("all")
}
```

#### 2.将异常映射为Optional值

        处理异常抛出的第2中方式是使用try?将异常映射为Optional值，可以简单理解为，对一个可能抛出异常的函数的调用，如果有异常抛出，则返回值为nil，如果没有，则函数顺利执行，返回值为其原返回值，示例如下:

```javascript
//将返回nil
try? MyFunc()
```

注意：返回值为Void并非为nil，结合if-let语句可以编写十分飘逸的代码，示例如下：

```javascript
if let _=try?MyFunc() {
    print("success")
}else{
    print("fail")
}
```

#### 3.终止异常传递

        有时候开发者可以保证一个可能抛出异常的函数绝对不会抛出异常，这时开发者可以使用try!的方式来终止异常的传递，但是这样做有一定风险，如果这个函数真的抛出了异常，则会产生运行时错误。示例如下：

```javascript
try! MyFunc()
```

### 三、延时执行语句

        对于某些释放资源类的操作，开发者总是希望其离开当前代码块时被执行，然后一个复杂流程结构可能会因异常抛出，return，break这些方式被终止，因此，Swift中提供了defer语句来进行延时执行一些操作，defer中的语句总是会在当前代码块将要结束时才执行，无论它是以哪种方式结束的，示例如下：

```javascript
//执行此函数将打印
/*
 Care
 finish
 */
func MyFunc()throws ->  Void {
    
    defer{
        print("finish")
    }
    print("Care")
    throw MyError.DesTroyError
    
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

---
title: Swift讲解专题六——流程控制
date: 2016-05-12
categories: Swift语法专题
tags: []
---
## Swift讲解专题六——流程控制

### 一、引言

        一种编程语言的强大与否，很大程度上取决于其提供的程序流程控制方案，就如使用汇编语言实现复杂的程序流程是一件痛苦的事情。Swift中提供了许多强大的流程控制语句，例如快速遍历for-in，while循环，repeat-while循环，switch选择等，需要注意的是，在Swift2.2中，for(a;b;c)循环已经被弃用掉，并且Swift中的Switch语句也更加强大，可以处理任意数据类型。

### 二、for-in循环

        配合范围运算符，for-in循环可以用来执行确定次数的循环，示例如下：

```
for index in 1...5 {
    print(index)
}
//如果不需要获取循环中每次的循环次数 可以使用如下方式
var sum=0;
for _ in 1...3 {
    sum += 1
}
```

for-in循环也通常会用来遍历数组，字典，集合等，示例如下：

```
var collection1:Array = [1,2,3,4]
var collection2:Dictionary = [1:1,2:2,3:4,4:4]
var collection3:Set = [1,2,3,4]
for obj in collection1 {
    print(obj)
}
for (key , value) in collection2 {
    print(key,value)
}
for obj in collection3 {
    print(obj)
}
```

### 三、while循环

        while语句进行循环操作，直到循环条件为false为止，这类型的循环通常适用于循环次数不定的循环需求，while循环提供两种语法格式，示例如下：

```
var i=0
//当i不小于10时跳出循环
while i<10 {
    print("while",i)
    i+=1
}
//先进行一次操作 在判断循环条件
repeat {
    print("repeat while")
} while i<10
```

### 四、if语句

        if语句是程序开发中最常用的语句之一，通过if将判断一个条件是否成立来进行程序的流程控制，if语句通常会和else语句结合进行使用，示例如下：

```
var c:Int
if 1>2 {
    c=1
}else if 1<0 {
    c=2
}else{
    c=3
}
```

### 五、Switch语句

        Switch语句作为开关选择语句，用来处理一组值的分支选择，Swift中的Switch语句格外强大，相比于Objective-C，Swift中的Switch语句每个case后不需要使用break进行手动中断，当代码匹配到一个case后语句将自行中断。用法示例代码如下：

```
var charac:Character = "b"
//使用switch语句进行字符分支判断
switch charac {
case "a":
    print("chara is a")
case "b":
    print("chara is b")
case "c":
    print("chara is c")
default ://default用于处理其他额外情况
    print("no charac")
}
//同一个case中可以包含多个分支
switch charac {
case "a","b","c" :
    print("chara is word")
case "1","2","3" :
    print("chara is num")
default :
    print("no charac")
}
//在case中也可以使用一个范围
var num = 3
switch num {
case 1...3 :
    print("1<=num<=3")
case 4 :
    print("chara is num")
default :
    print("no charac")
}
//使用Switch语句进行元组的匹配
var tuple = (0,0)
switch tuple {
case (0,1):
    print("Sure")
    //也可以只对元组中的某个元素进行匹配
case (_,1):
    print("Sim")
    //也可以对元组中的元素进行范围匹配
case(0...3,0...3):
    print("SIM")
default:
    print("")
}
//进行数据绑定
switch tuple {
case (let a,1):
    print(a)
case (let b,0):
    print(b)
    //let(a,b) 与 (let a,let b)意义相同
case let(a,b):
    print(a,b)
default:
    print("")
}
//对于进行了数据绑定的Switch语句 可以使用where关键字来进行条件判断
switch tuple {
case (let a,1):
    print(a)
case (let b,0):
    print(b)
//let(a,b) 与 (let a,let b)意义相同
case let(a,b) where a==b:
    print(a,b)
default:
    print("")
}

```

### 六、跳转语句

        Swift中提供了5种跳转语句，continue，break，fallthrough，return，throw。

continue：跳出到循环起始位置，直接开始下次循环。

break：break如果在循环语句中则是直接中断循环，跳出，若是在Switch结构中，则立即跳出Switch结构。

fallthrough语句需要和switch语句配合使用，在case中使用fallthrough，则会继续执行下一个case，需要注意，在下一个case中有进行数据绑定的，不可以使用fallthrough，示例如下：

```
var tuple = (0,0)
switch tuple {
case (0,0):
    print("Sure")
    //fallthrough会继续执行下面的case
    fallthrough
    //也可以只对元组中的某个元素进行匹配
case (_,0):
    print("Sim")
    fallthrough
    //也可以对元组中的元素进行范围匹配
case(0...3,0...3):
    print("SIM")
default:
    print("")
}
```

return：return语句直接从函数中返回。

throw：throw用于抛出异常。

        Swift还支持另一种语法，可以为while循环设置一个tip标签，使用break和continue等关键字来进行流程的控制，示例如下：

```
var tmp = 0;
tip:while tmp<10 {
    print("ccc")
    tmp+=1
    switch tmp {
    case 3:
        break tip
    default:
        break
    }
}
```

        Swift2.0之后，提供了一种新的语法，guard-else，这也被称作守护语句，只有当条件不满足时，才执行else后面的代码，示例如下：

```
var name = "HS"
func nameChange(name:String) {
    guard name=="HS" else{
        print(name)
        return
    }
    print("name is HS")
}

nameChange(name)

```

在开发中，函数中常常会需要检查传入的参数是否符合标准，guard-else语句就是为这种需求所生，正如其名，它用于守护函数执行的精确度。

### 七、系统版本检查

        使用如下示例代码进行系统支持版本的检查：

```
if #available(iOS 9, *){
    print("iOS 9")
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

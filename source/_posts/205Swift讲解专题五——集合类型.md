---
title: Swift讲解专题五——集合类型
date: 2016-05-11
categories: Swift语法专题
tags: []
---
## Swift讲解专题五——集合类型

### 一、引言

        Swift中提供了3种集合类型，Array数据类型，Set集合类型，Dictionary字典类型。Array用于存放一组有序的数据，数据角标从0开始一次递增；Set用于存放一组无序的数据，数据不可以重复；Dictionary也用于存放一组无序的数据，只是其是按照键值对的方式存储，键值必须唯一。这里借用官方文档中的一张图来表示3种集合类型的特点：

![](http://static.oschina.net/uploads/space/2016/0511/112738_Qddl_2340880.png)

### 二、Array类型

        Array通常也被称为数组，Swift是一种类型安全语言，其中的Array类型也必须确定其元素的类型，声明数组类型有两种方法，示例如下：

```
//将数组声明为Int类型值集合的数组
var array1:[Int]
var array2:Array<Int>
//创建空数组
array1 = []
array2 = Array()
```

数组对象如果通过var变量也接收，则其为可变的数组，可以通过append方法来追加元素，示例如下：

```
//向数组中追加元素
array1.append(3)
```

在创建数组时，也可以对数组进行初始化，示例如下：

```
//创建数组[0,0,0]
var array3 = [Double](count: 3, repeatedValue: 0)
//创建数组[2.5,2.5,2.5]
var array4 = Array(count: 3, repeatedValue: 2.5)
//数组可以使用+号直接进行追加 [0,0,0,2.5,2.5,2.5]
var array5 = array3+array4
```

Swift中提供了许多访问和修改数组的方法，示例代码如下：

```
//获取数组中元素个数
array5.count
//判断数组是否为空
array5.isEmpty
//通过下标访问数组中的元素
array5[1]
//通过下标修改数组元素
array5[1]=2
//修改数据中的一组数据
array5[0...3] = [1,1,1,1]
//向数组中某个位置插入一个数据
array5.insert(3, atIndex: 1)
//移除数组某个角标处的元素
array5.removeAtIndex(1)
//移除数组的最后一个元素
array5.removeLast()
//移除数组第一个元素
array5.removeFirst()
//遍历整个数组
for item in array5 {
    print(item)
}
//遍历数组枚举
for (index,item) in array5.enumerate() {
    print(index,item)
}

```

### 三、Set类型

        Set类型集合不关注元素的顺序，但是其可以保证其中元素的唯一性。和Array类型一样，Set类型来声明时也需要确定其内元素的类型，示例如下：

```
var set1:Set<Character> = ["a","b","c","d"]
```

下面示例代码演示对集合进行操作：

```
var set1:Set<Character> = ["a","b","c","d"]
var set2:Set<Character> = ["e","f","g"]
//向集合中插入元素
set1.insert("z")
//获取集合中元素个数
set1.count
//判断集合是否为空
set1.isEmpty
//将集合中的某个元素移除
set1.remove("a")
//移除集合中的所有元素
set1.removeAll()
//判断集合中是否包含某个元素
set2.contains("e")
//遍历集合
for item in set2 {
    print(item)
}
//进行从小到大的排序遍历
for item in set2.sort() {
    print(item)
}

```

Set也支持进行一些集合的数学运算，例如交集，并集，补集等，下面一张图演示了Set进行集合运算的一些特性：

![](http://static.oschina.net/uploads/space/2016/0511/122935_Fb1k_2340880.png)

intersect()方法返回两个集合的交集。

exclusiveOr()方法用于返回两个集合交集的补集。

union()方法用于返回两个集合的并集。

subtract()方法用于返回第二个集合的补集。

示例代码如下：

```
var set3:Set<Int> = [1,2,3,4]
var set4:Set<Int> = [1,2,5,6]
//返回交集 {1，2}
var setInter = set3.intersect(set4)
//返回交集的补集{3，4，5，6}
var setEx = set3.exclusiveOr(set4)
//返回并集{1，2，3，4，5，6}
var setUni = set3.union(set4)
//返回第二个集合的补集{3，4}
var setSub = set3.subtract(set4)
```

使用比较运算符==可以比较两个Set集合是否相等，当两个Set集合中所有元素都相等时，这两个集合才相等。下面代码显示了与子集相关的运算：

```
var set5:Set = [1,2]
var set6:Set = [2,3]
var set7:Set = [1,2,3]
var set8:Set = [1,2,3]
//判断是否是某个集合的子集 set5是set7的子集 返回ture
set5.isSubsetOf(set7)
//判断是否是某个集合的超集 set7是set5的超集 返回ture
set7.isSupersetOf(set5)
//判断是否是某个集合的真子集 set5是set7的真子集 返回ture
set5.isStrictSubsetOf(set7)
//判断是否是某个集合的真超集 set7不是set8的真超集 返回false
set7.isStrictSupersetOf(set8)
```

### 四、Dictionary类型

        Swift中的Dictionary在声明时必须明确键的类型和值的类型，示例如下：

```
var dic:Dictionary<Int,String>
var dic2:[Int:String] = [1:"one",2:"Two"]
```

访问与操作Dictionary的方法，代码示例如下：

```
var dic2:[Int:String] = [1:"One",2:"Two",3:"Three",4:"Four"]
//获取字典键值对个数
dic2.count
//判断字典是否为空
dic2.isEmpty
//通过键获取值
dic2[1]
//通过键修改值
dic2[1] = "First"
//添加键值
dic2[0] = "Zero"
//updateValue 方法将更新一个键值 如果此键存在 则更新键值 并且将旧的键值返回 如果此键不存在 则添加键值 返回nil 其返回的为一个Optional类型值 可以使用if let进行处理
dic2.updateValue("9", forKey: 1)
//使用if let 处理updateValue的返回值
if let oldValue = dic2.updateValue("One", forKey: 1) {
    print("Old Value is \(oldValue)")
}
//通过键值获取的数据也将是有个Optional类型的值 也可以使用if let
if let value = dic2[1] {
    print("The Value is \(value)")
}
//移除某个键值对
dic2[9]=nil
dic2.removeValueForKey(9)
//对字典进行遍历
for (key,value) in dic2 {
    print(key,value)
}
//遍历所有键
for key in dic2.keys {
    print(key)
}
//遍历所有值
for value in dic2.values {
    print(value)
}
//进行从小到大的排序遍历
for key in dic2.keys.sort() {
    print(key)
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

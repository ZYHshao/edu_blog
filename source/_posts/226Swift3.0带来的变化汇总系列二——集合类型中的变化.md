---
title: Swift3.0带来的变化汇总系列二——集合类型中的变化
date: 2016-06-18
categories: Swift语法专题
tags: []
---
## Swift3.0带来的变化汇总系列二——集合类型中的变化

    与字符串类似，Swift中集合的类型在3.0版本中也做了大量API上面的修改。

### 一、Array数组的更改

    array数组中修改的API示例如下：

```swift
//创建大量相同元素的数组
//创建有10个String类型元素的数组，并且每个元素都为字符串"Hello"
//swift2.2
//var array3 = [String](count: 10, repeatedValue: "Hello")
//swift3.0
var array3 = [String](repeating: "Hello", count: 10)
//创建有10个Int类型元素的数组，且每个元素都为1
//swift2.2
//var array4 = Array(count: 10, repeatedValue: 1)
//swift3.0
var array4 = Array(repeating: 1, count: 10)

var array = [1,2,3,4,5,6,7,8,9]
//向数组中追加一组元素
//swift2.2
//array.appendContentsOf([11,12,13])
//swift3.0
array.append(contentsOf: [11,12,13])
//向数组中的某个位置插入一个元素
//swift2.2
//array.insert(0, atIndex: 0)
//swift3.0
array.insert(0, at: 0)
//向数组中的某个位置插入一组元素
//swift2.2
//array.insertContentsOf([-2,-1], at: 0)
//swift3.0
array.insert(contentsOf: [-2,-1], at: 0)
//移除数组中某个位置的元素
//swift2.2
//array.removeAtIndex(1)
//swift3.0
array.remove(at: 1)
//移除一个范围内的元素
//swift2.2
//array.removeRange(0...2)
//swift3.0
array.removeSubrange(0...2)
//修改一个范围内的元素
//swift2.2
//array.replaceRange(0...2, with: [0,1])
//swift3.0
array.replaceSubrange(0...2, with: [0,1])
//进行数组枚举遍历 将输出 (0,0) (1,1) (2,2) (3,3) (4,4)
//swift3.0 中将枚举属性enumerate 修改为enumerated()方法
for item in arrayLet.enumerated(){
    print(item)
}
var arraySort = [1,3,5,6,7]
//获取数组中的最大值
//swift2.2
//arraySort.maxElement()
//swift3.0
arraySort.max()
//获取数组中的最小值
//swift2.2
//arraySort.minElement()
//swift3.0
arraySort.min()
//从大到小排序
//swift2.2
//arraySort = arraySort.sort(>)
//swift3.0
arraySort = arraySort.sorted(isOrderedBefore: >)
//从小到大排序
//swift2.2
//arraySort = arraySort.sort(<)
//swift3.0
arraySort = arraySort.sorted(isOrderedBefore: <)
```

### 二、Set集合中的更改

    Set集合中的修改示例如下：

```swift
//创建set集合
var set1:Set<Int> = [1,2,3,4]
//进行下标的移动
//获取某个下标后一个元素
//swlft2.2
//set1[set1.startIndex.successor()]
//swift3.0
set1[set1.index(after: set1.startIndex)]
//获取某个下标后几位的元素
//swift2.2
//set1[set1.startIndex.advancedBy(3)]
//swift3.0
set1[set1.index(set1.startIndex, offsetBy: 3)]
//获取集合中的最大值
//swift2.2
//set1.maxElement()
//swift3.0
set1.max()
//获取集合中的最小值
//swift2.2
//set1.minElement()
//swift3.0
set1.min()
//移除集合中某个位置的元素
//swift2.2
//set1.removeAtIndex(set1.indexOf(3)!)
//swift3.0
set1.remove(at: set1.index(of: 3)!)
var set3:Set<Int> = [1,2,3,4]
var set4:Set<Int> = [1,2,5,6]
//返回交集 {1，2}
//swift2.2
//var setInter = set3.intersect(set4)
//swift3.0
var setInter = set3.intersection(set4)
//返回交集的补集{3，4，5，6}
//swift2.2
//var setEx = set3.exclusiveOr(set4)
//swift3.0
var setEx = set3.symmetricDifference(set4)

var set5:Set = [1,2]
var set6:Set = [2,3]
var set7:Set = [1,2,3]
var set8:Set = [1,2,3]
//判断是否是某个集合的子集 set5是set7的子集 返回ture
//swift2.2
//set5.isSubsetOf(set7)
//swift3.0
set5.isSubset(of: set7)
//判断是否是某个集合的超集 set7是set5的超集 返回ture
//swift2.2
//set7.isSupersetOf(set5)
//swift3.0
set7.isSuperset(of: set5)
//判断是否是某个集合的真子集 set5是set7的真子集 返回ture
//swift2.2
//set5.isStrictSubsetOf(set7)
//swift3.0
set5.isStrictSubset(of: set7)
//判断是否是某个集合的真超集 set7不是set8的真超集 返回false
//swift2.2
//set7.isStrictSupersetOf(set8)
//swift3.0
set7.isStrictSuperset(of: set8)
```

### 三、Dictionary字典中的更改

    Dictionary字典中修改示例如下：

```swift
//通过键删除某个键值对
//swift2.2
//dic1.removeValueForKey(1)
//swift3.0
dic1.removeValue(forKey: 1)
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

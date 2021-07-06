---
title: iOS中CoreData数据管理系列一——初识CoreData
date: 2016-01-27
categories: iOS逻辑初窥
tags: []
---
## iOS中CoreData数据管理系列一——初识CoreData

### 一、何为CoreData

    CoreData是一个专门用来管理数据的框架，其在性能与书写方便上都有很大的优势，在数据库管理方面，apple强烈推荐开发者使用CoreData框架，在apple的官方文档中称，使用CoreData框架可以减少开发者50%——70%的代码量，这虽然有些夸张，但由此可见，CoreData的确十分强大。

### 二、设计数据模型

    在iOS开发中，时常使用SQL数据库对大量的表结构数据进行处理，但是SQL有一个十分明显的缺陷，对于常规数据模型的表，其处理起来是没问题的，例如一个班级表，其中每条数据中有班级名称，人数这样的属性，一个学生表，其中每条数据有学生的姓名，性别，年龄这样的属性。但是如果要在表与表之间建立联系，自定义对象与自定义对象之间产生从属关系，使用SQL处理起来就十分麻烦了，例如如果这个班级表中有一个班长的属性，这个属性是一个学生类型。关于iOS中SQL的使用相关博客，地址如下：

Sqlite数据库相关知识：[http://my.oschina.net/u/2340880/blog/600820](http://my.oschina.net/u/2340880/blog/600820)

iOS中sqlite3框架的使用和封装：[http://my.oschina.net/u/2340880/blog/601802](http://my.oschina.net/u/2340880/blog/601802)

    CoreData的一大优势即是其可以方便的在对象之间建立关系。

#### 1.创建实体类型及其属性

    使用Xcode创建一个工程，在工程中新建一个文件，选择Core Data分类中的DataModel创建，如下图：

![](http://static.oschina.net/uploads/space/2016/0127/153925_vkws_2340880.png)

这时在Xcode的文件导航区会出现一个以xcdatamodeld为扩展名的文件，这个文件就是数据模型文件，点击Add Entity按钮添加一个实体类型，取名为SchoolClass，为这个类型添加两个属性，分别为名字name和学生数量stuNum，如下图:

![](http://static.oschina.net/uploads/space/2016/0127/155647_LNhR_2340880.png)

#### 2.对实体类型进行设置

    在Xcode右侧的工具栏中可以对实体类型进行一些设置，选中一个实体类型，如下图：

![](http://static.oschina.net/uploads/space/2016/0127/160413_H3tm_2340880.png)

Name设置实体类型的名称，Abstract Entity设置是否是抽象实体，如果勾选，则此实体不能被实例化，只能被继承，类似于抽象类，比如定义人为一个实体类型，在定义继承于人实体类型的老师、学生等来进行实例化。Parent Entity用来选择父类实体，Class用于设置对应的类的。

#### 3.在实体对象之间建立关系

    再创建一个学生类实体Student，添加name和age两个属性。选中SchoolClass，在其中的Relationships模块中点击+号，来添加一个关系，如下图：

![](http://static.oschina.net/uploads/space/2016/0127/162515_Dvfv_2340880.png)

这时，SchoolClass实体类型中就有了一个Student类型的班长属性。如果切换一下编辑风格，可以更加清晰的看到实体类型之间的关系，如下图：

![](http://static.oschina.net/uploads/space/2016/0127/162855_koUL_2340880.png)

#### 4.对属性和关系进行设置

    选中一个属性或者关系，在右侧的工具栏中可以对属性进行一些设置，如下图：

![](http://static.oschina.net/uploads/space/2016/0127/164054_PMXW_2340880.png)

name设置属性的名字，Optional类型代表可选，即在实例化对象时可以赋值也可以不赋值。Attribute设置属性的数据类型，Default Value设置数据的默认值。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

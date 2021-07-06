---
title: SQLite数据库常用语句及MAC上的SQLite可视化工具MeasSQLlite使用
date: 2016-01-11
categories: 小码工具
tags: []
---
## SQLite数据库常用语句及MAC上的SQLite可视化工具MeasSQLlite使用

### 一、引言

        在移动开发中，通常会用到一些小型的数据库进行数据管理。SQLite是一款十分小巧便捷的数据库，在iOS开发中，原生框架也对其有很好的支持。

### 二、SQLite常用语句

    数据库存在的意义就在于其对数据的整合和管理，所以数据库的核心操作无非是对数据进行增，删，改，查得操作。

#### 1.建立数据表语句

    一个数据库文件中可以由一些表组成，通过下面的语句在数据库文件中创建一张表：

```
create table class(num integer PRIMARY KEY,name text NOT NULL DEFAULT "1班",count integer CHECK(count>10))
```

上面的语句代码可以简化成如下的格式：

create table 表名(参数名1 类型 修饰条件，参数名2，类型 修饰参数，···)

sqlite中支持如下的类型：

smallint 短整型

integer 整型

real 实数型

float 单精度浮点

double 双精度浮点

currency 长整型

varchar 字符型

text 字符串

binary 二进制数据

blob 二进制大对象

boolean 布尔类型

date 日期类型

time 时间类型

timestamp 时间戳类型

关于修饰条件，常用的有如下几种：

PRIMARY KEY：将本参数这个为主键，主键的值必须唯一，可以作为数据的索引，例如编号。

NOT NULL ：标记本参数为非空属性。

UNIQUE：标记本参数的键值唯一，类似主键。

DEFAULT:设置本参数的默认值

CHECK：参数检查条件，例如上面代码，写入数据是count必须大于时才有效。

#### 2.添加数据

使用下面的语句来进行数据行的添加操作：

```
insert into class(num,name,count) values(2,"三年2班",58)
```

上面的语句代码可以简化成如下格式：

insert into 表名(键1，键2，···) values(值1，值2，···)

使用下面的语句进行数据列的添加，即添加一个新的键：

```
alter table class add new text
```

alter table 表名 add 键名 键类型

#### 3.修改数据

使用如下语句来进行改操作：

```
update class set num=3,name="新的班级" where num=1
```

update 表名 set 键1=值1，键2=值2 where 条件

where后面添加修改数据的条件，例如上面代码修改num为1的班级的名字和mun值。

#### 4.删除数据

```
delete from class where num=1
```

delete from 表名 where 条件

上面代码删除num为1的一条数据。

删除一张表适用下面的语句：

```
drop table class
```

drop table 表名

#### 5.查询操作

查询操作是数据库的核心功能，sqlite的许多查询命令可以快捷的完成复杂的查询功能。

查询表中某些键值：

```
select num from class
```

select 键名，键名··· from 表名

查询全部键值数据：

```
select * from class
```

select * from 表名

*是一个全通配符，代表不限个数任意字符

查询排序：

```
select * from class order by count asc
```

select 键名，键名，··· from 表名 order by 键名 排序方式

order by 后面写要进行排序的键名，排序方式有 asc升序 desc降序

查找数据条数与查找位置限制：

```
select * from class limit 2 offset 0
```

select 键名 from 表名 limit 最大条数 offset 查询起始位置

条件查询：

```
select * from class where num>2
```

select 键名 from 表名 where 条件

查询数据条数：

```
select count(*) from class
```

select count(键名) from 表名

去重查询：

```
select distinct num from class
```

select distinct 键名 from 表名

### 三、MesaSQLite的简单使用

        MesaSQLite是一款可视化的SQLite数据库编辑软件，使用十分方便。如下地址是下载链接：[http://pan.baidu.com/s/1sjW6DC5](http://pan.baidu.com/s/1sjW6DC5)。

#### 1.创建数据库文件

打开MesaSQLite软件，在导航栏中选择File，选择弹出菜单中的New DataBase创建一个新的数据库文件，也可以选择Open Database打开一个数据库。

注意：默认创建的数据库文件为rdb格式，手动改成db格式即可。

![](http://static.oschina.net/uploads/space/2016/0111/163153_fg35_2340880.png)

#### 2.创建表

MesaSQLite有两种方式对数据库进行操作，一种是通过sql语句，一种是通过可视化的界面。在SQL Query工具窗口中，可以通过SQL语句对数据库进行操作，如下图：

![](http://static.oschina.net/uploads/space/2016/0111/163815_GvzN_2340880.png)

或者在Structure工具窗口中进行可视化的创建：

![](http://static.oschina.net/uploads/space/2016/0111/164441_w7hj_2340880.png)

#### 3.查询操作

对于数据的查询操作，同样可以通过SQL Query工具通过语句进行查询或者在Content窗口中填写查询条件进行查询，如下：

![](http://static.oschina.net/uploads/space/2016/0111/165229_9XgH_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

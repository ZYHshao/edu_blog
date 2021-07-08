---
title: Mac系统Git生成ssh公钥
date: 2016-04-13
categories: 日常技巧
tags: []
---
## Mac系统Git生成ssh公钥

        在使用Git仓库进行代码管理时，新的电脑上往往需要生成ssh公钥进行匹配，Mac系统生成Git公钥过程如下：

1.检查本机是否已有公钥

在终端中输入如下命令：

```
$ cd ~/.ssh
```

2.如果电脑中有以前遗留的密钥，将其删除掉

使用如下命令：

```
$ mkdir key_backup
$ cp id_rsa* key_backup
$ rm id_rsa*
```

3.生成新的公钥

终端中输入如下命令

```
$ ssh-keygen -t rsa -C "邮箱地址"
```

之后终端会提示几次密码设置，如果设置了密码，在向Git仓库进行代码交互操作时需要键入密码，也可以全部回车带过，表示不需要密码。

4.向Git仓库中导入公钥

在.ssh文件夹下使用ls命令查看所有文件，可以看到生成了一个id_rsa.pub的文件，使用vi工具打开它，将其内容复制出来，在Git仓库中新建公钥，复制上去即可。例如github中导入密钥过程如下图：

![](http://static.oschina.net/uploads/space/2016/0413/172839_dvtw_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

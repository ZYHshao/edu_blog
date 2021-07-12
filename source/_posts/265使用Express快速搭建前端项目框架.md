---
title: 使用Express快速搭建前端项目框架
date: 2016-11-27
categories: 前后端
tags: []
---
## 使用Express快速搭建前端项目框架

    Express是基于Node.js的前端Web开发框架，使用其可以简洁快速的创建健壮友好的API服务。在前端或移动端的开发过程中，可以借助Express的这项功能模拟API数据，方便开发调试。

    Express是基于Node.js平台的，因此在安装Express之前，需要先安装Node.js。使用如下命令来检查系统中所安装的node版本：

```
node -v
```

如果系统中没有安装Node.js，可以在如下网站进行下载安装：

[https://nodejs.org/en/](https://nodejs.org/en/)。

    创建一个测试工程目录，用于存放Express项目框架，首先在终端，使用如下命令进行Express的全局安装：

```
npm install express-generator -g
```

需要注意，很多时候国内网络使用npm的时候会非常慢，可以通过如下命令来修改仓库源。

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

之后使用cnpm来进行包的安装：

```
cnpm install express-generator -g
```

    安装好Express后，在新建的文件夹目录下，执行Express的初始化：

```
express
```

如果文件夹不为空，会提示是否继续操作，输入yes后回车即可。

    初始化完成后的Express项目结构如下：

![](https://static.oschina.net/uploads/space/2016/1127/113528_uaMm_2340880.png)

其中会默认创建一个package.json文件，其中会添加许多依赖包，在项目目录中执行如下命令来安装这些依赖：

```
npm install
```

依赖安装完成后，工程中会多一个node_modules的文件夹，里面是所有依赖包文件。

    再来看Express模板中的文件，其中bin文件夹下面的www.js文件是服务的启动文件，其中启动了HTTP的服务，默认端口为3000。routes文件夹下面的文件用于配置api路由，默认有index.js与users.js两个。app.js文件中对api进行了初始化与配置。可以在users.js中添加一个测试api如下：

```javascript
var express = require('express');
var router = express.Router();

/* 这个是默认生成的. */
router.get('/', function(req, res, next) {
  res.send('respond with a resource');
});
/* 添加一个测试api*/
router.get('/testAPi',function(rep,res,next){
  res.send('{name:jaki,age:24}');
});
module.exports = router;
```

在项目目录下通过终端执行如下命令来将服务开启：

```
node bin/www
```

如果服务启动成功，在浏览器输入http://127.0.0.1:3000/users/testAPi会返回我们send()方法传递的字符串。

小提示：MacOS系统在服务进行中，可以使用control+c来释放端口的监听，如果不小心使用control+z或者关闭了终端，会导致所监听端口的无法释放，下次如果再次启动node服务，会报Port 3000 is already in use的错误，可以使用如下方法来进行所监听端口的释放：

首先使用如下命令查看所有监听某个端口的服务，例如3000端口：

```
sudo lsof -i:3000
```

之后终端会将服务名与进行id告诉我们，如下：

```
COMMAND PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
node    829  vip   13u  IPv6 0x9c3536500e84e203      0t0  TCP *:hbci (LISTEN)
```

使用如下命令来杀死对应进程即可：

```
 sudo kill -9 829
```

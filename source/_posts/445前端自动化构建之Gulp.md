---
title: 前端自动化构建之Gulp
date: 2022-06-18
categories: 前后端
tags: []
---
# 前端自动化构建之Gulp

本篇文章的核心是介绍一款强大的任务流工具Gulp，之所以题目叫做“前端自动化构建之Gulp”，是因为Gulp本身是使用JS编写的运行在Node环境的一个npm包，并且大部分开发者也都使用它来作为前端项目的自动化构建工具。不过，从本质上说，Gulp并非只能做前端自动化构建，后端构建发布自动化，脚本工具集自动化，测试流程自动化等都可以使用Gulp。所以，无论你是哪个技术栈的开发者，学习使用Gulp都会对你大有裨益。

这边文章能够带给你：

-   Gulp工具的安装
-   Gulp工作原理
-   创建自动化流程中的独立任务
-   任务的串行与并行
-   使用管道处理数据流
-   监听文件变更
-   简易前端自动化构建DEMO流程

本篇文章虽然无法将Gulp所有的细节都介绍到，但总体上来说挑选了其中最核心的部分，也是最实用的部分进行了介绍，并且通过一个前端DEMO的示例，演示使用Gulp进行\[环境初始化->代码合并->代码压缩->启动服务器->监听文件->热更新浏览器\]的完整自动化流程。

## 一.Gulp基础

好像任何技术栈的开发者，都会遇到耗费时间且让人痛苦不堪的流程问题。除了基础的研发，测试，部署流程外，往往中间还需要根据需求执行额外的任务，例如代码检查，代码压缩，代码混淆等。这些流程虽然与研发无关，但是却是产品上线的必备过程，繁琐的流程不仅会消耗开发者额外的精力，对于团队来说，也很难控制所有开发者所执行的流程都一致且完整，例如某些成员可能由于疏忽大意而忘记做代码压缩就直接将项目发布上线了。因此，将流程自动化不仅可以解放人力，更是项目稳定性的保障。Gulp就是为解决这样问题而发现的一款简单，高效且生态完整的自动化工具。

### 1.安装Gulp

Gulp是运行在Node环境中的，因此你需要先安装Node.js，这里不再赘述。

Gulp工具分两部分构成，全局Gulp CLI和本地Gulp。关于他们之间的关系，后面我们再详细介绍。首先先来安装全局的Gulp CLI工具，在终端执行如下指令：

```
sudo npm install -g gulp-cli      
```

安装完成后，要检查是否安装成功，可以键入如下指令：

```
gulp -v
```

此时，你应该可以看到如下的输出：

```
CLI version: 2.3.0
Local version: Unknown
```

其中，第一行表示我们安装的全局的Gulp CLI工具的版本号，第二行表示本地Gulp版本号，本地Gulp是安装在具体的项目目录中的，此时我们尚未安装。下面我们可以创建一个项目目录，例如创建名为gulp-demo文件夹，在目录下使用下面的指令来初始化Node模块：

```
npm init
```

在当前目录下使用如下指令来安装本地Gulp：

```
npm install --save-dev gulp 
```

安装完成后，再次执行gulp -v，可以看到本地Gulp的版本，如下：

```
CLI version: 2.3.0
Local version: 4.0.2
```

需要注意，你在测试时，要使用4.x及以上的本地Gulp版本。

### 2.Gulp的工作原理

前面，我们安装了Gulp CLI和本地Gulp，现在是时候来介绍下Gulp的工作原理了。在使用Gulp时，我们需要编写一个名为Gulpfile.js的文件，这个文件是Gulp工作的核心，其中会定义各种任务及任务流。Gulp CLI工具是启动Gulp的入口，其通过指令来调用项目中的本地Gulp，本地Gulp会调用Gulpfile.js文件，加载和执行定义好的任务，在Gulpfile.js文件中，通常又需要调用本地Gulp模块中提供的API方法，以及本地Gulp插件或自定义函数的功能从而实现任务。工作原理如下图所示：

![](https://oscimg.oschina.net/oscnet/up-2856d5e01870a50854315c2d48a39ef8322.png)

### 3.编写Gulpfile.js文件

在学习编程语言时，第一个程序通常都是HelloWorld，Gulp的学习也不例外，我们先来测试下Gulp的HelloWorld程序。在工程目录下新建一个命名为Gulpfile.js的文件，在其中编写如下代码：

```javascript
// 引入本地Gulp模块
var gulp = require('gulp');

// 调用task方法来定义任务
gulp.task('gulp', function(done){
  console.log("Hello Gulp!");
  done();
});

```

其中，task函数用来定义任务，其第1个参数为任务名，第2个参数为要执行的任务方法，这里我们传入了一个自定义的函数，此函数中会传入一个回调done参数，当任务逻辑代码结束后，要调用此回调通知Gulp此任务已经结束，否则会阻塞Gulp后续任务的执行。

在当前目录下，执行如下终端指令：

```
gulp gulp
```

此指令的作用是让全局Gulp CLI工具调用本地Gulp来执行Gulpfile.js中定义的名为gulp的任务。执行后，终端输出如下：

```
[09:37:18] Using gulpfile ~/Desktop/gulp-demo/gulpfile.js
[09:37:18] Starting 'gulp'...
Hello Gulp!
[09:37:18] Finished 'gulp' after 2.55 ms
```

可以看到，正确输出的Hello Gulp!，并且执行任务的开始时间和结束时间都有记录，任务耗时也有记录。

## 二.Gulp任务链与数据流

我们已经知道如何通过Gulp来执行独立的任务，但更多实际场景中，自动化的流程都不是简单独立的任务可以完成的，我们需要将任务组成任务链。例如我们模拟这几个任务：

```javascript
gulp.task('clean', function(done){
  console.log("清理构建目录");
  done();
});

gulp.task('copy', function(done){
  console.log("复制模板文件");
  done();
});

gulp.task('build', function(done){
  console.log("编译代码");
  done();
});

gulp.task('server', function(done){
  console.log("启动开发服务器");
  done();
});

gulp.task('watch', function(done){
  console.log("监听文件变化持续热更新");
  done();
});
```

上面5个任务模拟了前端自动化构建的基本流程，这些任务间有些是有依赖关系的，比如清理构建目录任务来在复制模板文件和编译代码之前，不然可能刚编译好的代码就又被清掉了，启动开发服务器则要在复制模板文件和编译代码之后，但是复制模板文件和编译代码这两个任务是可以并行进行的。使用Gulp任务链，可以非常方便的实现任务间依赖处理。

### 1.series和parallel

series意为串行，parallel意为并行。通过Gulp中的这两个API，可以方便的将任务组合成串行链或并行链，任务链也可以作为一个独立的任务在组合进其他的任务链，这样就是Gulp任务的组合变得非常灵活。按照前面描述的需求添加一个任务链如下：

```javascript
gulp.task('dev', gulp.series(
  'clean',
  gulp.parallel('copy', 'build'),
  'server',
  'watch'
));
```

在终端中键入gulp dev，输出情况如下：

```
[10:38:08] Using gulpfile ~/Desktop/gulp-demo/gulpfile.js
[10:38:08] Starting 'dev'...
[10:38:08] Starting 'clean'...
清理构建目录
[10:38:08] Finished 'clean' after 915 μs
[10:38:08] Starting 'copy'...
[10:38:08] Starting 'build'...
复制模板文件
[10:38:08] Finished 'copy' after 349 μs
编译代码
[10:38:08] Finished 'build' after 423 μs
[10:38:08] Starting 'server'...
启动开发服务器
[10:38:08] Finished 'server' after 180 μs
[10:38:08] Starting 'watch'...
监听文件变化持续热更新
[10:38:08] Finished 'watch' after 153 μs
[10:38:08] Finished 'dev' after 5.57 ms
```

从输出可以看到，任务间的串行和并行关系已经可以满足我们的需求，使用Gulp处理任务依赖就是这么简单。

我们再回过头来看一下series和parallel两个方法，其内可以传入不限个数个参数，参数可以是字符串，可以是函数，也可以是其他的任务链组合。当参数为字符串时，其会被当成任务名来调用指定任务，当参数为函数时，会执行此函数，当参数为任务链组合时，会执行对应的任务链。当参数为函数时要特别注意，必须回调通知Gulp任务完成，否则会阻塞任务链。如下：

```javascript
gulp.task('dev', gulp.series(
  'clean',
  function(done){
    console.log('custom method');
    done();
  },
  gulp.parallel('copy', 'build'),
  'server',
  'watch'
));
```

执行后，终端输出如下：

```
[10:46:31] Using gulpfile ~/Desktop/gulp-demo/gulpfile.js
[10:46:31] Starting 'dev'...
[10:46:31] Starting 'clean'...
清理构建目录
[10:46:31] Finished 'clean' after 738 μs
[10:46:31] Starting '<anonymous>'...
custom method
[10:46:31] Finished '<anonymous>' after 238 μs
[10:46:31] Starting 'copy'...
[10:46:31] Starting 'build'...
复制模板文件
[10:46:31] Finished 'copy' after 520 μs
编译代码
[10:46:31] Finished 'build' after 490 μs
[10:46:31] Starting 'server'...
启动开发服务器
[10:46:31] Finished 'server' after 168 μs
[10:46:31] Starting 'watch'...
监听文件变化持续热更新
[10:46:31] Finished 'watch' after 117 μs
[10:46:31] Finished 'dev' after 7.16 ms
```

你或许发现了，执行到自定义的函数时，终端显示的任务名是anonymous，表示匿名的，更好的方式是我们使用具名函数，这样方便对任务执行状态进行回溯，如下：

```javascript
gulp.task('dev', gulp.series(
  'clean',
  function customMethod(done){
    console.log('custom method');
    done();
  },
  gulp.parallel('copy', 'build'),
  'server',
  'watch'
));
```

### 2.关于Gulp数据流

从宏观上了解了Gulp任务的串行与并行，现在我们再来关注下更具体的问题。本地Gulp本身提供了许多强大的API，像前面用到的task，series，parallel都是API其中之一。自动化构建，本质上是对文件的扫描和处理，因此Gulp中也封装了与文件操作相关的API。

首先，读取文件流和输出文件流的两个API分别是src和dest。

src方法用来创建一个文件数据流，可以从文件系统读取文件数据到虚拟文件对象，虚拟文件对象是Gulp中的一个概念，其会将文件的名称，信息，数据等抽象成一个虚拟的对象，之后文件的操作都在内存中进行，速度很快，直到处理完成后在输出到指定文件。例如开发时，我们的模板代码通常在src目录下，编译打包后会将处理后的文件放入dest目录下，这就涉及到文件的拷贝处理。

在工程目录下创建一个名为src的子文件夹，在其中创建一个简单的HTML文件，修改下copy任务如下：

```javascript
gulp.task('copy', function(done){
  return gulp.src('src/*.html').
  pipe(gulp.dest('dest'));
});

```

再次执行整个任务流，可以看到项目根目录中自动生成了一个dest文件夹，并且已经将HTML文件拷贝了进去。在这个copy任务的实现函数中，我们并没有调用done回调，这是因为gulp的API已经实现了Gulp任务接口规范，直接返回结果即可。

src方法读取文件数据流时，可以指定具体的文件或通配符，如上代码所示，表示将src文件夹下所有的HTML文件选中，创建成数据流，数据流提供的主要API是pipe方法，这个方法用来连接各个处理数据流的节点，可以将它理解为一个管道，输入从一端流入，处理后从另一端流出，如果需要继续进行其他处理，还可以连接其他管道。这种责任链的设计模式，可以非常灵活的连接各个处理插件。以前端代码合并压缩处理为例，流程如下：

_**文件系统---->src\[虚拟文件流\]--->文件合并模块\[合并后的虚拟文件流\]--->代码压缩模块\[压缩有的虚拟文件流\]--->dest\[回写到文件系统\]--->文件系统**_

## 三.实践：简易前端自动化构建流程

前面我们逻辑上模拟了前端自动化构建过程，只是在实现上，都是用的log进行模拟，这并不十分有趣。使其，使用Gulp搭建一个简易的前端构建自动化流程并不复杂，几分钟就可以搞定。

### 1.安装几个Gulp插件

Gulp有着非常丰富的生态，本质上，任意npm模块我们都可以直接使用，除此之外，还有4000多个专门为Gulp设计的插件，可以方便的支持各种文件处理，代码检查等需求。插件地址如下：

[https://gulpjs.com/plugins/](https://gulpjs.com/plugins/)

本次实践，需要使用到2个Gulp插件和2个npm模块，分别进行文件合并，代码压缩，文件删除和浏览器测试热更新。在项目目录下使用如下指令来安装这些模块：

```
npm install --save-dev gulp-concat 
npm install --save-dev gulp-uglify  
npm install --save-dev del   
npm install --save-dev browser-sync
```

在Gulpfile.js中引入这些模块：

```javascript
var gulp = require('gulp');
var concat = require ('gulp-concat');
var uglify = require('gulp-uglify');
var del = require('del')
var bSync = require('browser-sync').create();
```

### 2.实现自动化构建流程中的几个任务

首先我们先来实现clean任务，修改代码如下：

```javascript
gulp.task('clean', function(done){
  return del(['dest'])
});

```

上面代码的作用是将dest文件夹删除，每次初始构建，我们都将旧的目录删掉，以免缓存的文件对最终的软件包造成影响。

copy任务的实现方法不变，直接将HTML文件模板拷贝到指定的构建目录即可：

```javascript
gulp.task('copy', function(done){
  return gulp.src('src/*.html').
  pipe(gulp.dest('dest'));
});

```

编译的过程主要涉及到CSS相关的SASS或LESS编译，或TS，CoffeeScript的编译等等，为了简单起见，我们只做JS文件的合并和压缩，如下：

```javascript
gulp.task('build', function(done){
  return gulp.src('src/*.js')
  .pipe(concat('main.js'))
  .pipe(uglify())
  .pipe(gulp.dest('dest'))
});

```

上面代码的作用是将src文件夹下所有的JS文件合并到一个名为main.js的文件中，并进行代码压缩，最后输出到dest文件夹。

实现server任务如下：

```javascript
gulp.task('server', function(done){
  bSync.init({
    server:'./dest',
  });
  done();
});
```

上面代码会启动本地服务器，并引导chrome项目页面。

对于监听本地文件的变化，Gulp中自带对应的API，实现watch任务如下：

```javascript
gulp.task('watch', function(done){
  gulp.watch(['src/*.js'],gulp.parallel('build'));
  gulp.watch(['src/*.html'],gulp.parallel('copy'));
  gulp.watch(['dest/**/*'],function(done){
    bSync.reload('index.html');
    done()
  });
  done();
});
```

这里需要注意，由于我们对JS文件的编译和HTML模板的拷贝任务是分开的，因此监听也可以分开，当发现JS文件发生了变化时，只进行编译任务即可，当发现HTML文件变化时，只进行模板拷贝任务即可，当构建文件夹有变更时，重新刷新浏览器，从而实现开发过程中的测试热更新。

最后，我们可以写一些测试文件了，在src文件夹下新建两个js文件和一个html文件，代码分别如下：

file1.js文件中代码：

```javascript
function mlog(msg) {
	console.log(Date(),': 自定义输出 -',msg);
}
mlog("hello");
```

file2.js文件中代码：

```javascript
var a = 1;
var b = 2;
var c = a + b;
console.log(c);
```

index.html文件中代码：

```html
<html>
<header>
	<script type="text/javascript" src="./main.js"></script>
</header>
<body>
	<h1>标题11</h1>
</body>
	
</html>
```

使用gulp dev来开启自动化构建流程，你可以打开chome的控制台，查看控制台的输出，尝试修改下src文件夹中的HTML文件和JS文件，保存后可以看到对应的页面也会发生更新。

到此，我们已经实现了一个简单的前端开发自动化构建流程，Gulp不负你所望吧！

温馨提示：在子进程结束之前，Gulp进程是不会自动结束的，因此其会一直监听文件的变更刷新页面，要关闭开发服务器，只需control+c即可。

## 四.结语

最后，虽然大多数情况下Gulp会用在前端自动化构建中，但是这并不是Gulp唯一的用途，任何开发工作流都可以使用Gulp来构建，npm包的丰富极大的扩展了Gulp的应用场景。希望本篇文章可以帮你打开技术视野，能够探索出Gulp的更多应用场景，帮助你简化工作流程。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> —— 珲少 QQ：316045346
> 
> 同时，如果本篇文章让你觉得有用，欢迎分享给更多朋友，请标明出处。
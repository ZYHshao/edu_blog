---
title: ReactNative开发环境的搭建与开发前准备
date: 2016-12-23
categories: ReactNative
tags: []
---
## ReactNative开发环境的搭建与开发前准备

### 一、准备工作

    在ReactNative中文网上有详细的开发文档与教程，首先，想要系统了解ReactNative的朋友可以在如下网站中获取详细信息：

[http://reactnative.cn/](http://reactnative.cn/)。

本篇博客记录搭建ReactNative开发环境中的一些问题与注意点，也介绍在MacOS系统上搭建ReactNative开发环境的全过程与一些小经验技巧。

    ReactNative最大的魅力在于其编写的代码可以跨平台应用，因此我极力推荐在MacOS上进行ReactNative应用的开发，由于Xcode开发工具只能运行与MacOS系统，在Windows或Linux系统上将无法进行iOS平台的调试，因此本篇博客也将基于MacOS系统进行演示。    

    在ReactNative环境之前，开发者需要先安装一些小工具，首先需要安卓Homebrew工具，Homebrew工具是Mac系统的包管理器，在终端运行如下命令进行安装：

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

安装完成后，可以使用brew -v命令检查Homebrew版本，正常输出版本号说明安装成功：

![](https://static.oschina.net/uploads/space/2016/1223/173800_dW98_2340880.png)

在安装过程中，如果遇到权限问题，需要使用如下命令进行修复：

```
sudo chown -R `whoami` /usr/local
```

    安装完成Homebrew后，需要使用其来进行Node环境的安装，使用如下命令：

安装完成后，同样可以使用node -v命令来检查是否安装成功：

![](https://static.oschina.net/uploads/space/2016/1223/174507_5sLk_2340880.png)

虽然Yarn是facebook提供的代替npm的包管理工具，但我个人更倾向使用npm来进行ReactNative包的安装，在使用之前，可以通过替换源镜像的方式来进行npm的加速(在无法科学上网的前提下):

```
npm config set registry https://registry.npm.taobao.org --global
npm config set disturl https://npm.taobao.org/dist --global
```

之后使用npm config get registry来检查源镜像的替换是否成功：

![](https://static.oschina.net/uploads/space/2016/1223/175056_I4nM_2340880.png)

之后我们需要安装ReactNative的命令行工具react-natice-cli。这个工具用来初始化ReactNative项目，命令如下：

```
npm install -g react-native-cli
```

使用react-native -v命令来检查是否安装成功：

![](https://static.oschina.net/uploads/space/2016/1223/175805_OAC8_2340880.png)

到此，ReactNative的基础环境已经搭建完成了，下面需要配置iOS与Android开发工具。

### 二、Xcode与Android Studio配置

    Xcode基本无需进行额外的配置，你只需要从AppStore上下载下来最新版本的Xcode开发工具安装完成即可，Xcode会打包安装命令行工具，git工具和所需要的模拟器。

    对于Android开发环境，首先你需要保证你的Android Studio工具版本在2.0以上并且Java版本要在1.8以上，javac -version命令可以查看当前的JDK版本，如果低于1.8，可以到官网下载：

![](https://static.oschina.net/uploads/space/2016/1223/193831_PkVy_2340880.png)

官网：[http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)。

下载安装完成Android Studio后，首先需要对SDK进行相应配置，打开Android Studio，打开欢迎界面的SDK Manager，如下图：

![](https://static.oschina.net/uploads/space/2016/1223/194348_ydYM_2340880.png)

选中其中的SDK Platforms，勾选Show Package Details。我选择Android6.0相关的SDK进行安装，Android SDK Platform 23,Intel x86 Atom_64 System Image为必选， 如下图所示：

![](https://static.oschina.net/uploads/space/2016/1223/194821_D56v_2340880.png)

除此之外，还需要安装SDK Tools，必须安装其中的23.0.1版本，切记，这是必须！如下图：

![](https://static.oschina.net/uploads/space/2016/1223/195929_sBey_2340880.png)

之后随便使用Android Studio创建一个项目，打开其中的AVD Manager，如下：

![](https://static.oschina.net/uploads/space/2016/1223/200348_qGF1_2340880.png)

AVD Manager用来管理Android模拟器，如果以后模拟器，可以点击运行按钮开运行模拟器，如果没有，可以创建一个模拟器，如下图：

![](https://static.oschina.net/uploads/space/2016/1223/200603_Ak4E_2340880.png)

做完上述步骤后，切记要配置Android SDK的环境变量，在终端使用如下命令进行环境变量文件的编辑：

```
sudo vi ~/.bash_profile
```

在文件中添加如下路径：

```
export ANDROID_HOME=~/Library/Android/sdk
```

之后在终端执行如下命令来使设置生效：    

```
source ~/.bash_profile
```

可以使用echo $ANDROID_HOME命令来检查环境变量的配置是否正确，如下：

![](https://static.oschina.net/uploads/space/2016/1223/201747_IcXF_2340880.png)

### 三、运行第一个项目HelloWorld

    如果上面的环境配置和开发工具的配置都已顺利完成，那么你离第一个ReactNative项目已经不远了，下面我们来通过ReactNative创建HelloWorld项目。

    在终端运行react-native init HelloWorld命令来创建ReactNative项目，这个命令是一个一站式集成命令，其会创建项目并且将所有依赖包都安装完成。命令成功执行后，进入到项目根目录中，如下：

![](https://static.oschina.net/uploads/space/2016/1223/202836_dlGy_2340880.png)

使用react-native run-ios或者react-native run-android来进行iOS项目或者Android项目的运行，如果你看到如下图所示的界面，恭喜你，你的ReactNative项目已经可以跑起来了(需要注意：运行安卓项目的时候，安卓模拟器必须先启动)：

![](https://static.oschina.net/uploads/space/2016/1223/203514_yEIc_2340880.png)

需要注意，运行iOS项目时，会默认启动Xcode的默认模拟器，如果要启动特定的模拟器，可以使用如下命令：

```
react-native run-ios --simulator "iPhone SE"
```

xcrun simctl list devices命令可以打印出所有可用的iOS模拟器，示例如下：

![](https://static.oschina.net/uploads/space/2016/1223/203927_BZuo_2340880.png)

观察HelloWorld项目结构，其目录如下图：

![](https://static.oschina.net/uploads/space/2016/1223/203621_emnu_2340880.png)

其中node_modules为node依赖包的目录，andorid文件夹为安卓项目目录，ios文件夹为iOS项目目录。index.android.js与index.ios.js两个文件是最为重要的两个文件，这两个文件是iOS项目与Android项目的入口文件，打开index.ios.js文件，将其中的代码修改如下：

```javascript
import React, { Component } from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  View
} from 'react-native';

export default class HelloWorld extends Component {
  render() {
    return (
      <Text style={{
        flex:1,
        top:100,
        left:100,
        fontSize:30
      }}>HelloWorld</Text>  
    );
  }
}
AppRegistry.registerComponent('HelloWorld', () => HelloWorld);
```

上面的代码就是一个最简单的项目HelloWorld，在iOS模拟器中使用command+R来进行界面的刷新，效果如下：

![](https://static.oschina.net/uploads/space/2016/1223/204637_riIu_2340880.png)

在安卓模拟器中双击R键来进行界面的刷新。

提示：如果在iOS模拟器中使用command+R无效，需要将模拟器的Connect Hardware Keyboard进行勾选，如下：

![](https://static.oschina.net/uploads/space/2016/1223/204906_CwDU_2340880.png)

### 四、ReactNative开发工具的选择

    facebook提供了一个叫做Nuclide的工具专门开发ReactNative应用，其实一个基于atom的集成开发环境，但是我个人更喜欢使用SublimeText来进行ReactNative应用的开发。通过安装相应的插件，SublimeText来编写ReactNative应用将十分畅快。

    首先下载SublineText编辑工具，可以在官网进行下载：

[http://www.sublimetext.com/](http://www.sublimetext.com/)。

在对SublimeText进行插件安装前，需要先为其安装包管理工具PackageControl。在SublimeText工具的导航中选择View下的Show Console来打开命令行，如下：

![](https://static.oschina.net/uploads/space/2016/1223/210147_sMfW_2340880.png)

在命令行中输入如下代码进行，敲击回车进行安装：

SublimeText2：

```python
import urllib2,os; pf='Package Control.sublime-package'; ipp = sublime.installed_packages_path(); os.makedirs( ipp ) if not os.path.exists(ipp) else None; urllib2.install_opener( urllib2.build_opener( urllib2.ProxyHandler( ))); open( os.path.join( ipp, pf), 'wb' ).write( urllib2.urlopen( 'http://sublime.wbond.net/' +pf.replace( ' ','%20' )).read()); print( 'Please restart Sublime Text to finish installation')
```

SublimeText3:

```python
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
```

由于网络等多种原因，上面使用代码安装PackageControl的方法很大可能会失败，我们也可以从官网下载安装包，将其放在SublimeText的包安装目录中，重启SublimeText来进行安装，点击SublimeText的Preferences->Browse Packages选项可以浏览SublimeText的包目录：

![](https://static.oschina.net/uploads/space/2016/1223/212040_NKVY_2340880.png)

将下载好的PackageControl安装包放入Installed Package目录中重启即可，如果没有这个目录，可以手动创建：

![](https://static.oschina.net/uploads/space/2016/1223/212244_HUxq_2340880.png)

PackageControl的官方下载地址为：[http://sublime.wbond.net/Package%20Control.sublime-package](http://sublime.wbond.net/Package%20Control.sublime-package)。

温馨提示：PackageControl的官方下载地址访问起来也有一定难度，我将这个安装包放在了我的github上一份，如果需要，可以从下面的地址下载：

[http://zyhshao.github.io/file/Package%20Control.sublime-package](http://zyhshao.github.io/file/Package%20Control.sublime-package)。

    安装完成PackageControl工具后，即可使用其来进行SublimeText插件的安装。

在SublimeText中选择Preferences->PackageControl即可调出PackageControl命令面板，如下：

![](https://static.oschina.net/uploads/space/2016/1223/215615_ilOh_2340880.png)

PackageControl中提供了许多实用的方法，例如install Package用来安装插件，List Packages用来查看已经安装过的插件，Remove Package用来删除一个已经安装的插件等，如下：

![](https://static.oschina.net/uploads/space/2016/1223/215752_xDBF_2340880.png)

点击Install Package进入SublimeText插件的搜索界面，搜索到所需要安装的插件安装即可，如下：

![](https://static.oschina.net/uploads/space/2016/1223/220342_WuDF_2340880.png)

温馨提示：在使用PackageContrl的Install Package命令时，很有可能会出现超时问题，原因是PackageControl需要拉取一个channels文件列表，而这个文件在国内往往难以访问到，我也在我的github上存放了一份备份，需要将PackageControl的channels拉取路径做下修改，选择SublimeText的Preferences->PackageSettings->PackageControl->Settings-Default选项，如下：

![](https://static.oschina.net/uploads/space/2016/1223/220952_HcfD_2340880.png)

将其中的channels参数修改如下即可：

```
    "channels": [
        "http://zyhshao.github.io/file/channel_v3.json"
    ],
```

### 五、推荐几个编写ReactNative好用的SublimeText插件

#### 插件一：Emmet

    Emmet插件是前端神器，其提供了js的自动补全功能，使用PackageControl搜索安装emmet插件后，打开SublimeText中的Preferences->Package Settings->Emmet->Key Bindings-User，将其文件修改为如下：

```json
[{
    "keys": ["tab"],
    "command": "expand_abbreviation_by_tab",

    // put comma-separated syntax selectors for which 
    // you want to expandEmmet abbreviations into "operand" key 
    // instead of SCOPE_SELECTOR.
    // Examples: source.js, text.html - source
    "context": [{
            "operand": "source.js",
            "operator": "equal",
            "match_all": true,
            "key": "selector"
        },

        // run only if there's no selected text
        {
            "match_all": true,
            "key": "selection_empty"
        },

        // don't work if there are active tabstops
        {
            "operator": "equal",
            "operand": false,
            "match_all": true,
            "key": "has_next_field"
        },

        // don't work if completion popup is visible and you
        // want to insert completion with Tab. If you want to
        // expand Emmet with Tab even if popup is visible -- 
        // remove this section
        {
            "operand": false,
            "operator": "equal",
            "match_all": true,
            "key": "auto_complete_visible"
        }, {
            "match_all": true,
            "key": "is_abbreviation"
        }
    ]
}]
```

#### 插件二：jsformat

    jsformat插件可以进行js代码的格式化，使用PackageControl安装完成后，选中js代码，使用control+option+f即可进行代码的格式化操作。

#### 插件三：ReactJS

    ReactJS插件支持对React代码进行高亮，并且支持快捷创建函数，原型等操作，熟练使用可以大大提高开发效率，其用法github如下：

[https://github.com/facebookarchive/sublime-react](https://github.com/facebookarchive/sublime-react)。

#### 插件四：Terminal

    Terminal也是SublimeText开发ReactNative应用神器，安装好后，使用command+shift+T可以直接在当前目录打开终端。

#### 插件五：react-native-snippets

    react-native-snippets可以快速的创建ReactNative类等代码块，用法github如下：

[https://github.com/Shrugs/react-native-snippets](https://github.com/Shrugs/react-native-snippets)。

最后，再推荐一款SublimeText皮肤Hero，我个人非常喜欢这款皮肤，其安装与配置方法在如下github上有详细介绍：

[https://github.com/nickbalestra/hero](https://github.com/nickbalestra/hero)。

效果如下：

![](https://static.oschina.net/uploads/space/2016/1223/225913_Jz7P_2340880.png)

有了上面的这些工具，我们的SublimeText就编程了一款强大ReactNative开发IDE，尽情享受畅快编码的感觉吧！

    到此为止，本篇博客将所有开发ReactNative应用的准备工作已经介绍完毕，后面的博客将记录手把手开发一款ReactNative应用程序的学习过程：ReactNative简易汇率换算器！期待与您的共同交流与进步！

> ReactNative兴趣群：605794212，欢迎交流学习。

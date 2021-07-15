---
title: 使用Jenkins配置iOS自动化构建工具
date: 2018-06-28
categories: 小码工具
tags: []
---
## 使用Jenkins配置iOS自动化构建工具

      关于iOS自动化构建其实并不复杂，通过一些简单的Git与Xcode指令，加上UI，我们自己也可以动手编写一款自动化构建工具。这在之前的博客中也有涉及，有兴趣的朋友可以在如下地址找到这篇博客：

自己动手设计一款iOS自动构建发布工具：[https://my.oschina.net/u/2340880/blog/1486246](https://my.oschina.net/u/2340880/blog/1486246)

       本篇博客主要记录使用Jenkins搭建iOS自动化构建项目的过程，关于Jenkins的更多自动化脚本的应用，有机会后面再出专门的博客介绍。

### 一、Jenkins的安装与启动

    Jenkins的安装非常方面，在如下官网可以直接下载Jenkins的安装包，其中有支持各个平台的安装包，选择自己所需要的进行下载安装即可。

[https://jenkins.io/](https://jenkins.io/)

   安装完成后，Jenkins会自动启动运行，在当前电脑的8080端口开启一个Web应用服务，如果是第一次安装启动，我们需要配置一个账户作为初始用户。

   对于在Mac上Jenkins的启动，有两种方式：

方式一：直接运行Java归档文件启动Jenkins

    如果是Mac电脑，Jenkins安装完成后，在Applications目录下会多出一个Jenkins文件夹，这个文件夹中包含一个jenkins.war的文件，如下图所示：

![](https://oscimg.oschina.net/oscnet/7df1b73abe670d73753ab25c0f27b93017e.jpg)

使用如下命令来启动Jenkins：

java -jar /Applications/Jenkins/jenkins.war

使用这种方式启动的Jenkins，要关闭服务需要找到Jenkins服务对应的PID，在终端输入如下命令:

```
ps
```

在终端输出的信息中，可以看到Jenkins服务所对应的PID号，如下图：

![](https://oscimg.oschina.net/oscnet/64b6241c4362e48641327fc377b92efad09.jpg)

终端使用如下命令将此服务杀死即可：

```
kill -9 PID号
```

方式二：使用Mac的启动进行控制器启动Jenkins

    如果成功安装了Jenkins，在Mac电脑磁盘的资源库中的LaunchDaemons文件夹下可以找到Jenkins的启动配置文件，如下图：

![](https://oscimg.oschina.net/oscnet/b0a8f8c9679bb19d89e1a418b328742aea4.jpg)

在命令行中执行如下命令即可启动Jenkins服务：

```
sudo launchctl load /Library/LaunchDaemons/org.jenkins-ci.plist
```

使用如下命令关闭Jenkins服务：

```
sudo launchctl unload /Library/LaunchDaemons/org.jenkins-ci.plist
```

### 二、创建持续集成项目

    在Jenkins的主页选择新建一个项目，创建选项中，我们输入项目名称，并选择构建一个自由风格的软件，如下图：

![](https://oscimg.oschina.net/oscnet/ad5bb236ddc9a9059aa85326b461f72b9f7.jpg)

下一步将进入到项目配置界面，首先需要设置下通用的配置，如下图：

![](https://oscimg.oschina.net/oscnet/98f4a0d4eb538d88d4bc139c162e1b6387c.jpg)

其中，描述部分可以填写项目的相关介绍，丢弃旧的构建设置构建记录保存的天数和最多保持多少个构建记录等。

    源码管理的配置是比较重要的一步，其用来设置构建项目从哪里拉取项目的源代码以及进行源码更新的操作。如下图：

![](https://oscimg.oschina.net/oscnet/96b9fffbfff489bd48b3271c0af353d3f3f.jpg)

如果使用的是Git仓库，如上图所示，需要配置项目的路径，账户以及要进行构建的分支。账户的主要用途是使得Jenkins有权限拉取项目的代码，如果之前没有添加过，可以点击右侧的Add按钮进行添加，如下图：

![](https://oscimg.oschina.net/oscnet/40737a8f93954fe7bd116ab73bc159b3a7f.jpg)

可以选择配置用户名加密码的方式添加账户，也可以使用SSH公钥的方式。

    下一步我们需要配置构建的触发器，构建触发器有多种形式，比如定时触发构建，远程触发，代码提交后触发等等，如下图：

![](https://oscimg.oschina.net/oscnet/4b565fbe29b5f22fdc5a2e4eb1cfeb54ba8.jpg)

其中远程触发是指我们可以通过远程访问Jenkins服务器地址加上令牌参数来触发构建。

之后再构建一栏中选择增加构建步骤->执行Shell，添加如下Shell脚本：

```
export LANG=en_US.UTF-8
export LANGUAGE=en_US.UTF-8
export LC_ALL=en_US.UTF-8
/usr/local/bin/pod install
xcodebuild -archivePath "/Users/Shared/Jenkins/Home/workspace/Jenkins项目名称/你的项目名.xcarchive" -workspace 你的项目名.xcworkspace -sdk iphoneos -scheme "你的项目名" -configuration "Release" archive

xcodebuild -exportArchive -archivePath "/Users/Shared/Jenkins/Home/workspace/Jenkins项目名称/你的项目名.xcarchive" -exportPath "/Users/Shared/Jenkins/Home/workspace/Jenkins项目名称/buildIPA" -exportOptionsPlist '/Users/Shared/Jenkins/Home/workspace/Tictalk-iOS/ExportOptions.plist' -allowProvisioningUpdates
curl -F "file=@/Users/Shared/Jenkins/Home/workspace/Jenkins项目名称/buildIPA/你的项目名.ipa" -F "uKey=蒲公英userKey" -F "_api_key=蒲公英apikey" https://qiniu-storage.pgyer.com/apiv1/app/upload
```

上面脚本中，xcodebuild -archivePath 命令用来编译项目，如果你的项目没有使用workspace，需要将命令中的workspace修改成project，configuration参数用来配置编辑的方式，Release为发布环境。xcodebuild -exportArchive 命令用来到处API包，需要额外注意，提前我们需要在/Users/Shared/Jenkins/Home/workspace/Jenkins项目名称/你的项目名这个目录下添加一个ExportOptions.plist文件，新Xcode如果不配置这个文件是无法打包成功的。curl -F 命令是用来将打包好的IPA包自动上传到蒲公英分发平台。

    ExportOptions.plist文件编写格式如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>compileBitcode</key>
    <false/>
    <key>method</key>
    <string>ad-hoc(IPA包模式)</string>
    <key>provisioningProfiles</key>
    <dict>(下面设置bundleID对应的provisioningProfiles文件名)
        <key>com.***</key>
        <string>AD_HOC</string>
        <key>com.***.TKNotificationCentent</key>
        <string>Content</string>
        <key>com.***.TKNotificationService</key>
        <string>Service</string>
    </dict>(下面配置证书)
    <key>signingCertificate</key>
    <string>iPhone Distribution</string>
    <key>signingStyle</key>
    <string>manual</string>
    <key>stripSwiftSymbols</key>
    <true/>(下面配置teamID)
    <key>teamID</key>
    <string>KJYHPT****</string>
    <key>thinning</key>
    <string>&lt;none&gt;</string>
</dict>
</plist>

```

### 三、构建可能出错的地方

    配置完了上面的脚本，你可以尝试点击立即构建按钮进行构建，当然构建过程中极有可能会出错，你可以根据log输出检查下是否是因为下面的问题。

#### 1.git相关命令出错

    可能是Jenkins找不到git所在位置，在Jenkins的系统设置中选择全局工具配置，配置git路径如下图所示：

![](https://oscimg.oschina.net/oscnet/20ec0ea63d8d7911b843dbfde841df9433d.jpg)

#### 2.pod相关命令出错

   这一步出错的可能性极大，首先你的电脑可以使用pod不代表jenkins用户有使用pod的权限，最好使用jenkins用户登录电脑，进行pod的更新升级，或者直接使用jenkins用户登录，找到我们的项目，手动使用pod进行第三方的安装。

#### 3.xcodebuild相关命令出错

    和git命令出错的问题基本一致，我们需要配置路径。在Jenkins的系统设置中找到系统配置，设置xcode相关工具如下：

![](https://oscimg.oschina.net/oscnet/a6908ca6ff17bd267ce6f060717ab84b83a.jpg)

#### 4.编译过程中证书或配置文件出错

    首先确保你的应用证书放在了钥匙串的系统分类下，如图：

![](https://oscimg.oschina.net/oscnet/c28de6063fdb4f09e1dacec548e3d472eee.jpg)

其次，需要将Provisioning Profiles文件复制到下面的目录下，切记：

/Users/Shared/Jenkins/Library/MobileDevice/Provisioning Profiles

最后，请确认可以使用Xcode手动进行编辑和打包。而且证书和Provisioning Profiles文件一定要正确和匹配。

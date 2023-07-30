---
title: Mac部署AIGC图片生成服务——基于stable-diffusion
date: 2023-07-30
categories: 从机器学习到AI
tags: []
---
# Mac部署AIGC图片生成服务——基于stable-diffusion

AIGC即人工智能内容生成，是目前非常火的一个概念。随着各种大模型的问世，通过AI来生成内容的能已经越来越强大。本文将从应用实践方面进行介绍如何在自己的PC电脑上部署一个强大的AI图片生成服务。

关于AI绘图，我相信你一定不太陌生，现在很多宣传图，插图其实都是有AI根据输入的文本信息来进行生成的。stable-diffusion就是这样的一种图片生成AI系统，更重要的是他是开源的，通过它我们可以快速的部署自己的AI图片生成服务。

首先，我们可以从Github上下载一个开源的基于stable-diffusion的网页端应用程序，地址如下：

[https://github.com/AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui)

此服务的运行依赖Python环境，需要在个人电脑上安装Python3.10.6的版本。

之后需要安装Pytorch，Pytorch是一款基于Python的机器学习库，我们需要根据自己电脑的系统来选择安装命令，Pytorch网址如下：

[https://pytorch.org/get-started/locally/](https://pytorch.org/get-started/locally/)

如下图所示：

![](https://oscimg.oschina.net/oscnet/up-b37ad8577a76f06dd18fe988d0009180ee7.png)

要验证Pytorch是否安装成功也很简单，直接进入Python可交互环境，输入：

```python
import torch
```

如果没有任何报错，则说明已经安装成功，如下图所示：

![](https://oscimg.oschina.net/oscnet/up-527e938e2b625468e0e4e8efe72f2549ef4.png)

现在我们还需要一个训练好的AI模型来指导生成效果，在如下网站中可以下载到很多训练好的模型：

https://civitai.com/

![](https://oscimg.oschina.net/oscnet/up-b38ebf8c1e67ec12bcf35b9ae799a122b24.png)

选择一个感兴趣的模型，下载好的，将模型文件放入工程目录下的models/Stable-diffusion文件夹下面即可。

下面可以尝试运行下此Web项目，在工程目录下执行：

```
./webui.sh 
```

执行完成后，在浏览器打开如下地址：

http://127.0.0.1:7860/

页面如下所示：

![](https://oscimg.oschina.net/oscnet/up-525e46b72b6abb9de0d8c32b8fbc6fa1009.png)

可以看到，界面上左上角的位置可以选择要使用的模型，支持文字生成图片，图片生成图片，并可以进行图片生成的参数调整。下面可以尝试输入一个prompt来进行图片生成，需要注意，prompt分为两部分，上面输入正向词汇，下面输入负向词汇。

![](https://oscimg.oschina.net/oscnet/up-2ec42d2648d31baee8b030cd2673a7d56b8.png)

需要注意，如果你无法访问https://civitai.com/网站，也可以尝试在[https://huggingface.co/](https://huggingface.co/)上面找一些训练好的生成图片的模型进行使用。
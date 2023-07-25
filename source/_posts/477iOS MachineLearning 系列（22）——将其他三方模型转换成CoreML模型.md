---
title: iOS MachineLearning 系列（22）——将其他三方模型转换成CoreML模型
date: 2023-07-25
categories: 从机器学习到AI
tags: []
---
# iOS MachineLearning 系列（22）——将其他三方模型转换成CoreML模型

本篇文章将是本系列文章的最后一篇。本专题将iOS中有关Machine Learning的相关内容做了整体梳理。下面是专题中的其他文章地址，希望如果你有需要，本专题可以帮助到你。

[iOS MachineLearning 系列（1）—— 简介](https://my.oschina.net/u/2340880/blog/8660282)

[iOS MachineLearning 系列（2）—— 静态图像分析之矩形识别](https://my.oschina.net/u/2340880/blog/8671152)

[iOS MachineLearning 系列（3）—— 静态图像分析之区域识别](https://my.oschina.net/u/2340880/blog/8681809)

[iOS MachineLearning 系列（4）—— 静态图像分析之物体识别与分类](https://my.oschina.net/u/2340880/blog/8694846)

[iOS MachineLearning 系列（5）—— 视频中的物体运动跟踪](https://my.oschina.net/u/2340880/blog/8695239)

[iOS MachineLearning 系列（6）—— 视频中的物体轨迹分析](https://my.oschina.net/u/2340880/blog/8695726)

[iOS MachineLearning 系列（7）—— 图片相似度分析](https://my.oschina.net/u/2340880/blog/8695891)

[iOS MachineLearning 系列（8）—— 图片热区分析](https://my.oschina.net/u/2340880/blog/8695939)

[iOS MachineLearning 系列（9）—— 人物蒙版图生成](https://my.oschina.net/u/2340880/blog/8695980)

[iOS MachineLearning 系列（10）—— 自然语言分析之文本拆解](https://my.oschina.net/u/2340880/blog/8703599)

[iOS MachineLearning 系列（11）—— 自然语言识别与文本分析](https://my.oschina.net/u/2340880/blog/8705432)

[iOS MachineLearning 系列（12）—— 自然语言之词句相似性分析](https://my.oschina.net/u/2340880/blog/8709156)

[iOS MachineLearning 系列（13）—— 语音与音频相关的 AI 能力](https://my.oschina.net/u/2340880/blog/8721474)

[iOS MachineLearning 系列（14）—— 使用官方模型进行预测](https://my.oschina.net/u/2340880/blog/8740535)

[iOS MachineLearning 系列（15）—— 可进行个性化更新的 CoreML 模型](https://my.oschina.net/u/2340880/blog/8749962)

[iOS MachineLearning 系列（16）—— 几个常用的图片分类 CoreML 模型](https://my.oschina.net/u/2340880/blog/8761463)

[iOS MachineLearning 系列（17）—— 几个常用的对象识别 CoreML 模型](https://my.oschina.net/u/2340880/blog/8867228)

[iOS MachineLearning 系列（18）—— PoseNet，DeeplabV3 与 FCRN-DepthPrediction 模型](https://my.oschina.net/u/2340880/blog/9038544)

[iOS MachineLearning 系列（19）—— 分析文本中的问题答案](https://my.oschina.net/u/2340880/blog/9268546)

[iOS MachineLearning 系列（20）—— 训练生成 CoreML 模型](https://my.oschina.net/u/2340880/blog/9377371)

[iOS MachineLearning 系列（21）——CoreML 模型的更多训练模板](https://my.oschina.net/u/2340880/blog/10090735)

专题中，从iOS中Machine Learning相关的API开始介绍，后续扩展到如何使用模型进行预测，如何自定义的训练模型。其实CoreML框架只是Machine Learning领域内的一个框架而已，市面上还有许多流行的用来训练模型的框架。如TensorFlow，PyTorch，LibSVM等。在iOS平台中直接使用这些框架训练完成的模型是比较困难的，但是Core ML Tools提供了一些工具可以方便的将这些模型转换成CoreML模型进行使用，大大降低了模型的训练成本。

此工具官网：

[https://coremltools.readme.io/docs](https://coremltools.readme.io/docs)

![截屏2023-07-25 16.40.04.png](https://ucc.alicdn.com/pic/developer-ecology/ww5utkr4n4ptw_7aea1320100b47ecbe9c326f9b940c5e.png)


首先需要有安装Python运行环境，从Core ML Tools4.1版本开始将不再支持Python2，因此建议直接使用Python3。安装Python会默认安装pip工具，使用如下命令来安装Core ML Tools:

```
pip install coremltools
```

coremltools模块并不包含三方库（如TensorFlow），因此安装会比加快。

要使用三方的模型，需要做如下几步操作：

1.  下载三方模型。
2.  将三方模型转换为CoreML格式。
3.  设置CoreML模型的元数据。
4.  进行测试验证。
5.  存储模型，之后在Xcode中进行使用即可。

其中最核心的是模型的转换和元数据的写入。

以TensorFlow的MobileNetV2模型为例，我们下面尝试将其转换成CoreML模型。要转换TensorFlow格式的模型，首先需要安装对应的框架，使用pip来安装如下依赖：

```
pip install tensorflow h5py pillow

```

第一步，下载三方模型，使用tensorflow框架提供的API可以将模型加载的到内存中去，代码如下：

```python
import tensorflow as tf 

keras_model = tf.keras.applications.MobileNetV2(
    weights="imagenet", 
    input_shape=(224, 224, 3,),
    classes=1000,
)
```

其中applications.MobileNetV2是tensorflow框架中提供好的API，在此文档中可以查看这个API的更多用法：

[https://www.tensorflow.org/api\_docs/python/tf/keras/applications/mobilenet\_v2/MobileNetV2](https://www.tensorflow.org/api_docs/python/tf/keras/applications/mobilenet_v2/MobileNetV2)

同时我们还需要下载一个索引文件，此文件定义了模型所能预测的标签数据，Python代码如下：

```python
import urllib
# 模型对应的索引文件地址
label_url = 'https://storage.googleapis.com/download.tensorflow.org/data/ImageNetLabels.txt'
class_labels = urllib.request.urlopen(label_url).read().splitlines()
class_labels = class_labels[1:]
assert len(class_labels) == 1000
for i, label in enumerate(class_labels):
  if isinstance(label, bytes):
    class_labels[i] = label.decode("utf8")
```

下面进行模型的转换，直接使用coremltools模块提供的API即可，如下：

```python
import coremltools as ct

# 定义输入
image_input = ct.ImageType(shape=(1, 224, 224, 3,),
                           bias=[-1,-1,-1], scale=1/127)

# 设置可预测的标签
classifier_config = ct.ClassifierConfig(class_labels)

# 进行模型转换
model = ct.convert(
    keras_model, 
    inputs=[image_input], 
    classifier_config=classifier_config,
)
```

这一步做完成，实际上已经完整了核心的转换部分，我们还需要为model实例追加一些元数据，你应该还记得，将CoreML模型引入Xcode工程后，可以在Xcode中看到模型的简介和使用方法等信息，这些信息就是通过追加元数据写入的。上面实例代码中，默认将其转换成neuralnetwork（神经网络）模式的模型，转换模型时我们也可以选择了添加conver_to参数为mlprogram，这表示将模型转换成CoreML程序模式的。

写入元数据实例代码如下：

```python
# 写入元数据
model.input_description["input_1"] = "输入要分类的图片"
model.output_description["classLabel"] = "最可靠的结果"

# 模型作者
model.author = "TensorFlow转换"

# 许可
model.license = "Please see https://github.com/tensorflow/tensorflow for license information, and https://github.com/tensorflow/models/tree/master/research/slim/nets/mobilenet for the original source of the model."

# 描述
model.short_description = "图片识别模型"

# 版本号
model.version = "1.0"
```

最后，就可以进行模型的导出了，代码如下：

```python
# 存储模型
model.save("MobileNetV2.mlmodel")
```

需要注意，此时导出的模型格式，与前面转换成设置的模型类型有关，转换为mlprogram模式的模型需要导出mlpackage格式的，转换为neuralnetwork的模型需要导出为mlmodel格式的。

完整的Python文件代码如下：

```python
import tensorflow as tf 
# 加载模型
keras_model = tf.keras.applications.MobileNetV2(
    weights="imagenet", 
    input_shape=(224, 224, 3,),
    classes=1000,
)


import urllib
# 模型对应的索引文件地址
label_url = 'https://storage.googleapis.com/download.tensorflow.org/data/ImageNetLabels.txt'
class_labels = urllib.request.urlopen(label_url).read().splitlines()
class_labels = class_labels[1:]
assert len(class_labels) == 1000
for i, label in enumerate(class_labels):
  if isinstance(label, bytes):
    class_labels[i] = label.decode("utf8")


import coremltools as ct

# 定义输入
image_input = ct.ImageType(shape=(1, 224, 224, 3,),
                           bias=[-1,-1,-1], scale=1/127)

# 设置可预测的标签
classifier_config = ct.ClassifierConfig(class_labels)

# 进行模型转换
model = ct.convert(
    keras_model, 
    inputs=[image_input], 
    classifier_config=classifier_config,
)

# 写入元数据
model.input_description["input_1"] = "输入要分类的图片"
model.output_description["classLabel"] = "最可靠的结果"

# 模型作者
model.author = "TensorFlow转换"

# 许可
model.license = "Please see https://github.com/tensorflow/tensorflow for license information, and https://github.com/tensorflow/models/tree/master/research/slim/nets/mobilenet for the original source of the model."

# 描述
model.short_description = "图片识别模型"

# 版本号
model.version = "1.0"

# 存储模型
model.save("XMobileNetV2.mlmodel")
```

运行此Python脚本，如果没有报错，则会在当前脚本的同级目录下生成模型文件，下面我们可以将此模型文件引入到Xcode中，如下：

![截屏2023-07-25 17.27.35.png](https://ucc.alicdn.com/pic/developer-ecology/ww5utkr4n4ptw_04281b6205f44225aee67bcc94a5020c.png)


下面可以尝试下此模型的预测效果，如下：

![截屏2023-07-25 17.28.52.png](https://ucc.alicdn.com/pic/developer-ecology/ww5utkr4n4ptw_de40bd74d29b4cd885f914788c56b797.png)


可以看到，将三方模型转成成CoreML模型非常简单，同理对于PyTroch，LibSVM等模型也类似，安装对应的三方模块，读取模型后进行转换即可。
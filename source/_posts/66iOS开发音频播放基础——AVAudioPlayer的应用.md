---
title: iOS开发音频播放基础——AVAudioPlayer的应用
date: 2015-05-26
categories: iOS逻辑初窥
tags: []
---
## iOS音频开发——AVAudioPlayer应用

AVAudioPlayer是系统提供给我们的一个音频播放类，在AVFoundation框架下，通过它，我们可以实现一个功能强大的音乐播放器。首先，在项目中我们需要导入AVFoundation这个框架。

### ![](http://static.oschina.net/uploads/space/2015/0526/140410_HfD6_2340880.png)

### 一、AVAudioPlayer方法与属性详解

初始化方法有两种，通过音频的路径或者音频data数据初始化player对象

```
- (instancetype)initWithContentsOfURL:(NSURL *)url error:(NSError **)outError;
- (instancetype)initWithData:(NSData *)data error:(NSError **)outError;
```

注意：支持的音频格式有:AAC,ALAC,HE-AAC,iLBC,IMA4,MP3.

准备播放音频，返回值标志是否解析成功，是否可以播放。

```
- (BOOL)prepareToPlay;
```

开始播放音频

```
- (BOOL)play;
```

在一段时间间隔后播放

```
- (BOOL)playAtTime:(NSTimeInterval)time;
```

暂停播放，并且准备好继续播放

```
- (void)pause;
```

停止播放，不再准备好继续播放

```
- (void)stop;
```

获取是否正在播放

```
@property(readonly, getter=isPlaying) BOOL playing;
```

获取当前音频声道数

```
@property(readonly) NSUInteger numberOfChannels;
```

获取当前音频时长

```
@property(readonly) NSTimeInterval duration;
```

获取创建时的音频路径

```
@property(readonly) NSURL *url;
```

获取创建时的音频数据

```
@property(readonly) NSData *data;
```

设置声道偏移量，0为中心，-1为只有左声道，1为只有右声道

```
@property float pan;
```

设置音频音量，取值为0-1之间

```
@property float volume;
```

设置是否可以改变播放速度

```
@property BOOL enableRate;
```

注意:设置这个属性前必须先调用prepareToPlay这个方法。

设置播放速度，1为正常，0.5为一半速度，2.0为2倍速度

```
@property float rate;
```

设置当前播放的时间点

```
@property NSTimeInterval currentTime;
```

设置音频播放循环次数

```
@property NSInteger numberOfLoops;
```

获取音频设置字典

```
@property(readonly) NSDictionary *settings;
```

是否开启仪表计数功能

```
@property(getter=isMeteringEnabled) BOOL meteringEnabled;
```

更新仪表计数的值

```
- (void)updateMeters;
```

获取指定声道音频峰值

```
- (float)peakPowerForChannel:(NSUInteger)channelNumber;
```

获取指定声道音频平均值

```
- (float)averagePowerForChannel:(NSUInteger)channelNumber;
```

### 二、AVAudioPlayerDelegate方法详解

音频播放结束后调用的函数

```
- (void)audioPlayerDidFinishPlaying:(AVAudioPlayer *)player successfully:(BOOL)flag;
```

播放遇到错误时调用的函数

```
- (void)audioPlayerDecodeErrorDidOccur:(AVAudioPlayer *)player error:(NSError *)error;
```

学习使用 欢迎转载

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

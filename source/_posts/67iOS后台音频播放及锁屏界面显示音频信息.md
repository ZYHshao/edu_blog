---
title: iOS后台音频播放及锁屏界面显示音频信息用
date: 2015-05-26
categories: iOS逻辑初窥
tags: []
---
## iOS后台播放音乐及用户交互处理

后台播放是任何一个音频软件都支持的功能，在上一篇博客中，详细介绍了使用AVAudioPlayer播放音频的方法，这篇博客将对后台的处理做介绍，关于播放与设置音频的博客地址：[http://my.oschina.net/u/2340880/blog/420129](http://my.oschina.net/u/2340880/blog/420129)。

### 一、设置后台播放

iOS设置后台音频播放的步骤非常简单，首先需要在系统设置的plist文件中添加一个键Required background modes，值为App plays audio or streams audio/video using AirPlay，如下：

![](http://static.oschina.net/uploads/space/2015/0526/153912_NYRK_2340880.png)

然后进行如下代码设置：

```
    AVAudioSession *session = [AVAudioSession sharedInstance];
    [session setActive:YES error:nil];
    [session setCategory:AVAudioSessionCategoryPlayback error:nil];
```

此时播放音频时我们点击HOME回到主页面，会发现音频不会停，已经实现后台播放的功能。

### 二、设置后台用户交互

在appDelegate中，我们需要先注册响应后台控制：

```
[[UIApplication sharedApplication] beginReceivingRemoteControlEvents];
```

然后在appDelegate中我们实现如下函数处理后台传递给我们的信息：

```
-(void)remoteControlReceivedWithEvent:(UIEvent *)event{
    if (event.type==UIEventTypeRemoteControl) {
        NSLog(@"%ld",event.subtype);
    }
}
```

event中的subtype是操作类型，我们打开系统桌面抽屉，可以看到如下的控制键：

![](http://static.oschina.net/uploads/space/2015/0526/155022_kapo_2340880.png)

subtype中的枚举便是点击这些控制键后传递给我们的消息，我们可以根据这些消息在app内做逻辑处理。枚举如下，其中只有100之后的在音频控制中对我们有效：

```
typedef NS_ENUM(NSInteger, UIEventSubtype) {
    // available in iPhone OS 3.0
    UIEventSubtypeNone                              = 0,
    // for UIEventTypeMotion, available in iPhone OS 3.0
    UIEventSubtypeMotionShake                       = 1,
    //这之后的是我们需要关注的枚举信息
    // for UIEventTypeRemoteControl, available in iOS 4.0
    //点击播放按钮或者耳机线控中间那个按钮
    UIEventSubtypeRemoteControlPlay                 = 100,
    //点击暂停按钮
    UIEventSubtypeRemoteControlPause                = 101,
    //点击停止按钮
    UIEventSubtypeRemoteControlStop                 = 102,
    //点击播放与暂停开关按钮(iphone抽屉中使用这个)
    UIEventSubtypeRemoteControlTogglePlayPause      = 103,
    //点击下一曲按钮或者耳机中间按钮两下
    UIEventSubtypeRemoteControlNextTrack            = 104,
    //点击上一曲按钮或者耳机中间按钮三下   
    UIEventSubtypeRemoteControlPreviousTrack        = 105,
    //快退开始 点击耳机中间按钮三下不放开
    UIEventSubtypeRemoteControlBeginSeekingBackward = 106,
    //快退结束 耳机快退控制松开后
    UIEventSubtypeRemoteControlEndSeekingBackward   = 107,
    //开始快进 耳机中间按钮两下不放开
    UIEventSubtypeRemoteControlBeginSeekingForward  = 108,
    //快进结束 耳机快进操作松开后
    UIEventSubtypeRemoteControlEndSeekingForward    = 109,
};
```

### 三、设置后台信息显示及锁屏界面设置

设置锁屏界面显示信息的原理是通过设置一个系统的字典，当音频开始播放时，系统会自动从这个字典中读取要显示的信息，如果需要动态显示，我们只需要不断更新这个字典即可。首先需要添加<MediaPlayer/MediaPlayer.h>这个头文件。

代码示例如下：

```
 NSMutableDictionary *dict = [[NSMutableDictionary alloc] init];
    //设置歌曲题目
    [dict setObject:@"题目" forKey:MPMediaItemPropertyTitle];
    //设置歌手名
    [dict setObject:@"歌手" forKey:MPMediaItemPropertyArtist];
    //设置专辑名
    [dict setObject:@"专辑" forKey:MPMediaItemPropertyAlbumTitle];
    //设置显示的图片
    UIImage *newImage = [UIImage imageNamed:@"43.png"];
    [dict setObject:[[MPMediaItemArtwork alloc] initWithImage:newImage]
             forKey:MPMediaItemPropertyArtwork];
    //设置歌曲时长
    [dict setObject:[NSNumber numberWithDouble:300] forKey:MPMediaItemPropertyPlaybackDuration];
    //设置已经播放时长
    [dict setObject:[NSNumber numberWithDouble:150] forKey:MPNowPlayingInfoPropertyElapsedPlaybackTime]; 
    //更新字典
    [[MPNowPlayingInfoCenter defaultCenter] setNowPlayingInfo:dict];
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/0526/162609_oD80_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

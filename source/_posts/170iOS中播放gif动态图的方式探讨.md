---
title: iOS中播放gif动态图的方式探讨
date: 2016-01-24
categories: iOS逻辑初窥
tags: []
---
## iOS中播放gif动态图的方式探讨

### 一、引言

    在iOS开发中，UIImageView类专门来负责图片数据的渲染，并且UIImageView也有帧动画的方法来播放一组图片，但是对于gif类型的数据，UIImageView中并没有现成的接口提供给开发者使用，在iOS中一般可以通过两种方式来播放gif动态图，一种方式是通过ImageIO框架中的方法将gif文件中的数据进行解析，再使用coreAnimation核心动画来播放gif动画，另一种方式计较简单，可以直接通过webView来渲染gif图。

### 二、为原生的UIImageView添加类别来支持gif动态图的播放

     gif动态图文件中包含了一组图片及其信息，信息主要记录着每一帧图片播放的时间，我们如果获取到了gif文件中所有的图片同时又获取到每一帧图片播放的时间，就可以为UIImageView添加核心动画的方法来让其播放gif的内容了。

    首先解析gif文件中的数据，代码如下：

```
//要引入ImageIO库
#import <ImageIO/ImageIO.h>

//解析gif文件数据的方法 block中会将解析的数据传递出来
-(void)getGifImageWithUrk:(NSURL *)url
               returnData:(void(^)(NSArray<UIImage *> * imageArray,
                                NSArray<NSNumber *>*timeArray,
                                CGFloat totalTime,
                                NSArray<NSNumber *>* widths,
                                NSArray<NSNumber *>* heights))dataBlock{
    //通过文件的url来将gif文件读取为图片数据引用
    CGImageSourceRef source = CGImageSourceCreateWithURL((CFURLRef)url, NULL);
    //获取gif文件中图片的个数
    size_t count = CGImageSourceGetCount(source);
    //定义一个变量记录gif播放一轮的时间
    float allTime=0;
    //存放所有图片
    NSMutableArray * imageArray = [[NSMutableArray alloc]init];
    //存放每一帧播放的时间
    NSMutableArray * timeArray = [[NSMutableArray alloc]init];
    //存放每张图片的宽度 （一般在一个gif文件中，所有图片尺寸都会一样）
    NSMutableArray * widthArray = [[NSMutableArray alloc]init];
    //存放每张图片的高度
    NSMutableArray * heightArray = [[NSMutableArray alloc]init];
    //遍历
    for (size_t i=0; i<count; i++) {
        CGImageRef image = CGImageSourceCreateImageAtIndex(source, i, NULL);
        [imageArray addObject:(__bridge UIImage *)(image)];
        CGImageRelease(image);
        //获取图片信息
        NSDictionary * info = (__bridge NSDictionary*)CGImageSourceCopyPropertiesAtIndex(source, i, NULL);
        CGFloat width = [[info objectForKey:(__bridge NSString *)kCGImagePropertyPixelWidth] floatValue];
        CGFloat height = [[info objectForKey:(__bridge NSString *)kCGImagePropertyPixelHeight] floatValue];
        [widthArray addObject:[NSNumber numberWithFloat:width]];
        [heightArray addObject:[NSNumber numberWithFloat:height]];
        NSDictionary * timeDic = [info objectForKey:(__bridge NSString *)kCGImagePropertyGIFDictionary];
        CGFloat time = [[timeDic objectForKey:(__bridge NSString *)kCGImagePropertyGIFDelayTime]floatValue];
        allTime+=time;
        [timeArray addObject:[NSNumber numberWithFloat:time]];
        CFRelease(info);
    }
    CFRelease(source);
    dataBlock(imageArray,timeArray,allTime,widthArray,heightArray);
}
```

为UIImageView添加一个设置gif图内容的方法：

```
-(void)yh_setImage:(NSURL *)imageUrl{
        __weak id __self = self;
        [self getGifImageWithUrk:imageUrl returnData:^(NSArray<UIImage *> *imageArray, NSArray<NSNumber *> *timeArray, CGFloat totalTime, NSArray<NSNumber *> *widths, NSArray<NSNumber *> *heights) {
            //添加帧动画
            CAKeyframeAnimation *animation = [CAKeyframeAnimation animationWithKeyPath:@"contents"];
            NSMutableArray * times = [[NSMutableArray alloc]init];
            float currentTime = 0;
            //设置每一帧的时间占比
            for (int i=0; i<imageArray.count; i++) {
                [times addObject:[NSNumber numberWithFloat:currentTime/totalTime]];
                currentTime+=[timeArray[i] floatValue];
            }
            [animation setKeyTimes:times];
            [animation setValues:imageArray];
            [animation setTimingFunction:[CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionLinear]];
            //设置循环
            animation.repeatCount= MAXFLOAT;
            //设置播放总时长
            animation.duration = totalTime;
            //Layer层添加
            [[(UIImageView *)__self layer]addAnimation:animation forKey:@"gifAnimation"];
        }];
}
```

使用代码示例如下：

```
    UIImageView * imageView = [[UIImageView alloc]initWithFrame:CGRectMake(0,0 , 320, 200)];
    NSURL * url = [[NSURL alloc]initFileURLWithPath:[[NSBundle mainBundle] pathForResource:imageName ofType:nil]];
    [imageView yh_setImage:url];
    [self.view addSubview:imageView];
```

![](http://static.oschina.net/uploads/space/2016/0124/120319_dq0O_2340880.png)

### 三、使用UIWebView来加载gif动态图数据

    iOS中的UIWebView功能十分强大，可以通过UIWebView为载体，来展示gif图。并且这种方法也十分简单，代码如下：

```
         //读取gif数据
         NSData *gifData = [NSData dataWithContentsOfURL:imageUrl];
        UIWebView *webView = [[UIWebView alloc] initWithFrame:CGRectMake(0, 0, self.frame.size.width, self.frame.size.height)];
        //取消回弹效果
        webView.scrollView.bounces=NO;
        webView.backgroundColor = [UIColor clearColor];
        //设置缩放模式
        webView.scalesPageToFit = YES;
        //用webView加载数据
        [webView loadData:gifData MIMEType:@"image/gif" textEncodingName:nil baseURL:nil];
```

### 四、两种加载gif动态图方式的优劣

    经过测试，从加载速度上来说，通过UIImageView类别加载的方式更加快速，UIWebView的方式加载时间会稍长，但是从性能上来比较，WebView的方式性能更优，播放的gif动态图更加流畅。在开发中，可以根据需求，适当选择，例如虽然WebView加载的方式性能更好，但是在许多情况下，原生的UIImageView能够更加自由的让开发者进行扩展。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

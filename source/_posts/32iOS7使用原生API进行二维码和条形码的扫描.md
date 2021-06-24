---
title: iOS7使用原生API进行二维码和条形码的扫描
date: 2015-04-24
categories: iOS逻辑初窥
tags: []
---
## 使用iOS7原生API进行二维码条形码的扫描

IOS7之前，开发者进行扫码编程时，一般会借助第三方库。常用的是ZBarSDK，IOS7之后，系统的AVMetadataObject类中，为我们提供了解析二维码的接口。经过测试，使用原生API扫描和处理的效率非常高，远远高于第三方库。

### 一、使用方法示例

官方提供的接口非常简单，代码如下：

```
@interface ViewController ()<AVCaptureMetadataOutputObjectsDelegate>//用于处理采集信息的代理
{
    AVCaptureSession * session;//输入输出的中间桥梁
}
@end
@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    //获取摄像设备
    AVCaptureDevice * device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    //创建输入流
    AVCaptureDeviceInput * input = [AVCaptureDeviceInput deviceInputWithDevice:device error:nil];
    //创建输出流
    AVCaptureMetadataOutput * output = [[AVCaptureMetadataOutput alloc]init];
    //设置代理 在主线程里刷新
    [output setMetadataObjectsDelegate:self queue:dispatch_get_main_queue()];
    
    //初始化链接对象
    session = [[AVCaptureSession alloc]init];
    //高质量采集率
    [session setSessionPreset:AVCaptureSessionPresetHigh];
    
    [session addInput:input];
    [session addOutput:output];
    //设置扫码支持的编码格式(如下设置条形码和二维码兼容)
    output.metadataObjectTypes=@[AVMetadataObjectTypeQRCode,AVMetadataObjectTypeEAN13Code, AVMetadataObjectTypeEAN8Code, AVMetadataObjectTypeCode128Code];
       
    AVCaptureVideoPreviewLayer * layer = [AVCaptureVideoPreviewLayer layerWithSession:session];
    layer.videoGravity=AVLayerVideoGravityResizeAspectFill;
    layer.frame=self.view.layer.bounds;
    [self.view.layer insertSublayer:layer atIndex:0];
    //开始捕获
    [session startRunning];
}
```

之后我们的UI上已经可以看到摄像头捕获的内容，只要实现代理中的方法，就可以完成二维码条形码的扫描：

```
-(void)captureOutput:(AVCaptureOutput *)captureOutput didOutputMetadataObjects:(NSArray *)metadataObjects fromConnection:(AVCaptureConnection *)connection{
    if (metadataObjects.count>0) {
        //[session stopRunning];
        AVMetadataMachineReadableCodeObject * metadataObject = [metadataObjects objectAtIndex : 0 ];
        //输出扫描字符串
        NSLog(@"%@",metadataObject.stringValue);
    }
}
```

### 二、一些优化

通过上面的代码测试，我们可以发现系统的解析处理效率是相当的高，IOS官方提供的API也确实非常强大，然而，我们可以做进一步的优化，将效率更加提高：

首先，AVCaptureMetadataOutput类中有一个这样的属性(在IOS7.0之后可用)：

[@property](http://my.oschina.net/property)(nonatomic) CGRect rectOfInterest;

这个属性大致意思就是告诉系统它需要注意的区域，大部分APP的扫码UI中都会有一个框，提醒你将条形码放入那个区域，这个属性的作用就在这里，它可以设置一个范围，只处理在这个范围内捕获到的图像的信息。如此一来，可想而知，我们代码的效率又会得到很大的提高，在使用这个属性的时候。需要几点注意：

1、这个CGRect参数和普通的Rect范围不太一样，它的四个值的范围都是0-1，表示比例。

2、经过测试发现，这个参数里面的x对应的恰恰是距离左上角的垂直距离，y对应的是距离左上角的水平距离。

3、宽度和高度设置的情况也是类似。

3、举个例子如果我们想让扫描的处理区域是屏幕的下半部分，我们这样设置

```
output.rectOfInterest=CGRectMake(0.5,0,0.5, 1);
```

具体apple为什么要设计成这样，或者是这个参数我的用法那里不对，还需要了解的朋友给个指导。

疏漏之处 欢迎指正

学习使用 欢迎转载

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

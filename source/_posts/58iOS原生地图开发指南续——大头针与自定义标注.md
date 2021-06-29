---
title: iOS原生地图开发指南续——大头针与自定义标注
date: 2015-05-15
categories: iOS逻辑初窥
tags: []
---
## iOS原生地图开发指南续——大头针与自定义标注

在上一篇博客中[http://my.oschina.net/u/2340880/blog/415360](http://my.oschina.net/u/2340880/blog/415360)系统总结了iOS原生地图框架MapKit中主体地图的设置与应用。这篇是上一篇的一个后续，总结了系统的大头针视图以及自定义标注视图的方法。

### 一、先来认识一个协议MKAnnotation

官方文档告诉我们，所有标注的类必须遵守这个协议。所以可以了解，标注这个概念在逻辑属性和视图上是分开的。先来看下这个协议声明了哪些方法：

```
@protocol MKAnnotation <NSObject>
@property (nonatomic, readonly) CLLocationCoordinate2D coordinate;//地理坐标位置
@optional
@property (nonatomic, readonly, copy) NSString *title;//标题
@property (nonatomic, readonly, copy) NSString *subtitle;//副标题
//拖动时调用
- (void)setCoordinate:(CLLocationCoordinate2D)newCoordinate;

@end
```

### 二、创建一个系统标注大头针

```
- (void)viewDidLoad {
    [super viewDidLoad];
    //初始化地图
    mapView =[[MKMapView alloc]initWithFrame:self.view.frame];
    //设置代理
    mapView.delegate=self;
    //设置位置
    mapView.region=MKCoordinateRegionMake(CLLocationCoordinate2DMake(39.26, 116.3), MKCoordinateSpanMake(1.8, 1));
    mapView.mapType=MKMapTypeStandard;
    //初始化一个大头针类
    MKPointAnnotation * ann = [[MKPointAnnotation alloc]init];
    //设置大头针坐标
    ann.coordinate=CLLocationCoordinate2DMake(39.26, 116.3);
    ann.title=@"我";
    ann.subtitle=@"看这里";
    [mapView addAnnotation:ann];
    [self.view addSubview:mapView];
}
```

效果如下：  
![](http://static.oschina.net/uploads/space/2015/0515/164550_5WgF_2340880.png)

重绘大头针视图，大头针渲染时会调用地图代理的方法，我们可以重写这个方法进行大头针的重绘，来更改其颜色：

```
-(MKAnnotationView *)mapView:(MKMapView *)mapView viewForAnnotation:(id<MKAnnotation>)annotation{
    //创建一个系统大头针对象
    MKPinAnnotationView * view = [[MKPinAnnotationView alloc]initWithAnnotation:annotation reuseIdentifier:@"pin"];
    view.pinColor=MKPinAnnotationColorGreen;//设置颜色为绿色
    return view;
}
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/0515/170439_spnd_2340880.png)

MKAnnotationView是标注的视图类，一会我们通过它来自定义我们自己的标注，先来看MKPinAnnotationView这个类，这个类继承于MKAnnotationView，是一个大头针视图类。这个类根简单，只有一下两个属性：

[@property](http://my.oschina.net/property) (nonatomic) MKPinAnnotationColor pinColor;

设置大头针的颜色，枚举如下：

```
typedef NS_ENUM(NSUInteger, MKPinAnnotationColor) {
    MKPinAnnotationColorRed = 0,//红色
    MKPinAnnotationColorGreen,//绿色
    MKPinAnnotationColorPurple//紫色
};
```

[@property](http://my.oschina.net/property) (nonatomic) BOOL animatesDrop;

设置添加时是否显示降落动画

### 三、自定义标注视图

```
-(MKAnnotationView *)mapView:(MKMapView *)mapView viewForAnnotation:(id<MKAnnotation>)annotation{
    MKAnnotationView * view = [[MKAnnotationView alloc]initWithAnnotation:annotation reuseIdentifier:@"annotation"];
    //设置标注的图片
    view.image=[UIImage imageNamed:@"保温车0.png"];
    //点击显示图详情视图 必须MKPointAnnotation对象设置了标题和副标题
    view.canShowCallout=YES;
    //创建了两个view
    UIView * view1 = [[UIView alloc]initWithFrame:CGRectMake(0, 0, 50, 50)];
    view1.backgroundColor=[UIColor redColor];
    UIView * view2 = [[UIView alloc]initWithFrame:CGRectMake(0, 0, 30, 50)];
    view2.backgroundColor=[UIColor blueColor];
    //设置左右辅助视图
    view.leftCalloutAccessoryView=view1;
    view.rightCalloutAccessoryView=view2;
    //设置拖拽 可以通过点击不放进行拖拽
    view.draggable=YES;
    return view;
}
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/0515/174218_jqQb_2340880.png)

### 四、标注视图类MKAnnotationView的其他常用属性解读

[@property](http://my.oschina.net/property) (nonatomic) CGPoint centerOffset;

视图中心的偏移量

[@property](http://my.oschina.net/property) (nonatomic) CGPoint calloutOffset;

点击后弹出视图的偏移量

[@property](http://my.oschina.net/property) (nonatomic, getter=isEnabled) BOOL enabled;

设置是否有效

@property (nonatomic, getter=isHighlighted) BOOL highlighted;

是否高亮状态

@property (nonatomic) CGPoint leftCalloutOffset;

设置左辅助视图的偏移量

@property (nonatomic) CGPoint rightCalloutOffset;

设置右辅助视图的偏移量

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

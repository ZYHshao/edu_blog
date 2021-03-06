---
title: iOS流布局UICollectionView系列六——将布局从平面应用到空间
date: 2015-10-28
categories: iOS之UI控件
tags: []
---
## iOS流布局UICollectionView系列六——将布局从平面应用到空间

### 一、引言

        前面，我们将布局由线性的瀑布流布局扩展到了圆环布局，这使我们使用UICollectionView的布局思路大大迈进了一步，这次，我们玩的更加炫一些，想办法将布局应用的空间，你是否还记得，在管理布局的item的具体属性的类UICollectionViewLayoutAttributrs类中，有transform3D这个属性，通过这个属性的设置，我们真的可以在空间的坐标系中进行布局设计。iOS系统的控件中，也并非没有这样的先例，UIPickerView就是很好的一个实例，这篇博客，我们就通过使用UICollectionView实现一个类似系统的UIPickerView的布局视图，来体会UICollectionView在3D控件布局的魅力。系统的pickerView效果如下：

![](http://static.oschina.net/uploads/space/2015/1028/221936_DNmc_2340880.png)

### 二、先来实现一个炫酷的滚轮空间布局

        万丈的高楼也是由一砖一瓦堆砌而成，在我们完全模拟系统pickerView前，我们应该先将视图的布局摆放这一问题解决。我们依然来创建一个类，继承于UICollectionViewLayout：

```
@interface MyLayout : UICollectionViewLayout

@end
```

对于.m文件的内容，前几篇博客中我们都是在prepareLayout中进行布局的静态设置，那是因为我们前几篇博客中的布局都是静态的，布局并不会随着我们的手势操作而发生太大的变化，因此我们全部在prepareLayout中一次配置完了。而我们这次要讨论的布局则不同，pickerView会随着我们手指的拖动而进行滚动，因此UICollectionView中的每一个item的布局是在不断变化的，所以这次，我们采用动态配置的方式，在layoutAttributesForItemAtIndexPath方法中进行每个item的布局属性设置。

        至于layoutAttributesForItemAtIndexPath方法，它也是UICollectionViewLayout类中的方法，用于我们自定义时进行重写，至于为什么动态布局要在这里面配置item的布局属性，后面我们会了解到。

        在编写我们的布局类之前，先做好准备工作，在viewController中，实现如下代码：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    MyLayout * layout = [[MyLayout alloc]init];
     UICollectionView * collect  = [[UICollectionView alloc]initWithFrame:CGRectMake(0, 0, 320, 400) collectionViewLayout:layout];
    collect.delegate=self;
    collect.dataSource=self;
    [collect registerClass:[UICollectionViewCell class] forCellWithReuseIdentifier:@"cellid"];
    [self.view addSubview:collect];
}

-(NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView{
    return 1;
}
-(NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section{
    return 10;
}
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewCell * cell  = [collectionView dequeueReusableCellWithReuseIdentifier:@"cellid" forIndexPath:indexPath];
    cell.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    UILabel * label = [[UILabel alloc]initWithFrame:CGRectMake(0, 0, 250, 80)];
    label.text = [NSString stringWithFormat:@"我是第%ld行",(long)indexPath.row];
    [cell.contentView addSubview:label];
    return cell;
}
```

上面我创建了10个Item，并且在每个Item上添加了一个标签，标写是第几行。

在我们自定义的布局类中重写layoutAttributesForElementsInRect，在其中返回我们的布局数组：

```
-(NSArray<UICollectionViewLayoutAttributes *> *)layoutAttributesForElementsInRect:(CGRect)rect{
    NSMutableArray * attributes = [[NSMutableArray alloc]init];
    //遍历设置每个item的布局属性
    for (int i=0; i<[self.collectionView numberOfItemsInSection:0]; i++) {
        [attributes addObject:[self layoutAttributesForItemAtIndexPath:[NSIndexPath indexPathForItem:i inSection:0]]];
    }
    return attributes;
}
```

之后，在我们布局类中重写layoutAttributesForItemAtIndexPath方法：

```
-(UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath{
    //创建一个item布局属性类
    UICollectionViewLayoutAttributes * atti = [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:indexPath];
    //获取item的个数
    int itemCounts = (int)[self.collectionView numberOfItemsInSection:0];
    //设置每个item的大小为260*100
    atti.size = CGSizeMake(260, 100);  
   /*
   后边介绍的代码添加在这里
   
   */ 
    return atti;
}
```

上面的代码中，我们什么都没有做，下面我们一步步来实现3D的滚轮效果。

首先，我们先将所有的item的位置都设置为collectionView的中心：

```
atti.center = CGPointMake(self.collectionView.frame.size.width/2, self.collectionView.frame.size.height/2);
```

这时，如果我们运行程序的话，所有item都将一层层贴在屏幕的中央，如下：

![](http://static.oschina.net/uploads/space/2015/1028/214944_3Vp7_2340880.png)

很丑对吧，之后我们来设置每个item的3D效果,在上面的布局方法中添加如下代码:

```
    //创建一个transform3D类
    //CATransform3D是一个类似矩阵的结构体
    //CATransform3DIdentity创建空得矩阵
    CATransform3D trans3D = CATransform3DIdentity;
    //这个值设置的是透视度，影响视觉离投影平面的距离
    trans3D.m34 = -1/900.0;
    //下面这些属性 后面会具体介绍
    //这个是3D滚轮的半径
    CGFloat radius = 50/tanf(M_PI*2/itemCounts/2);
    //计算每个item应该旋转的角度
    CGFloat angle = (float)(indexPath.row)/itemCounts*M_PI*2;
    //这个方法返回一个新的CATransform3D对象，在原来的基础上进行旋转效果的追加
    //第一个参数为旋转的弧度，后三个分别对应x，y，z轴，我们需要以x轴进行旋转
    trans3D = CATransform3DRotate(trans3D, angle, 1.0, 0, 0);
    //进行设置
    atti.transform3D = trans3D;
```

        对于上面的radius属性，运用了一些简单的几何和三角函数的知识。如果我们将系统的pickerView沿着y轴旋转90°，你会发现侧面的它是一个规则的正多边形，这里的radius就是这个多边形中心到其边的垂直距离，也是内切圆的半径，所有的item拼成了一个正多边形，示例如下：

![](http://static.oschina.net/uploads/space/2015/1028/222828_1lzx_2340880.png)

通过简单的数学知识，h/2弦对应的角的弧度为2*pi/(边数)/2，在根据三角函数相关知识可知，这个角的正切值为h/2/radius，这就是我们radius的由来。 

        对于angle属性，它是每一个item的x轴旋转度数，如果我们将所有item的中心都放在一点，通过旋转让它们散开如下图所示：

![](http://static.oschina.net/uploads/space/2015/1028/224126_kqNq_2340880.png)

每个item旋转的弧度就是其索引/(2*pi)。

通过上面的设置，我们再运行代码，效果如下：

![](http://static.oschina.net/uploads/space/2015/1028/224557_f2qr_2340880.png)

仔细观察我们可以发现，item以x中轴线进行了旋转平均布局，侧面的效果就是我们上面的简笔画那样，下面要进行我们的第三步了，将这个item，全部沿着其Z轴向前拉，就可以成为我们滚轮的效果，示例图如下：

![](http://static.oschina.net/uploads/space/2015/1028/225143_Px5S_2340880.png)

我们继续在刚才的代码后面添加这行代码：

```
 //这个方法也返回一个transform3D对象，追加平移效果，后面三个参数，对应平移的x，y，z轴，我们沿z轴平移
 trans3D = CATransform3DTranslate(trans3D, 0, 0, radius);
```

再次运行，效果如下：

![](http://static.oschina.net/uploads/space/2015/1028/225453_tEDp_2340880.png)

布局的效果我们已经完成了，离成功很近了对吧，只是现在的布局是静态的，我们不能滑动这个滚轮，我们还需要用动态滑动做一些处理。

### 三、让滚轮滑动起来

            通过上面的努力，我们已经静态布局出了一个类似pickerView的滚轮，现在我们再来添加滑动滚动的效果

        首先，我们需要给collectionView一个滑动的范围，我们以一屏collectionView的滑动距离来当做滚轮滚动一下的参照，我们在布局类中的如下方法中返回滑动区域：

```
-(CGSize)collectionViewContentSize{
    return CGSizeMake(self.collectionView.frame.size.width, self.collectionView.frame.size.height*[self.collectionView numberOfItemsInSection:0]);
}
```

这时我们的collectionView已经可以进行滑动，但是并不是我们想要的效果，滚轮并没有滚动，而是随着滑动出了屏幕，因此，我们需要在滑动的时候不停的动态布局，将滚轮始终固定在collectionView的中心，先需要在布局类中实现如下方法：

```
//返回yes，则一有变化就会刷新布局
-(BOOL)shouldInvalidateLayoutForBoundsChange:(CGRect)newBounds{
    return YES;
    
}
```

将上面的布局的中心点设置加上一个动态的偏移量：

```
 atti.center = CGPointMake(self.collectionView.frame.size.width/2, self.collectionView.frame.size.height/2+self.collectionView.contentOffset.y);
```

现在在运行，会发现滚轮会随着滑动始终固定在中间，但是还是不如人意，滚轮并没有转动起来，我们还需要动态的设置每个item的旋转角度，这样连续看起来，滚轮就转了起来，在上面设置布局的方法中，我们在添加一些处理：

```
    //获取当前的偏移量
    float offset = self.collectionView.contentOffset.y;
    //在角度设置上，添加一个偏移角度
    float angleOffset = offset/self.collectionView.frame.size.height;
    CGFloat angle = (float)(indexPath.row+angleOffset)/itemCounts*M_PI*2;
```

再看看效果，没错，就是这么简单，滚轮已经转了起来。

### 四、让其循环滚动的逻辑

        我们再进一步，如果滚动可以循环，这个控件将更加炫酷，添加这样的逻辑也很简单，通过监测scrollView的偏移量，我们可以对齐进行处理，因为collectionView继承于scrollView，我们可以直接在ViewController中实现其代理方法，如下：

```
-(void)scrollViewDidScroll:(UIScrollView *)scrollView{
    //小于半屏 则放到最后一屏多半屏
    if (scrollView.contentOffset.y<200) {
        scrollView.contentOffset = CGPointMake(0, scrollView.contentOffset.y+10*400);
    //大于最后一屏多一屏 放回第一屏
    }else if(scrollView.contentOffset.y>11*400){
        scrollView.contentOffset = CGPointMake(0, scrollView.contentOffset.y-10*400);
    }
}
```

因为咱们的环状布局，上面的逻辑刚好可以无缝对接，但是会有新的问题，一开始运行，滚轮就是出现在最后一个item的位置，而不是第一个，并且有些相关的地方，我们也需要一些适配：

在viewController中：

```
//一开始将collectionView的偏移量设置为1屏的偏移量
collect.contentOffset = CGPointMake(0, 400);
```

在layout类中：

```
//将滚动范围设置为(item总数+2)*每屏高度 
-(CGSize)collectionViewContentSize{
    return CGSizeMake(self.collectionView.frame.size.width, self.collectionView.frame.size.height*([self.collectionView numberOfItemsInSection:0]+2));
}
```

```
//将计算的具体item角度向前递推一个
CGFloat angle = (float)(indexPath.row+angleOffset-1)/itemCounts*M_PI*2;
```

OK，我们终于大功告成了，可以发现，实现这样一个布局效果炫酷的控件，代码其实并没有多少，相比，数学逻辑要比编写代码本身困难，这十分类似数学中的几何问题，如果你弄清了逻辑，解决是分分钟的事，我们可以通过这样的一个思路，设计更多3D或者平面特效的布局方案，抽奖的转动圆盘，书本的翻页，甚至立体的标签云，UICollectionView都可以实现，这篇博客中的代码在下面的连接中，疏漏之处，欢迎指正！

[http://pan.baidu.com/s/1jGCmbKM](http://pan.baidu.com/s/1jGCmbKM)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

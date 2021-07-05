---
title: iOS流布局UICollectionView系列三——使用FlowLayout进行更灵活布局
date: 2015-10-27
categories: iOS之UI控件
tags: []
---
## iOS流布局UICollectionView系列三——使用FlowLayout进行更灵活布局

### 一、引言

        前面的博客介绍了UICollectionView的相关方法和其协议中的方法，但对布局的管理类UICollectionViewFlowLayout没有着重探讨，这篇博客介绍关于布局的相关设置和属性方法。

UICollectionView的简单使用：[http://my.oschina.net/u/2340880/blog/522613](http://my.oschina.net/u/2340880/blog/522613)  

UICollectionView相关协议方法：[http://my.oschina.net/u/2340880/blog/522613](http://my.oschina.net/u/2340880/blog/522613)

通过layout的设置，我们可以编写更加灵活的布局效果。

### 二、将九宫格式的布局进行升级

        在第一篇博客中，通过UICollectionView，我们很轻松的完成了一个九宫格的布局，但是如此中规中矩的布局方式，有时候并不能满足我们的需求，有时我们需要每一个Item展示不同的大小，代码如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc]init];
    layout.scrollDirection = UICollectionViewScrollDirectionVertical;
    UICollectionView *collect = [[UICollectionView alloc]initWithFrame:CGRectMake(0, 0, 320, 400) collectionViewLayout:layout];
    collect.delegate=self;
    collect.dataSource=self;
    
    [collect registerClass:[UICollectionViewCell class] forCellWithReuseIdentifier:@"cellid"];
  ;
    [self.view addSubview:collect];
    
    
}
//设置每个item的大小，双数的为50*50 单数的为100*100
-(CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout *)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath{
    if (indexPath.row%2==0) {
        return CGSizeMake(50, 50);
    }else{
        return CGSizeMake(100, 100);
    }
}

//代理相应方法
-(NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView{
    return 1;
}
-(NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section{
    return 100;
}
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewCell * cell  = [collectionView dequeueReusableCellWithReuseIdentifier:@"cellid" forIndexPath:indexPath];
    cell.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    return cell;
}
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/1027/161602_qJDH_2340880.png)

现在的布局效果是不是炫酷了许多。

### 三、UICollectionViewFlowLayout相关属性方法

        UICollectionViewFlowLayout是系统提供给我们一个封装好的流布局设置类，其中有一些布局属性我们可以进行设置：

设置行与行之间的间距最小距离

[@property](http://my.oschina.net/property) (nonatomic) CGFloat minimumLineSpacing;

设置列与列之间的间距最小距离

[@property](http://my.oschina.net/property) (nonatomic) CGFloat minimumInteritemSpacing;

设置每个item的大小

[@property](http://my.oschina.net/property) (nonatomic) CGSize itemSize;

设置每个Item的估计大小，一般不需要设置

[@property](http://my.oschina.net/property) (nonatomic) CGSize estimatedItemSize NS\_AVAILABLE\_IOS(8_0);

设置布局方向

[@property](http://my.oschina.net/property) (nonatomic) UICollectionViewScrollDirection scrollDirection;

这个UICollectionViewScrollDirection的枚举如下：

```
typedef NS_ENUM(NSInteger, UICollectionViewScrollDirection) {
    UICollectionViewScrollDirectionVertical,//水平布局
    UICollectionViewScrollDirectionHorizontal//垂直布局
};
```

设置头视图尺寸大小

@property (nonatomic) CGSize headerReferenceSize;

设置尾视图尺寸大小

@property (nonatomic) CGSize footerReferenceSize;

设置分区的EdgeInset

@property (nonatomic) UIEdgeInsets sectionInset;

这个属性可以设置分区的偏移量，例如我们在刚才的例子中添加如下设置：

```
 layout.sectionInset = UIEdgeInsetsMake(20, 20, 20, 20);
```

效果如下，会看到分区的边界闪出了20像素

![](http://static.oschina.net/uploads/space/2015/1027/170052_X0F1_2340880.png)

下面这两个方法设置分区的头视图和尾视图是否始终固定在屏幕上边和下边

@property (nonatomic) BOOL sectionHeadersPinToVisibleBounds NS\_AVAILABLE\_IOS(9_0);

@property (nonatomic) BOOL sectionFootersPinToVisibleBounds NS\_AVAILABLE\_IOS(9_0);

### 四、动态的配置layout的相关属性UICollectionViewDelegateFlowLayout

        上面的方法在创建FlowLayout时静态的进行设置，如果我们需要动态的设置这些属性，就像我们例子中的，每个item的大小会有差异，我们可以通过代理来实现。

        UICollectionViewDelegateFlowLayout是UICollectionViewDelegate的子协议，其中常用方法如下，我们只需要实现我们需要的即可：

动态设置每个Item的尺寸大小

\- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath;

动态设置每个分区的EdgeInsets

\- (UIEdgeInsets)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout insetForSectionAtIndex:(NSInteger)section;

动态设置每行的间距大小

\- (CGFloat)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout minimumLineSpacingForSectionAtIndex:(NSInteger)section;

动态设置每列的间距大小

\- (CGFloat)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout minimumInteritemSpacingForSectionAtIndex:(NSInteger)section;

动态设置某个分区头视图大小

\- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout referenceSizeForHeaderInSection:(NSInteger)section;

动态设置某个分区尾视图大小

\- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout referenceSizeForFooterInSection:(NSInteger)section;

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

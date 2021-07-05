---
title: iOS流布局UICollectionView系列四——自定义FlowLayout进行瀑布流布局
date: 2015-10-27
categories: iOS之UI控件
tags: []
---
## iOS流布局UICollectionView系列四——自定义FlowLayout进行瀑布流布局

### 一、引言

        前几篇博客从UICollectionView的基础应用到设置UICollectionViewFlowLayout更加灵活的进行布局，但都限制在系统为我们准备好的布局框架中，还是有一些局限性，例如，如果我要进行瀑布流似的不定高布局，前面的方法就很难满足我们的需求了，如下：

![](http://static.oschina.net/uploads/space/2015/1027/185334_wXcE_2340880.png)

这种布局无疑在app的应用中更加广泛，商品的展示，书架书目的展示，都会倾向于采用这样的布局方式，当然，通过自定义FlowLayout，我们也很容易实现。

### 二、进行自定义瀑布流布局

        首先，我们新建一个文件继承于UICollectionViewFlowLayout：

```
@interface MyLayout : UICollectionViewFlowLayout
```

为了演示的方面，这里我不错更多的封装，添加一个属性，直接让外界将item个数传递进来，我们把重心方法重写布局的方法上：

```
@interface MyLayout : UICollectionViewFlowLayout
@property(nonatomic,assign)int itemCount;
@end
```

前面说过，UICollectionViewFlowLayout是一个专门用来管理collectionView布局的类，因此，collectionView在进行UI布局前，会通过这个类的对象获取相关的布局信息，FlowLayout类将这些布局信息全部存放在了一个数组中，数组中是UICollectionViewLayoutAttributes类，这个类是对item布局的具体设置，以后咱们在讨论这个类。总之，FlowLayout类将每个item的位置等布局信息放在一个数组中，在collectionView布局时，会调用FlowLayout类layoutAttributesForElementsInRect：方法来获取这个布局配置数组。因此，我们需要重写这个方法，返回我们自定义的配置数组，另外，FlowLayout类在进行布局之前，会调用prepareLayout方法，所以我们可以重写这个方法，在里面对我们的自定义配置数据进行一些设置。

简单来说，自定义一个FlowLayout布局类就是两个步骤：

1、设计好我们的布局配置数据 prepareLayout方法中

2、返回我们的配置数组 layoutAttributesForElementsInRect方法中

示例代码如下：

```
@implementation MyLayout
{
    //这个数组就是我们自定义的布局配置数组
    NSMutableArray * _attributeAttay;
}
//数组的相关设置在这个方法中
//布局前的准备会调用这个方法
-(void)prepareLayout{
    _attributeAttay = [[NSMutableArray alloc]init];
    [super prepareLayout];
    //演示方便 我们设置为静态的2列
    //计算每一个item的宽度
    float WIDTH = ([UIScreen mainScreen].bounds.size.width-self.sectionInset.left-self.sectionInset.right-self.minimumInteritemSpacing)/2;
    //定义数组保存每一列的高度
    //这个数组的主要作用是保存每一列的总高度，这样在布局时，我们可以始终将下一个Item放在最短的列下面
    CGFloat colHight[2]={self.sectionInset.top,self.sectionInset.bottom};
    //itemCount是外界传进来的item的个数 遍历来设置每一个item的布局
    for (int i=0; i<_itemCount; i++) {
        //设置每个item的位置等相关属性
        NSIndexPath *index = [NSIndexPath indexPathForItem:i inSection:0];
        //创建一个布局属性类，通过indexPath来创建
        UICollectionViewLayoutAttributes * attris = [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:index];
        //随机一个高度 在40——190之间
        CGFloat hight = arc4random()%150+40;
        //哪一列高度小 则放到那一列下面
        //标记最短的列
        int width=0;
        if (colHight[0]<colHight[1]) {
            //将新的item高度加入到短的一列
            colHight[0] = colHight[0]+hight+self.minimumLineSpacing;
            width=0;
        }else{
            colHight[1] = colHight[1]+hight+self.minimumLineSpacing;
            width=1;
        }
        
        //设置item的位置
        attris.frame = CGRectMake(self.sectionInset.left+(self.minimumInteritemSpacing+WIDTH)*width, colHight[width]-hight-self.minimumLineSpacing, WIDTH, hight);
        [_attributeAttay addObject:attris];
    }
    
    //设置itemSize来确保滑动范围的正确 这里是通过将所有的item高度平均化，计算出来的(以最高的列位标准)
    if (colHight[0]>colHight[1]) {
        self.itemSize = CGSizeMake(WIDTH, (colHight[0]-self.sectionInset.top)*2/_itemCount-self.minimumLineSpacing);
    }else{
          self.itemSize = CGSizeMake(WIDTH, (colHight[1]-self.sectionInset.top)*2/_itemCount-self.minimumLineSpacing);
    }
    
}
//这个方法中返回我们的布局数组
-(NSArray<UICollectionViewLayoutAttributes *> *)layoutAttributesForElementsInRect:(CGRect)rect{
    return _attributeAttay;
}


@end
```

自定义完成FlowLayout后，我们在ViewController中进行使用：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    MyLayout * layout = [[MyLayout alloc]init];
    layout.scrollDirection = UICollectionViewScrollDirectionVertical;
    layout.itemCount=100;
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
    return 100;
}
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewCell * cell  = [collectionView dequeueReusableCellWithReuseIdentifier:@"cellid" forIndexPath:indexPath];
    cell.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    return cell;
}
```

运行效果就是我们引言中的截图。

### 三、UICollectionViewLayoutAttributes类中我们可以配置的属性

        通过上面的例子，我们可以了解，collectionView的item布局其实是LayoutAttributes类具体配置的，这个类可以配置的布局属性不止是frame这么简单，其中还有许多属性：

```
//配置item的布局位置
@property (nonatomic) CGRect frame;
//配置item的中心
@property (nonatomic) CGPoint center;
//配置item的尺寸
@property (nonatomic) CGSize size;
//配置item的3D效果
@property (nonatomic) CATransform3D transform3D;
//配置item的bounds
@property (nonatomic) CGRect bounds NS_AVAILABLE_IOS(7_0);
//配置item的旋转
@property (nonatomic) CGAffineTransform transform NS_AVAILABLE_IOS(7_0);
//配置item的alpha
@property (nonatomic) CGFloat alpha;
//配置item的z坐标
@property (nonatomic) NSInteger zIndex; // default is 0
//配置item的隐藏
@property (nonatomic, getter=isHidden) BOOL hidden; 
//item的indexpath
@property (nonatomic, strong) NSIndexPath *indexPath;
//获取item的类型
@property (nonatomic, readonly) UICollectionElementCategory representedElementCategory;
@property (nonatomic, readonly, nullable) NSString *representedElementKind; 

//一些创建方法
+ (instancetype)layoutAttributesForCellWithIndexPath:(NSIndexPath *)indexPath;
+ (instancetype)layoutAttributesForSupplementaryViewOfKind:(NSString *)elementKind withIndexPath:(NSIndexPath *)indexPath;
+ (instancetype)layoutAttributesForDecorationViewOfKind:(NSString *)decorationViewKind withIndexPath:(NSIndexPath *)indexPath;
```

通过上面的属性，可以布局出各式各样的炫酷效果，正如一句话：没有做不到，只有想不到。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

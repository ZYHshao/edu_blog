---
title: iOS第三方左对齐布局类——UICollectionViewLeftAlignedLayout
date: 2016-07-05
categories: iOS第三方库
tags: []
---
## iOS第三方左对齐布局类——UICollectionViewLeftAlignedLayout

        UICollectionViewLeftAlignedLayout是第三方的左对齐布局管理类，其继承自UICollectionViewFlowLayout，使用其可以方便的进行左对齐的瀑布流界面布局。

        UICollectionViewLeftAlignedLayout的git地址如下：[https://github.com/mokagio/UICollectionViewLeftAlignedLayout](https://github.com/mokagio/UICollectionViewLeftAlignedLayout)。

        使用示例如下：

```objectivec
#import <UICollectionViewLeftAlignedLayout.h>
@interface ViewController ()<UICollectionViewDataSource,UICollectionViewDelegateFlowLayout>
@property(nonatomic,strong)UICollectionView * collectionView;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    UICollectionViewLeftAlignedLayout* layout = [[UICollectionViewLeftAlignedLayout alloc]init];
    self.collectionView = [[UICollectionView alloc]initWithFrame:self.view.frame collectionViewLayout:layout];
    self.collectionView.dataSource = self;
    self.collectionView.delegate=self;
    [self.collectionView registerClass:[UICollectionViewCell class] forCellWithReuseIdentifier:@"cellID"];
    [self.view addSubview:self.collectionView];
}

-(CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout *)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath{
    return CGSizeMake(arc4random()%100+50, 100);
}

-(NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView{
    return 1;
}
-(NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section{
    return 10;
}
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewCell * cell = [collectionView dequeueReusableCellWithReuseIdentifier:@"cellID" forIndexPath:indexPath];
    cell.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    return cell;
}
@end
```

效果如下：

![](http://static.oschina.net/uploads/space/2016/0705/102934_z96g_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

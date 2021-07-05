---
title: iOS自定义的emoji表情键盘
date: 2015-11-11
categories: iOS之UI控件
tags: []
---
## iOS自定义的表情键盘

### 一、关于emoji表情

        随着iOS系统版本的升级，对原生emoji表情的支持也越来越丰富。emoji表情是unicode码中为表情符号设计的一组编码，当然，还有独立于unicode的另一套编码SBUnicode，在OS系统中，这两种编码都有很好的支持。UI系统会自动帮我们将编码转义成表情符号，例如用SBUnicode如下代码：

```
  UILabel * label = [[UILabel alloc]initWithFrame:CGRectMake(100, 100, 100, 100)];
    label.font = [UIFont systemFontOfSize:25];
    label.text = @"\uE056";
    [self.view addSubview:label];
```

就会在屏幕上出现一个笑脸：

![](http://static.oschina.net/uploads/space/2015/1111/183536_3L3Q_2340880.png)      

### 二、开发表情键盘的思路

        首先为了实现跨平台，无论iOS端，andorid端还是web端，都要有一个相同的标准，这个标准就可以是国际Unicode编码，我们的思路是将表情文字进行unicode编码后再进行传输，因此，有两中方式，一种是通过自定义一套表情切图，将其与unicode码一一对应，在转码的时候，我们一一遍历，转换成unicode后进行传输，这样的好处是我们可以保证所有平台所能使用的表情统一。在iOS端，可以有另一种方式，通过上面我们知道，通过SBUnicode码我们可以在客户端显示表情符号，并且这个码的排列是十分有规律的，通过这个特点，我们可以通过遍历SBUnicode码的范围进行表情的创建，省去的图片素材的麻烦。

        iOS中可用的表情unicode范围是：0xE001~0xE05A,0xE101~0xE15A,

0xE201~0xE253,0xE401~0xE44C,0xE501~0xE537。

        我们可以通过遍历的方法，将其都加入数据源数组中：

```
int emojiRangeArray[10] = {0xE001,0xE05A,0xE101,0xE15A,0xE201,0xE253,0xE401,0xE44C,0xE501,0xE537};
    for (int j = 0 ; j<10 ; j+=2 ) {
        
        int startIndex = emojiRangeArray[j];
        int endIndex = emojiRangeArray[j+1];
        
        for (int i = startIndex ; i<= endIndex ; i++ ) {
        //添加到数据源数组
            [dataArray addObject:[NSString stringWithFormat:@"%C", (unichar)i]];
        }
    }
```

键盘的摆放，可以通过collectionView来做，十分方便：

```
    //为了摆放分页控制器，创建一个背景view
    bgView = [[UIView alloc]initWithFrame:CGRectMake(0, 0, [UIScreen mainScreen].bounds.size.width, 200)];
    //分页控制器
    pageControlBottom = [[UIPageControl alloc]initWithFrame:CGRectMake(0, 170, [UIScreen mainScreen].bounds.size.width, 20)];
    [bgView addSubview:pageControlBottom];
    //collectionView布局
    UICollectionViewFlowLayout * layout = [[UICollectionViewFlowLayout alloc]init];
    //水平布局
    layout.scrollDirection=UICollectionViewScrollDirectionHorizontal;
    //设置每个表情按钮的大小为30*30
    layout.itemSize=CGSizeMake(30, 30);
    //计算每个分区的左右边距
    float xOffset = (kscreenWidth-7*30-10*6)/2;
    //设置分区的内容偏移
    layout.sectionInset=UIEdgeInsetsMake(10, xOffset, 10, xOffset);
    scrollView = [[UICollectionView alloc]initWithFrame:CGRectMake(0, 0, [UIScreen mainScreen].bounds.size.width, 160) collectionViewLayout:layout];
    //打开分页效果
    scrollView.pagingEnabled = YES;
    //设置行列间距
    layout.minimumLineSpacing=10;
    layout.minimumInteritemSpacing=5;
    
    scrollView.delegate=self;
    scrollView.dataSource=self;
    scrollView.backgroundColor = bgView.backgroundColor;
    [bgView addSubview:scrollView];
```

在collectionView的回调方法中，处理如下：

```
//每页28个表情
-(NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section{
    if (((dataArray.count/28)+(dataArray.count%28==0?0:1))!=section+1) {
         return 28;
    }else{
        return dataArray.count-28*((dataArray.count/28)+(dataArray.count%28==0?0:1)-1);
    }
   
}
//返回页数
-(NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView{
    return (dataArray.count/28)+(dataArray.count%28==0?0:1);
}
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewCell * cell = [collectionView dequeueReusableCellWithReuseIdentifier:@"biaoqing" forIndexPath:indexPath];
    for (int i=cell.contentView.subviews.count; i>0; i--) {
        [cell.contentView.subviews[i-1] removeFromSuperview];
    }
    UILabel * label = [[UILabel alloc]initWithFrame:CGRectMake(0, 0, 30, 30)];
    label.font = [UIFont systemFontOfSize:25];
    label.text =dataArray[indexPath.row+indexPath.section*28] ;
   
    
    [cell.contentView addSubview:label];
    return cell;
}
-(void)collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath{
    NSString * str = dataArray[indexPath.section*28+indexPath.row];
    //这里手动将表情符号添加到textField上
    
}
//翻页后对分页控制器进行更新
-(void)scrollViewDidScroll:(UIScrollView *)scrollView{
    CGFloat contenOffset = scrollView.contentOffset.x;
    int page = contenOffset/scrollView.frame.size.width+((int)contenOffset%(int)scrollView.frame.size.width==0?0:1);
    pageControlBottom.currentPage = page;

}
```

### 三、切换系统键盘和自定义的表情键盘

        UITextField和UITextView都会有下面这个属性和方法：

```
@property (nullable, readwrite, strong) UIView *inputView;   
- (void)reloadInputViews;
```

inputView我们可以设置textView和textField成为第一响应时的弹出附件，如果我们不设置或者设置为nil，则会弹出系统键盘，reloadInputView方法可以使我们刷新这个附件视图，通过这两个，我们可以非常轻松的实现键盘的切换，比如我们在一个出发方法中如下处理：

```
-(void)imageViewTap{
    if (![_publishContent isFirstResponder]) {
        return;
    }
    if (isEmoji==NO) {
        isEmoji=YES;
        //呼出表情
        _textView.inputView=bgView;
        [_textView reloadInputViews];
    }else{
        isEmoji=NO;
        _textView.inputView=nil;
        [_textView reloadInputViews];
    }

    
}
```

效果如下：

![](http://static.oschina.net/uploads/space/2015/1111/190433_4hT0_2340880.png)           ![](http://static.oschina.net/uploads/space/2015/1111/190433_R7eF_2340880.png)

  
**追注：测试上面的SBUnicode码在模拟器上可以正常显示，真机并不能识别，可以通过将表情符全部添加到一个plist文件中，通过文件读取来创建键盘的方式进行真机上的开发。plist文件地址如下：**

[**http://pan.baidu.com/s/1o6AdkBw**](http://pan.baidu.com/s/1o6AdkBw)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

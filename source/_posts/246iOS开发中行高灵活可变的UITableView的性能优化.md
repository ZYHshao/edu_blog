---
title: iOS开发中行高灵活可变的UITableView的性能优化
date: 2016-08-27
categories: iOS逻辑初窥
tags: []
---
## iOS开发中行高灵活可变的UITableView的性能优化

### 一、UITableView的构建原理

        在新闻类，电商类等应用中，应用着大量的图文混排视图，在表视图UITableView中，开发者通常需要在如下代理方法中计算出当前cell填充内容后的高度，之后将其返回：

```objectivec
-(CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath{
    //先根据数据源中数据计算高度
    CGFloat height = 0;
    return height;
}
```

然而，如果在如上方法中进行打印调试可以发现，heightForRowAtIndexPath方法会重复执行好多次，首先，并且heightForRowAtIndexPath方法的执行机制在不同版本的iOS系统还会有很大不同。以iOS9为例，一行cell要展示在屏幕上，至少要执行5遍TableView的heightForRowAtIndexPath方法：

TableView配置部分：

① 当TableView视图即将展现在屏幕上时，会把所有行的行高数据进行拉取。

![](http://static.oschina.net/uploads/space/2016/0827/120931_xUxT_2340880.png)

②当TableView在执行setLayoutMargins方法进行自身布局时会把所有行高数据进行拉取。

![](http://static.oschina.net/uploads/space/2016/0827/121136_l43D_2340880.png)

③TableView在执行layoutSubViews方法进行子视图布局时会再次把所有行高数据进行拉取。

![](http://static.oschina.net/uploads/space/2016/0827/121401_FLHn_2340880.png)

TableViewCell配置部分：

④当使用cellID进行与TableView绑定的cell获取时会拉取本行cell的高度数据。

![](http://static.oschina.net/uploads/space/2016/0827/121807_YYg4_2340880.png)

⑤当cell进行layoutSubViews方法进行布局时会再次拉取本行cell的高度数据。

![](http://static.oschina.net/uploads/space/2016/0827/121911_bc1i_2340880.png)

上面列举的5中拉取cell高度的场景中，TableView配置部分只会在TableView第一次展现在屏幕上时出现，但是其拉取的是所有行的行高数据，如果表视图有100行或者更多，这将是一个十分耗费性能的过程。TableViewCell配置部分，只有当cell将要出现在屏幕上时才会出现，并且只拉取当前行的行高，这两种场景会在用户滑动TableView时不断被执行，并且根据UITableView的布局cell原理，系统会默认准备当前一屏高度所能容纳cell个数加1个cell。

        当执行TableView的reloadData方法进行界面刷新时，系统先会把所有行的行高数据拉取一遍，之后和UITableViewCell配置部分的场景一直，会拉取即将出现在屏幕上的cell的行高数据。

用示意图形象的表示上述逻辑如下：

![](http://static.oschina.net/uploads/space/2016/0827/120009_ZJyg_2340880.png)

通过上面分析，以10行数据的表格视图为例，若一屏幕可以呈现7行数据(TableView需要准备8行)，则在第一次展示TableView视图时，会执行44次heightForRwoAtIndexPath方法，每次刷新TableView需要执行24次heightForRwoAtIndexPath方法，如果TableView的行数增加到3位数，则这个方法的执行次数将会十分恐怖👿。

_至于为何UITableView在进行配置时也需要拉取所有的行高数据，我猜想其为了进行视图的一些初始化操作，例如表视图右侧滚动条的宽度和所占比例等。并且，每次拉取高度都从代理方法拉取，而不是存入内部的一个变量属性中，避免了因为数据源更改时机巧合而产生的界面与预期不一致的风险。_

### 二、对UITableView可变行高的计算方式进行优化

        通过前面的分析，可以理解如果将复杂的计算代码写在heightForRowAtIndexPath方法中，代价将是非常惨重的。滑动不流畅，屏幕卡顿很多性能问题都是由于这个原因。对于行高固定的表格视图，开发者可以直接设置TableView的固定行高，如下：

```objectivec
_tableView.rowHeight = 200;
```

如果行高是不固定了，则应该想办法让heightForRowAtIndexPath方法完成最少的工作，其实最少的工作莫过于拿过一个高度，直接返回，因此开发者通常会将对应行的行高计算一次后，把值进行保存，之后在执行heightForRowAtIndexPath方法拉取行高时，直接返回已经计算过的行高数据，具体如何操作比较灵活，可以对应一个数组属性，将计算后的行高放入数组中，每次取行高时，检查数组中是否已经有计算过的行高数据，如果有直接返回。我个人更倾向将行高数据封装进cell的数据模型Model中。

        通过优化，可以有效的减少重复的高度计算，这也是我原先处理此类问题的主要方式。然而，只是提高了代码的性能，对开发者来说，工作量和复杂度有增而无减。在开发中通常会遇到一些十分复杂的界面，而这些界面中cell的高度都是需要通过请求到的数据动态改变的，每个cell都要写复杂的尺寸计算代码十分令人心烦。在iOS7之后，系统提供了一种自动计算cell高度的方法，这无论在性能还是工作量上，都完全解放了开发者。

        在iOS7系统之后，UITableView类中增加了一个estimatedRowHeight属性，顾名思义，这个属性是设置UITableViewCell中的大约行高值。这个值设置之后，开发者无需设置rowHeight属性，也不需要实现heightForRowAtIndexPath方法，系统会自动根据UITableViewCell中contentView的约束来计算自己的行高。estimatedRowHeight属性用于TableView进行初始化，其会影响到表格视图右侧滚动条的宽度。cell展现出来时真正的行高并不受这个属性值的影响。

        那么现在问题来了，如何才能让cell正确计算自己的高度，这就要使用到Autolayout了，无论是通过xib文件创建的cell还是代码创建的cell，若想让cell自动正确的计算出自身的高度，必须添加足够压力的约束。所谓足够压力，是指UITableViewCell的contentView的上、下、左、右必须被内部控件的约束所撑满，需要注意，cell上的视图必须添加在contentView上，否则计算会出现问题。

        例如下图所示，左侧的图标进行了与父视图的左侧距离约束，标题Label进行了与父视图的上侧距离约束和右侧距离约束，内容Label进行了与标题Label的上侧约束和与父视图的下册约束，并且对宽度进行了约束。此时，UITableViewCell的contentView四周都被子视图进行了约束，可以想象，内容Label的文本长度是不定的，当文本长度是的内容Label进行换行，内容Label的高度改变的时候，contentView下册会受到内容Label施加的压力，这时cell也会根据约束自动扩充自己的高度。

![](http://static.oschina.net/uploads/space/2016/0827/133018_SPfQ_2340880.png)

示例代码如下：

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    self.title = @"表视图";
    _tableView = [[UITableView alloc]initWithFrame:self.view.frame style:UITableViewStylePlain];
    [_tableView registerNib:[UINib nibWithNibName:@"TableViewCell" bundle:nil] forCellReuseIdentifier:@"cellid"];
    _tableView.delegate = self;
    _tableView.dataSource = self;
    //设置一个模糊的行高用于配置TableView右侧滚动条
    _tableView.estimatedRowHeight = 60;
    [self.view addSubview:_tableView];
    titleArray = @[@"标题1",@"标题2",@"标题3",@"标题4",@"标题5",@"标题6",@"标题7",@"标题8",@"标题9",@"标题10"];
    detailArray = @[@"内容内容内容内容内容内容内容内容内容",
                    @"内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容",
                    @"内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容",
                    @"内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容",
                    @"内容内容容内容内内容内容内容内容内容内容内容",
                    @"容内容内内容内容",
                    @"内容内容内容内容容内容内容内容",
                    @"内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容",
                    @"内容内容内容内容内容内容容内容内内容内容内容",
                    @"内容内容内容内容内容内容内容内容内容"];
}
-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
    return 10;
}


-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    TableViewCell * cell = [tableView dequeueReusableCellWithIdentifier:@"cellid" forIndexPath:indexPath];
    cell.title.text = titleArray[indexPath.row];
    cell.detail.text = detailArray[indexPath.row];
    return cell;
}
```

![](http://static.oschina.net/uploads/space/2016/0827/133935_BJ9v_2340880.png)

通过上面示例可以看到，十分简单的代码完美的解决了图文混排cell高度的自适应。Autolayout真的是一种十分强大的技术😄。

        关于细节方面，还有一个问题需要注意，预估的行高会影响到TableView右侧滚动条的展现，如果每个cell行高跳跃跨度十分大，滚动条宽度的配置会失准，随着用户滑动表视图，右侧滚动条可能会出现长短跳跃的情况，如果开发者需要精准这个滚动条的配置，可以在如下代理方法中返回具体cell的估计行高。

```objectivec
-(CGFloat)tableView:(UITableView *)tableView estimatedHeightForRowAtIndexPath:(NSIndexPath *)indexPath{
    //这里根据不同分区 或者不同行 设置估计的行高
    return 44;
}
```

关于estimatedHeightForRowAtIndexPath方法其实还有一种应用场景，前面介绍的优化方式都是以Autolyout为前提，对于没有使用自动布局，cell的高度需要手动计算的场景中，如果实现了这个方法，并且实现了heightForRowAtIndexPath方法，heightForRowAtIndexPath方法会以懒加载的方式执行，只有在cell将要展现在屏幕上时heightForRowAtIndexPath方法才会被执行，这也可以有效减小由于高度计算带来的性能负担。

### 三、关于高度不定的UITableView分区头尾视图

        一般情况下，TableView的分区头尾视图高度都是固定的，因此一般不需要考虑计算分区头尾视图高度产生的性能问题，类比如cell的布局原理，其实分区头尾视图也可以通过Autolayout实现自适应高度，示例代码如下：

```objectivec
//返回一个估计的分区头视图高度
-(CGFloat)tableView:(UITableView *)tableView estimatedHeightForHeaderInSection:(NSInteger)section{
    return 10;
}
//使用自动布局给头视图添加足够的布局压力
-(UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section{
    UIView * view = [[UIView alloc]init];
    UILabel * label = [[UILabel alloc]init];
    label.numberOfLines = 0;
    if (section==0) {
          label.text = @"头视图头视图头视图";
    }else{
        label.text = @"头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图头视图";
    }
  
    [view addSubview:label];
    [label mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(@10);
        make.right.equalTo(@-10);
        make.top.equalTo(@10);
        make.bottom.equalTo(@-10);
    }];
    return view;
}

```

效果如下图：

![](http://static.oschina.net/uploads/space/2016/0827/141128_9lYu_2340880.png)

分区为视图的设置方式与头视图一样。

        UITableView类中还有一个十分有趣的常量：

```objectivec
UIKIT_EXTERN const CGFloat UITableViewAutomaticDimension;
```

UITableViewAutomaticDimension是一个CGFloat类型的常量，其需要和用来处理返回头尾视图标题的方法结合使用，用它来作为TableView分区头尾视图的高度返回，系统会自动根据标题是否存在来进行自适应，举个例子，如果返回的标题为nil，则头视图会被自动隐藏，示例代码如下：

```objectivec

-(CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section{
    //视图为nil则会自动返回0
    return UITableViewAutomaticDimension;
}


-(NSString*)tableView:(UITableView *)tableView titleForHeaderInSection:(NSInteger)section{
    if (section==0) {
        return nil;
    }else{
        return @"头视图头视图头视图头视图头视图头视图头视图头视";
    }
}

```

小提示：UITableViewCell在创建出来时，其宽度并不一定和UITableView宽度一致，如果开发者需要通过获取cell的宽度来处理逻辑，要在cell的layoutSubViews里面进行，此时cell的宽度才正确。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

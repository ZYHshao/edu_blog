---
title: iOS中CoreData数据管理系列四——进行数据与页面的绑定
date: 2016-02-01
categories: iOS逻辑初窥
tags: []
---
## iOS中CoreData数据管理系列四——进行数据与页面的绑定

### 一、引言

    在上一篇博客中，我们讨论了CoreData框架中添加与查询数据的操作，事实上，在大多数情况下，这些数据都是由一个UITableView表视图进行展示的，因此，CoreData框架中还未开发者提供了一个类NSFetchedResultsController，这个类作为桥接，将视图与数据进行绑定。

添加与查询数据操作：[http://my.oschina.net/u/2340880/blog/611430](http://my.oschina.net/u/2340880/blog/611430)。

### 二、进行数据初始化

    NSFetchedResultsController的初始化需要一个查询请求和一个数据操作上下文。代码示例如下：

```
//遵守协议
@interface ViewController ()<NSFetchedResultsControllerDelegate>
{
    //数据桥接对象
    NSFetchedResultsController * _fecCon;
}
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    //进行初始化操作
    NSURL *modelUrl = [[NSBundle mainBundle]URLForResource:@"Model" withExtension:@"momd"];
    NSManagedObjectModel * mom = [[NSManagedObjectModel alloc]initWithContentsOfURL:modelUrl];
    NSPersistentStoreCoordinator * psc = [[NSPersistentStoreCoordinator alloc]initWithManagedObjectModel:mom];
    NSURL * path =[NSURL fileURLWithPath:[[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)lastObject] stringByAppendingPathComponent:@"CoreDataExample.sqlite"]];
    [psc addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:path options:nil error:nil];
    NSManagedObjectContext * moc = [[NSManagedObjectContext alloc]initWithConcurrencyType:NSMainQueueConcurrencyType];
    [moc setPersistentStoreCoordinator:psc];
    NSFetchRequest * request = [NSFetchRequest fetchRequestWithEntityName:@"SchoolClass"];
    //设置数据排序
    [request setSortDescriptors:@[[NSSortDescriptor sortDescriptorWithKey:@"stuNum" ascending:YES]]];
    //进行数据桥接对象的初始化
    _fecCon = [[NSFetchedResultsController alloc]initWithFetchRequest:request managedObjectContext:moc sectionNameKeyPath:nil cacheName:nil];
    //设置代理
    _fecCon.delegate=self;
    //进行数据查询
    [_fecCon performFetch:nil];
}
@end
```

用于初始化NSFecthedResultsController的数据请求对象必须设置一个排序规则。在initWithFetchRequest:managedObjectContext:sectionNameKeyPath:cacheName:方法中，如果设置第三个参数，则会以第三个参数为键值进行数据的分区。当数据发生变化时，将通过代理进行方法的回调。

### 三、与UITableView进行数据绑定 

```
-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    UITableViewCell * cell = [tableView dequeueReusableCellWithIdentifier:@"cellid"];
    if (!cell) {
        cell = [[UITableViewCell alloc]initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:@"cellid"];
    }
    //获取相应数据模型
    SchoolClass * obj = [_fecCon objectAtIndexPath:indexPath];
    cell.textLabel.text = obj.name;
    cell.detailTextLabel.text = [NSString stringWithFormat:@"有%@人",obj.stuNum];
    return cell;
}
-(NSInteger)numberOfSectionsInTableView:(UITableView *)tableView{
    return [_fecCon sections].count;
}
-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
    id<NSFetchedResultsSectionInfo> info =  [_fecCon sections][section];
    return [info numberOfObjects];
    
}
```

效果如下：

![](http://static.oschina.net/uploads/space/2016/0201/134121_hNPW_2340880.png)

### 四、将数据变化映射到视图

```
//数据将要改变时调用的方法
- (void)controllerWillChangeContent:(NSFetchedResultsController *)controller
{
    //开启tableView更新预处理
    [[self tableView] beginUpdates];
}
//分区数据改变时调用的方法
- (void)controller:(NSFetchedResultsController *)controller didChangeSection:(id <NSFetchedResultsSectionInfo>)sectionInfo atIndex:(NSUInteger)sectionIndex forChangeType:(NSFetchedResultsChangeType)type
{
    //判断行为类型
    switch(type) {
        //插入新分区
        case NSFetchedResultsChangeInsert:
            [[self tableView] insertSections:[NSIndexSet indexSetWithIndex:sectionIndex] withRowAnimation:UITableViewRowAnimationFade];
            break;
        //删除分区
        case NSFetchedResultsChangeDelete:
            [[self tableView] deleteSections:[NSIndexSet indexSetWithIndex:sectionIndex] withRowAnimation:UITableViewRowAnimationFade];
            break;
        //移动分区
        case NSFetchedResultsChangeMove:
        //更新分区
        case NSFetchedResultsChangeUpdate:
            break;
    }
}
//数据改变时回调的代理
- (void)controller:(NSFetchedResultsController *)controller didChangeObject:(id)anObject atIndexPath:(NSIndexPath *)indexPath forChangeType:(NSFetchedResultsChangeType)type newIndexPath:(NSIndexPath *)newIndexPath
{
    switch(type) {
        //插入数据
        case NSFetchedResultsChangeInsert:
            [[self tableView] insertRowsAtIndexPaths:@[newIndexPath] withRowAnimation:UITableViewRowAnimationFade];
            break;
        //删除数据
        case NSFetchedResultsChangeDelete:
            [[self tableView] deleteRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationFade];
            break;
        //更新数据
        case NSFetchedResultsChangeUpdate:
            [self reloadData];
            break;
        //移动数据
        case NSFetchedResultsChangeMove:
            [[self tableView] deleteRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationFade];
            [[self tableView] insertRowsAtIndexPaths:@[newIndexPath] withRowAnimation:UITableViewRowAnimationFade];
            break;
    }
}
//数据更新结束调用的代理
- (void)controllerDidChangeContent:(NSFetchedResultsController *)controller
{
    [[self tableView] endUpdates];
}
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

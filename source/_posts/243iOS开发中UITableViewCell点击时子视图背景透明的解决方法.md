---
title: iOS开发中UITableViewCell点击时子视图背景透明的解决方法
date: 2016-08-10
categories: 日常技巧
tags: []
---
## iOS开发中UITableViewCell点击时子视图背景透明的解决方法

        在做iOS项目的开发中，UITableView控件的应用十分广泛。在进行自定义UITableViewCell时，经常有小伙伴遇到这样的问题：在UITableViewCell上面添加了一个有背景颜色的子视图，当用户点击UITableViewCell或者选中UITableViewCell时，Cell上的子视图发生了奇怪的变化，其背景色变透明了，如果添加在Cell上的子视图只是一个色块，那么我们看起来，这个子视图好像莫名其妙的消失了一样。如下图所示：

![](http://static.oschina.net/uploads/space/2016/0810/142921_gC8o_2340880.png)

    产生这种情况的主要原因是由于UITableViewCell的选中风格所致。如果开发者不进行设置，UITableViewCell中的selectionStyle属性默认风格为UITableViewCellSelectionStyleBlue。这时，如果用户点击或者选中了某个Cell，系统会自动将其上子视图的背景色改成透明以便统一Cell的整体背景颜色。开发者可以将其设置为UITableViewCellSelectionStyleNone枚举值来不适用任何Cell的选中风格。

    如果需要使用Cell的选中风格同时又不想让Cell上的子视图收到影响，我们可以继承UITableViewCell后在其中覆写父类的如下两个方法，在这些方法中重新设置子视图的背景色：

```objectivec
//这个方法在Cell被选中或者被取消选中时调用
- (void)setSelected:(BOOL)selected animated:(BOOL)animated {
    [super setSelected:selected animated:animated];
    self.testLabel.backgroundColor = [UIColor orangeColor];
}
//这个方法在用户按住Cell时被调用
-(void)setHighlighted:(BOOL)highlighted animated:(BOOL)animated{
    [super setHighlighted:highlighted animated:animated];
    self.testLabel.backgroundColor = [UIColor orangeColor];
}
```

如下图：

![](http://static.oschina.net/uploads/space/2016/0810/144116_Ll58_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

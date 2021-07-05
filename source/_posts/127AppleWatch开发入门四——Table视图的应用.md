---
title: AppleWatch开发入门四——Table视图的应用
date: 2015-10-15
categories: Apple Watch开发手记
tags: []
---
## AppleWatch开发入门四——Table视图的应用

### 一、Watch上的Table

        WatchOS中的TableView和iOS中的TableView还是有很大的区别，在开发之前，首先我们应该明白WatchOS中的Table有哪些局限性和特点。下面几点是我总结WatchOS中Table的特殊之处：

1、Table只有行的概念，没有分区的概念，没有头尾视图的概念。

2、可以通过创建多个Table，来实现分区的效果。

3、因为Watch上是通过Gruop进行布局适应的，所以没有行高等设置。

4、Table没有代理，所有行的数据都是采用静态配置的方式，后面会介绍。

5、点击Table中的行触发的方法，是通过重写Interface中的方法来实现的。

### 二、创建一个Table

        在storyBoard中拖入你的Table，如下：

![](http://static.oschina.net/uploads/space/2015/1015/100504_0nPS_2340880.png)        

在Table上拉两个label：

![](http://static.oschina.net/uploads/space/2015/1015/101019_GMa7_2340880.png)

每一个Table中包含一个TableRowController，实际上我们Table上的控件都是通过这个TableRowController进行管理的，因此如果我们需要在代码中控制TableRow上的内容，我们需要创建一个文件作为Table的TableRowController：

![](http://static.oschina.net/uploads/space/2015/1015/101407_OVNY_2340880.png)

将storyBoard中TableRowController的类修改为我们创建的类并指定一个identifier：

![](http://static.oschina.net/uploads/space/2015/1015/101551_qOiw_2340880.png)                 ![](http://static.oschina.net/uploads/space/2015/1015/101553_8dpz_2340880.png)

![](http://static.oschina.net/uploads/space/2015/1015/104057_IXLw_2340880.png)

然后，我们将两个label关联到TableRowController中：

```
import WatchKit
class TableRowController: NSObject {

    @IBOutlet var numberLabel: WKInterfaceLabel!
    @IBOutlet var titleLabel: WKInterfaceLabel!
}
```

将Table关联到interfaceController中：

```
class InterfaceControllerMain: WKInterfaceController {
    
    @IBOutlet var Table: WKInterfaceTable!

}
```

下面，我们开始在interface中对Table做相关配置，首先我们可以先观察一下WKInterfaceTable中有哪些方法和属性：

```
public class WKInterfaceTable : WKInterfaceObject {
    //设置行的类型，数组中对应存放行的类型，数组元素的个数，就是行数
    /*
    通过这个方法，我们可以创建每一行样式都不同的table，行的类型
    实际上就是我们刚才用到的TableRowController，我们可以进行自定义
    */
    public func setRowTypes(rowTypes: [String]) 
    //设置行数和类型 用于创建单一行类型的table
    public func setNumberOfRows(numberOfRows: Int, withRowType rowType: String) // repeating row name
    //这个get方法获取行数，用于我们遍历table中的行，进行内容设置
    public var numberOfRows: Int { get }
    //这个方法会返回某一行，我们可以获取到后进行内容设置
    public func rowControllerAtIndex(index: Int) -> AnyObject?
    //插入一行
    public func insertRowsAtIndexes(rows: NSIndexSet, withRowType rowType: String)
    //删除一行
    public func removeRowsAtIndexes(rows: NSIndexSet)
    //滑动到某一行
    public func scrollToRowAtIndex(index: Int)
}
```

了解了上面的方法，可以看出，WatchOS的Table配置非常简单易用，例如我们如下配置：

```
@IBOutlet var Table: WKInterfaceTable!

    override func awakeWithContext(context: AnyObject?) {
        super.awakeWithContext(context)
        let dic:Dictionary<String,String> = ["中国建设银行":"￥1000","中国农业银行":"￥5000","中国银行":"20000","招商银行":"￥401","中国邮政储蓄":"1100"]
        //设置行数与类型
        Table.setNumberOfRows(dic.count, withRowType: "TableRowController")
        //遍历进行设置
        let titleArray:Array<String> = Array(dic.keys)
        for var i=0 ; i < dic.count ; i++ {
            let row:TableRowController = Table.rowControllerAtIndex(i) as! TableRowController
            row.titleLabel.setText(titleArray[i])
            row.numberLabel.setText(dic[titleArray[i]])
            row.numberLabel.setTextColor(UIColor.grayColor())
        }
        // Configure interface objects here.
    }
```

这样一个展示银行卡余额的界面我们就创建完成了，效果如下：

![](http://static.oschina.net/uploads/space/2015/1015/105932_5XtV_2340880.png)

### 三、关于Table的点击事件

        上面我们提到，Table没有所谓代理方法，点击row的时候，我们也是通过两种方式进行逻辑跳转的，一种是在storyBoard中，我们通过拉线跳转，这时如需传值，我们需在interface中实现如下方法：

```
 public func contextForSegueWithIdentifier(segueIdentifier: String, inTable table: WKInterfaceTable, rowIndex: Int) -> AnyObject?
```

        另一种方式，我们可以重写实现InterfaceController中的如下方法，来处理Table的点击事件：

```
public func table(table: WKInterfaceTable, didSelectRowAtIndex rowIndex: Int)
```

        无论哪种方式，我们都可以通过参数table和rowIndex来确认点击的具体是那个table和哪一行，进行传值和处理我们的逻辑。             

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

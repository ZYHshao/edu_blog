---
title: iOS:JSON转OC属性小工具能
date: 2018-06-09
categories: 小码工具
tags: []
---
## iOS:JSON转OC属性小工具

      在iOS开发中，只要有网络模块，就需要数据模型的编写。在进行数据模型的解析和映射时，JSONModel是一个非常常用且优秀的第三方框架，之前有有过博客对其分析，地址如下：

JSONModel源码分析：[https://my.oschina.net/u/2340880/blog/1787561](https://my.oschina.net/u/2340880/blog/1787561)。

      无论使用什么第三方的JSON数据解析框架，我们都需要手动来编写数据模型类，这是一个十分机械性的体力活，本篇博客将介绍一个配合与JSONModel使用的自动生成属性脚本(支持类的嵌套)。

      本脚本采用的语言为JavaScript，采用JavaScript编写有两个好处，首先其可以在node环境运行，可以十分方便的操作文件，使用它可以直接将JSON文件转换成OC数据模型类。其次，它也十分容易在Web端运行，可以通过网页可视化的进行数据模型的转换。

      闲话少说，直接上源码：

```javascript
var fileManager = require('fs');
var gettype=Object.prototype.toString;
String.prototype.firstUpperCase = function(){
    return this.replace(/\b(\w)(\w*)/g, function($0, $1, $2) {
        return $1.toUpperCase() + $2.toLowerCase();
    });
}
var arguments = process.argv.splice(2);
var path =  arguments[0];
if (!path) {
    console.log("请传入要转换的JSON文件路径");
    return;
}
console.log('json文件路径:', path);
try{
    var result = JSON.parse(fileManager.readFileSync(path));
}catch(error){
    console.log("解析JSON文件失败:"+error);
    return;
}
if (!result) {
    console.log("解析JSON文件无效");
    return;
}
var classArray = new Array();
parseObject("MyObject",result);

var stringAll="";
//输出
for (var i = classArray.length-1; i >=0 ; i--) {
    let cla = classArray[i];
    stringAll+="@protocol "+cla.name+" @end\r\n\r\n@interface "+cla.name+" : JSONModel\r\n\r\n";
    console.log("@protocol "+cla.name+" @end\r\n\r\n@interface "+cla.name+" : JSONModel\r\n\r\n");
    for (var j = 0; j < cla.property.length; j++) {
        stringAll+=cla.property[j]+"\r\n\r\n";
        console.log(cla.property[j]+"\r\n\r\n");
    }
    stringAll+="@end\r\n\r\n";
    console.log("@end\r\n\r\n");
}
stringAll+="===========m文件======================\r\n";
console.log("===========m文件======================\r\n");
for (var i = classArray.length-1; i >=0 ; i--) {
    let cla = classArray[i];
    stringAll+="@implementation "+cla.name+"\r\n\r\n@end\r\n\r\n";
    console.log("@implementation "+cla.name+"\r\n\r\n@end\r\n\r\n");
}
let paths = path.split("/");
paths.pop();
let newPath = paths.join("/")+"/oc.txt";
fileManager.writeFileSync(newPath,stringAll);
//核心解析函数
function parseObject(k,result){
    let c = new Class(k);
    classArray.push(c);
    for (var i = 0; i < Object.getOwnPropertyNames(result).length; i++) {
        let key = Object.getOwnPropertyNames(result)[i];
        let value = result[key];
        let type = getType(value);
        if(type==null){
            continue;
        }
        if (type=="Object") {
            //进行二次解析
            if (Object.getOwnPropertyNames(value).length==0) {
                c.property.push("@property(nonatomic,strong)NSDictionary<Optional>*"+key+";");
            }else{
                parseObject(key.firstUpperCase(),value);
                c.property.push("@property(nonatomic,strong)"+key.firstUpperCase()+"<Optional,"+key.firstUpperCase()+">*"+key+";");
            }
            continue;
        }
        if (type=="Array") {
            if (value.length>0) {
                let obj = value[0];
                let t = getType(obj);
                if (t==null) {
                    continue;
                }
                if (t=="Object") {
                    c.property.push("@property(nonatomic,strong)NSArray<"+key.firstUpperCase()+"*><Optional,"+key.firstUpperCase()+">*"+key+";");
                    parseObject(key.firstUpperCase(),obj);
                }else{
                    c.property.push("@property(nonatomic,strong)NSArray<"+t+"*><Optional>*"+key+";");
                }
            }else{
                c.property.push("@property(nonatomic,strong)NSArray<Optional>*"+key+";");
            }
            continue;
        }
        if (type=="id") {
            c.property.push("@property(nonatomic,strong)"+type+"<Optional>"+key+";");
            continue;
        }
        c.property.push("@property(nonatomic,strong)"+type+"<Optional>*"+key+";");
    }
}
//获取要转换的类型
function getType(obj){
    if (typeof obj == 'number') {
        return "NSNumber";
    }
    if (typeof obj == 'undefined') {
        return "id";
    }
    if (typeof obj == 'null') {
        return "id";
    }
    if (typeof obj == 'function') {
        return null;
    }
    if (typeof obj == 'string') {
        return "NSString"
    }
    if (typeof obj == 'boolean') {
        return "NSNumber"
    }
    if (typeof obj == 'object') {
        if (gettype.call(obj)=="[object Object]") {
            return "Object";
        }
        if (gettype.call(obj)=="[object Array]") {
            return "Array";
        } 
        if (gettype.call(obj)=="[object Null]"){
            return "id";
        }
    }
}
//类 
function Class(name){
    this.name = name;
    this.property = new Array();
}
```

在终端使用如下指令直接运行此脚本：

```
node Tool.js /Users/jaki/Desktop/json.json
```

命令后面所跟的参数为JSON文件的路径，JSON文件内容如下：

```json
{
  "code": 0,
  "message": "",
  "result": {
    "aid": "be3bdab8-fbf5-4e89-97cb-56b00048b09b",
    "audios": [],
    "avatar_url": "https://www.google.com/a.jpg",
    "call_price": 0,
    "cid": "efad6549-be62-40d7-a425-40f9b7730192",
    "cover_url": "https://www.google.com/a.jpg",
    "created_on": "1528349104",
    "fields": [
      {
        "key": "name",
        "value": "Mr.Wang"
      },
      {
        "key": "company",
        "value": "TicTalk"
      },
      {
        "key": "position",
        "value": "Senior Engineer"
      }
    ],
    "geo": {
      "description": "Shanghai, China",
      "latitude": 48.82694828196076,
      "longitude": 2.367038433387592
    },
    "id": 2,
    "message_price": 0,
    "photo_sets": [],
    "pid": "9b64395f-687c-4f34-9a5d-68d9adafa4cc",
    "rid": "578ff973-c707-42b2-bfc2-87e4dc8e0efd",
    "role_open": false,
    "sid": "8f560338-dfa5-48be-b44c-65fd87798543",
    "status": 0,
    "updated_on": "1528349104",
    "video_price": 0,
    "videos": [],
    "world_open": true
  }
}
```

运行后，可以看到在JSON文件同一目录下生成了oc.txt文件，内容如下：

```objectivec
@protocol Geo @end

@interface Geo : JSONModel

@property(nonatomic,strong)NSString<Optional>*description;

@property(nonatomic,strong)NSNumber<Optional>*latitude;

@property(nonatomic,strong)NSNumber<Optional>*longitude;

@end

@protocol Fields @end

@interface Fields : JSONModel

@property(nonatomic,strong)NSString<Optional>*key;

@property(nonatomic,strong)NSString<Optional>*value;

@end

@protocol Result @end

@interface Result : JSONModel

@property(nonatomic,strong)NSString<Optional>*aid;

@property(nonatomic,strong)NSArray<Optional>*audios;

@property(nonatomic,strong)NSString<Optional>*avatar_url;

@property(nonatomic,strong)NSNumber<Optional>*call_price;

@property(nonatomic,strong)NSString<Optional>*cid;

@property(nonatomic,strong)NSString<Optional>*cover_url;

@property(nonatomic,strong)NSString<Optional>*created_on;

@property(nonatomic,strong)NSArray<Fields*><Optional,Fields>*fields;

@property(nonatomic,strong)Geo<Optional,Geo>*geo;

@property(nonatomic,strong)NSNumber<Optional>*id;

@property(nonatomic,strong)NSNumber<Optional>*message_price;

@property(nonatomic,strong)NSArray<Optional>*photo_sets;

@property(nonatomic,strong)NSString<Optional>*pid;

@property(nonatomic,strong)NSString<Optional>*rid;

@property(nonatomic,strong)NSNumber<Optional>*role_open;

@property(nonatomic,strong)NSString<Optional>*sid;

@property(nonatomic,strong)NSNumber<Optional>*status;

@property(nonatomic,strong)NSString<Optional>*updated_on;

@property(nonatomic,strong)NSNumber<Optional>*video_price;

@property(nonatomic,strong)NSArray<Optional>*videos;

@property(nonatomic,strong)NSNumber<Optional>*world_open;

@end

@protocol MyObject @end

@interface MyObject : JSONModel

@property(nonatomic,strong)NSNumber<Optional>*code;

@property(nonatomic,strong)NSString<Optional>*message;

@property(nonatomic,strong)Result<Optional,Result>*result;

@end

===========m文件======================
@implementation Geo

@end

@implementation Fields

@end

@implementation Result

@end

@implementation MyObject

@end


```

后面我们只需要将类名调整成所需要的即可。

      下面是一个即用的网页转换器，采用的脚本代码和上面的代码基本一致：

[http://zyhshao.github.io/JSONToOC.html](http://zyhshao.github.io/JSONToOC.html)

使用效果如下：

![](https://oscimg.oschina.net/oscnet/df84e3d9c96e256ccf3067fac4432db8f14.jpg)

> 热爱技术，热爱生活中有趣的事务，互相交流，也在朋友 QQ：316045346
> 
> ——珲少

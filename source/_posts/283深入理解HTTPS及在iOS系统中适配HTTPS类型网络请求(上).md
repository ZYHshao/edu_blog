---
title: 深入理解HTTPS及在iOS系统中适配HTTPS类型网络请求(上)
date: 2016-12-16
categories: iOS逻辑初窥
tags: []
---
## 深入理解HTTPS及在iOS系统中适配HTTPS类型网络请求

### 一、引言

    本篇博客主要讨论如何在客户端与服务端之间进行HTTPS网络传输，为了深入理解网络传输的基础原理，更加灵活的校验证书，博客的前半部分也将介绍一些HTTPS网络传输原理。当然，文章中有不正和疏漏之处，还望朋友不吝指正，感谢！

### 二、HTTP与HTTPS

      我们都知道，HTTP是一种常用的网络传输协议，它是基于TCP的一种应用层协议，应用层是什么样的一个概念，通过下面这张示意图可以很好的理解：

![](https://static.oschina.net/uploads/space/2016/1216/180738_v5vY_2340880.png)

HTTP协议的网络传输十分常见，例如网易的主页http://www.163.com/。HTTP类型的网络传输使用十分方便，但是其在安全性上却有很大问题，列举如下：

1.HTTP协议在传输数据时是明文的，任何人通过一个简单的抓包工具，就可以截获到所有传输数据。

2.HTTP协议在传输数据时无法保证数据的完整，在截获到明文数据后，很容易就可以将其篡改，这也是一些网页总是被植入恶意广告的原因。

3.HTTP协议在传输数据时无法保证真实性，这也是最恐怖的一点。误入了域名欺骗的钓鱼网站，极容易对用户带来财产损失。

基于上面3点安全性的考虑，一种更加安全的网络传输协议势必要推行，那就是HTTPS。

      要理解HTTPS协议，首先需要明白什么是SSL/TLS。SSL全称“Secure Sockets Layer”，意思为安全套接层。其实由网景公司为了解决HTTP传输协议在安全方面的缺陷而设计的。后来被标准化，更名为TLS，全称“Transport Layer Security”，意思为传输层安全协议。

        那么现在就好理解了，其实HTTPS就是将HTTP协议与TLS协议组合起来，在不改变HTTP协议原设计的基础上，为其添加安全性校验并对传输的数据进行加密。那么TLS究竟在网络传输的那一层进行了处理了，下图可以很好的表示：

![](https://static.oschina.net/uploads/space/2016/1216/181011_7aze_2340880.png)

### 三、证书

      通过前面所介绍，我们知道HTTPS主要是为了解决3个问题：数据加密、数据完整、数据真实。那么下一步就是如何解决这些问题，数据加密在发送数据前依赖SSL层对数据进行加密，数据完整与真实性则要靠另一种关键技术：数字证书。

      通过一个小例子可以很容易的理解证书的作用，这个例子的来源是<编程随想>的作者，我这里暂且借用一下：A公司的a到B公司办事，为了证明a确实是A公司的职员而不是商业间谍，A公司会为a提供一个带有公章的证明，当B公司看到这个证明时，就可以信任办事员a。对比网络传输，这个证明就是证书，证书可以保证这个网站的真实性。我们继续往后分析，当B公司与越来越多的公司进行商业合作时，就又有新的问题出现了，比如C公司的c来B公司办事，就需要拿C公司带公章的证明，D公司的d来B公司办事就需要拿D公司带公章的证明...这样一来，B公司要存放好多公司的公章和证明的模板，才能够完成校验。这样未免也太麻烦了，对应到网络传输中，客户端就是B公司，各个网站都有自己的证书文件，这样客户端需要安装信任大量的证书，为了解决这样的问题，就有了第三方CA机构。第三方CA机构是由大家公认信任的机构，例如R公司为第三方信任机构，其业务是为其他公司提供公章证明，这样一来，B公司只要保有这个R公司的公章证明副本，其他A，C，D公司的办事员也只需要从R公司申请到一个公章证明就可以到B公司来交流业务了。

    CA的全称是“Certificate Authority”，意为证书授权中心。大部分CA机构颁发的证书都是需要付费的，CA机构颁发的证书一般都是根证书，根证书也比较容易理解，首先证书是有链式信任关系的，例如Y证书是由CA机构颁发的根证书，由这个Y证书还可以创建出许多子证书，子证书可以继续创建子证书，只要根证书是受信任的，其下所有的子证书都是受信任的，如下图：

![](https://static.oschina.net/uploads/space/2016/1216/181117_IgEB_2340880.png)

    我们可以打开开源中国博客的主页：[https://www.oschina.net/blog](https://www.oschina.net/blog)。在Chrome浏览器地址栏左边可以查看证书信息，如下：

![](https://static.oschina.net/uploads/space/2016/1216/181156_7klS_2340880.png)

点击证书信息，可以看到完整的证书链，如下图：

![](https://static.oschina.net/uploads/space/2016/1216/181242_THrb_2340880.png)

从图中可以看到，根证书是由CA机构VerSign公司颁发的。此处还可以看到当前证书是否有效以及过期时间，如果证书无效则说明此网页信息有可能被篡改过，用户在访问时就要小心了。

    除了CA机构可以签发证书外，个人其实也是可以创建证书的，当然个人创建的证书也是不被信任的，我们姑且把这类证书叫做自签名证书，如果用自签名证书搭建了HTTPS的服务，则客户端需要安装对应的证书信任，才可以进行此服务的访问。后面我们会进一步讨论自签名证书的使用。

### 四、搭建一个本地的HTTPS服务

    使用Node.js可以快速的搭建前端服务，我们这里使借助Express框架来搭建本地的HTTPS服务，用于测试我们后边将要进行HTTPS通讯。Express搭建搭建项目模板的过程在以前的一篇博客中有详细的介绍，这里就不再重复了，地址如下：

使用Express搭建前端项目：[https://my.oschina.net/u/2340880/blog/794928](https://my.oschina.net/u/2340880/blog/794928)。

    根据前面所述，搭建HTTPS服务需要有证书凭证，两种证书我们可以选择，一种是CA机构签发的证书，还有一种是我们自己制作的自签名证书，在Mac电脑上打开钥匙串访问应用，打开其中的证书助理，如下图所示：

![](https://static.oschina.net/uploads/space/2016/1216/181342_fdRw_2340880.png)

选择其中的为您自己创建证书选项，如下图：

![](https://static.oschina.net/uploads/space/2016/1216/181420_JjLk_2340880.png)

在之后的界面中，输入证书的名称，选择证书类型，如下图所示：

![](https://static.oschina.net/uploads/space/2016/1216/181501_CpTC_2340880.png)

上面，我把证书的名字创建成了珲少，身份类型选择的是自签名的根证书，证书类型选择SSL服务器，之后点击创建即可完成证书的创建。

    创建完成后，在钥匙串访问的登录证书中，可以看到已经有了珲少这个自签名的证书，如下图：

![](https://static.oschina.net/uploads/space/2016/1216/181606_ZBdu_2340880.png)

在证书上点击右键，选择导出选项，名字我将其取名为huishao，文件类型要选择.p12，如下图所示：

![](https://static.oschina.net/uploads/space/2016/1216/181658_lGT8_2340880.png)

点击存储后，需要设置一个访问密码，这个密码将来将用于从.p12文件中获取证书和密钥，如下图所示：

![](https://static.oschina.net/uploads/space/2016/1216/181745_rJu7_2340880.png)

之后，系统有可能会让你再次输入一个密码，将入下图所示，注意，这里需要输入的是系统的登录密码：

![](https://static.oschina.net/uploads/space/2016/1216/181822_yqxL_2340880.png)

完成上面操作后，我们已经将一个.p12文件导出到了桌面。那么这个.p12文件到底是个什么东西呢，它和证书之间又有什么关系呢，其实.p12文件一个复合文件，其中包装了私钥与证书信息，使用OpenSSL工具可以将其中的信息进行提取，搭建一个HTTPS的服务器需要两个文件，分别问证书文件和私钥文件，下面我们来从.p12文件中提取这些需要的文件。

     打开终端，cd到huishao.p12文件所在的目录下，使用如下命令可以将.p12文件中的私钥分解出来：

```bash
openssl pkcs12 -in huishao.p12 -nocerts -out privateKey.pem -nodes
```

之间会要求输入导出.p12文件时所设置的密码。

使用如下命令将.p12文件中的证书分解出来：

```bash
openssl pkcs12 -in huishao.p12 -nokeys -out cert.pem -nodes
```

之间也会要求输入导出.p12文件时所设置的密码。完成上面两部操作后，可以看到当前文件夹下多了两个文件，分别为cert.pem与privateKey.pem，他们分别是证书文件与密钥文件，将他们拷贝到Express项目的bin文件夹下，使得Express项目的结构看起来如下图所示：

![](https://static.oschina.net/uploads/space/2016/1216/140315_1XeC_2340880.png)

下面我们来配置Express项目。

      在生成好的Express项目中的www文件的末尾添加如下代码：

```javascript
/*
HTTPS
*/
var fs = require('fs');
var https = require('https');
/*
密钥文件
*/
var privatekey = fs.readFileSync('./privateKey.pem', 'utf8');  
/*
证书文件
*/
var certificate = fs.readFileSync('./cert.pem', 'utf8');  
var options={key:privatekey, cert:certificate};  
var serverHttps = https.createServer(options, app);  
/*
绑定端口
*/
serverHttps.listen(8080,function () {
    console.log('Https server listening on port ' + 8080);
}); 
```

用终端在bin文件夹下运行 node www，效果如下：

![](https://static.oschina.net/uploads/space/2016/1216/142118_dYVk_2340880.png)

在浏览器打开：https://localhost:8080/users，如果服务器搭建成功，Chrome中会出现如下效果：

![](https://static.oschina.net/uploads/space/2016/1216/143206_UCj3_2340880.png)

点击高级，点击其中的继续访问，可以正常获取到服务器返回的数据。到此，我们的HTTPS服务就搭建成功了。

### 五、iOS开发中通过配置info.plist文件来允许HTTP协议类型的通讯

      前面扯了太多，终于提到重点部分了。Apple在iOS9中就已经漏出一些强制HTTPS通讯的端倪，只是给了开发者一些过渡，在iOS10及以后的审核机制中，Apple对于强制HTTPS的推动将会越来越强，如何让自己的应用程序尽快的适配HTTPS相关的标准，是iOS开发者必须面对的任务。

      通过前面的分析我们了解，CA机构签发的证书是被默认信任的，这就是说，如果你的公司比较有钱，愿意花钱从CA机构申请一个付费的证书，那么很幸运，你的iOS工程是不需要做任何修改的，这些CA机构签发的证书是默认受信任的，因此你可以直接在程序中进行HTTPS类型的请求，所需要修改的只是将请求url改成https开头。但是另一种情况，无论出于什么原因，你的后台服务用的是自签名的证书，就想我们上面搭建的HTTPS服务一样，如果在不做任何处理的情况下在项目中访问这样的服务，就会出现问题了，原因是我们自己创建的自签名证书是不受信任的，系统默认拒绝了请求，示例如下：

```objectivec
-(void)normalHttps{
    NSURLRequest * req = [NSURLRequest requestWithURL:[NSURL URLWithString:@"https://localhost:8080/users"]];
    NSURLSessionConfiguration * config = [NSURLSessionConfiguration defaultSessionConfiguration];
    NSURLSession * session = [NSURLSession sessionWithConfiguration:config delegate:nil delegateQueue:[NSOperationQueue mainQueue]];
    [[session dataTaskWithRequest:req completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        NSLog(@"%@,%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding],error);
    }] resume];
}
```

运行工程后可以看到，并没有获取到相关数据，Xcode提示为：

```perl
NSURLSession/NSURLConnection HTTP load failed (kCFStreamErrorDomainSSL, -9802)
```

好了，那么我们先不管HTTPS的问题，如果我们直接对HTTP协议的服务进行请求，会不会有问题呢，将代码修改如下：

```objectivec
-(void)normalHttps{
    NSURLRequest * req = [NSURLRequest requestWithURL:[NSURL URLWithString:@"http://localhost:3000/users"]];
    NSURLSessionConfiguration * config = [NSURLSessionConfiguration defaultSessionConfiguration];
    NSURLSession * session = [NSURLSession sessionWithConfiguration:config delegate:nil delegateQueue:[NSOperationQueue mainQueue]];
    [[session dataTaskWithRequest:req completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        NSLog(@"%@,%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding],error);
    }] resume];
}
```

需要注意：Express在进行项目模板的创建时，会默认帮我们绑定一个3000端口的HTTP服务。

运行工程后，可以发现HTTP协议的请求也无法访问，报错如下：

```
 App Transport Security has blocked a cleartext HTTP (http://) resource load since it is insecure. Temporary exceptions can be configured via your app's Info.plist file.

```

其意思大致是说应用程序传输安全要求强制使用HTTPS类型的服务，但是开发者可以通过配置info.plsit文件来回避这一政策。这就是我们这节的重点，通过文件配置的方式来跳过应用安全传输协议。

    在iOS9之后，开发者可以在Info.plist文件中添加如下键：NSAppTransportSecurity。这个键用来配置APP传输安全的相关策略，是字典类型，其中可以设置的键有五个，如下：

NSAllowsArbitraryLoads：布尔值，默认为NO，设置为YES则代表除了NSExceptionDomains中设置的域名外，其他所有请求的协议类型都不受限制，也就是说可以支持HTTP类型的请求，这个键的作用域是全局的，App内所有的请求都受影响，但是如果开发者设置为了YES，在提交审核时需要说明原因。

NSAllowsArbitraryLoadsForMedia：布尔值，默认为NO，设置为YES的话，则应用程序内所有的媒体数据的加载将不受协议类型的限制，同样如果开发者设置为了YES，则在提交审核时需要说明原因。

NSAllowsArbitraryLoadsInWebContent：布尔值，默认为NO。如果设置为YES，则应用程序内所有WebView的请求加载不受协议类型的限制，开发者设置为了YES，则在提交审核时需要说明原因。

NSAllowsLocalNetworking：布尔值，默认为NO，如果设置为YES，则在加载本地资源时不受安全传输协议的限制。

NSExceptionDomains：字典，其主要对某些特殊域名做限制。其中结构可以表示如下：

```javascript
NSAppTransportSecurity : Dictionary {
    NSAllowsArbitraryLoads : Boolean
    NSAllowsArbitraryLoadsForMedia : Boolean
    NSAllowsArbitraryLoadsInWebContent : Boolean
    NSAllowsLocalNetworking : Boolean
    //对某些域名做特殊限制
    NSExceptionDomains : Dictionary {
        <domain-name-string> : Dictionary {
            NSIncludesSubdomains : Boolean
            NSExceptionAllowsInsecureHTTPLoads : Boolean
            NSExceptionMinimumTLSVersion : String
            NSExceptionRequiresForwardSecrecy : Boolean   // Default value is YES
            NSRequiresCertificateTransparency : Boolean
        }
    }
}

```

NSIncludesSubdomains：布尔值，这个键的作用是设置此域名下的所有子域名是否采用和父域名相同的配置。

NSExceptionAllowsInsecureHTTPLoads：布尔值，设置是否允许此域名使用自签名的证书进行请求，默认为NO，如果设置为YES，则在提交时需要说明原因。

NSExceptionMinimumTLSVersion：设置所使用的TLS版本。

NSExceptionRequiresForwardSecret：设置为NO，则不允许向前加密方式。

NSRequiresCertificateTransparency：如果设置为YES，则服务端的证书要有有效的透明时间戳。

### 六、iOS中使用自签名的证书进行HTTPS请求校验

    通过Info.plist文件我们是可以绕过安全传输协议的，但是不幸的是，从文档上看，无论开发者通过哪种方式来绕过安全传输协议，Apple都要求开发者在提审时提供合适的理由，这就是说：如果你使用了HTTP协议的请求，没有充足理由的话，你的App有很大的可能被审核拒绝。因此，更加保险的一种方式是将所有的服务都换成HTTPS协议的，如果有CA证书，当然完事大吉，如果没有，我们也可以通过验证自签名证书的方式来适配HTTPS协议。

    在进行HTTPS请求时，服务端会先将证书文件返回给客户端，如果客户端的证书信任列表中包含这个证书，则此请求可以正常进行，如果没有，则请求会被拒绝。因此，在iOS中适配自签名证书的HTTPS请求实际上就是将这个自签名的证书安装进客户端的信任列表。iOS中需要使用的证书是der格式的，可以使用如下命令将pem格式的证书转换成der格式的证书：

```
openssl x509 -inform PEM -in cert.pem -outform DER -out cert.der
```

将生成的cert文件添加进工程中，修改请求如下：

```objectivec
-(void)normalHttps{
    NSURLRequest * req = [NSURLRequest requestWithURL:[NSURL URLWithString:@"https://localhost:8080/users"]];
    NSURLSessionConfiguration * config = [NSURLSessionConfiguration defaultSessionConfiguration];
    NSURLSession * session = [NSURLSession sessionWithConfiguration:config delegate:self delegateQueue:[NSOperationQueue mainQueue]];
    NSURLSessionTask * task = [session dataTaskWithRequest:req completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        NSLog(@"%@,%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding],error);
    }];
    [task resume];
}

```

除此之外，需要实现一个SURLSessionDelegate的协议方法如下：

```objectivec
- (void)URLSession:(NSURLSession *)session didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge
 completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition, NSURLCredential * _Nullable credential))completionHandler {
    NSLog(@"证书认证");
    //先判断证书是否有效
    if ([[[challenge protectionSpace] authenticationMethod] isEqualToString: NSURLAuthenticationMethodServerTrust]) {
        //证书验证请求
        SecTrustRef serverTrust = [[challenge protectionSpace] serverTrust];
        /**
         *  导入多张CA证书（Certification Authority，支持SSL证书以及自签名的CA）
         */
        NSString *cerPath = [[NSBundle mainBundle] pathForResource:@"cert" ofType:@"der"];//自签名证书
        NSData* caCert = [NSData dataWithContentsOfFile:cerPath];
        //可以添加多张证书
        NSArray *caArray = @[caCert];
        //验证规则
        NSMutableArray *policies = [NSMutableArray array];
        [policies addObject:(__bridge_transfer id)SecPolicyCreateBasicX509()];
        SecTrustSetPolicies(serverTrust, (__bridge CFArrayRef)policies);
        NSMutableArray *pinnedCertificates = [NSMutableArray array];
        //进行自签名证书的添加
        for (NSData *certificateData in caArray) {
            [pinnedCertificates addObject:(__bridge_transfer id)SecCertificateCreateWithData(NULL, (__bridge CFDataRef)certificateData)];
        }
        SecTrustSetAnchorCertificates(serverTrust, (__bridge CFArrayRef)pinnedCertificates);
        SecTrustResultType result = -1;
        //通过本地导入的证书来验证服务器的证书是否可信
        SecTrustEvaluate(serverTrust, &result);
        NSURLCredential *credential = [NSURLCredential credentialForTrust:serverTrust];
        completionHandler(NSURLSessionAuthChallengeUseCredential,credential);
        return [[challenge sender] useCredential: credential
                      forAuthenticationChallenge: challenge];
        
    }
}
```

如上修改后，再次运行工程，可以看到已经成功请求到了HTTPS自签名证书服务提供的数据：

![](https://static.oschina.net/uploads/space/2016/1216/165455_bErP_2340880.png)

介于篇幅过长，关于NSURLAuthenticationChallenge相关类的更多探讨和常用网络库AFNetworking中HTTPS的适配，下篇博客会继续介绍。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

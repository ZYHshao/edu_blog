---
title: 分分钟搞定IOS远程消息推送
date: 2015-05-12
categories: iOS逻辑初窥
tags: []
---
## 分分钟搞定IOS远程消息推送

### 一、引言

IOS中消息的推送有两种方式，分别是本地推送和远程推送，本地推送在[http://my.oschina.net/u/2340880/blog/405491](http://my.oschina.net/u/2340880/blog/405491)这篇博客中有详细的介绍，这里主要讨论远程推送的流程与配置过程。

### 二、远程推送机制的原理

#### 1、从一张很火的图说起

搜索IOS远程推送，你总能看到一张如下的流程示意图，因为这张图确实很火，所以我也将它引用在此：

![](http://static.oschina.net/uploads/space/2015/0511/125733_182P_2340880.jpg)

这张图示意的很清晰，大致意思是这样：你的应用服务端将消息发送到apple的APNS服务器，APNS服务器将消息推送到指定的Iphone，最后由Iphone负责将消息推送至你的APP。在此先不说这个过程是如何实现的，仅仅看这个流程，你可能会觉得，在你们服务端和客户端之间增加了一个apple的APNS，不是增加开发者的负担么？其实结果恰恰相反，因为apple对推送的统一管理，使我们开发者的工作变得异常简单。

#### 2、服务端如何连接到客户端的

如果你是做android开发的，你一定非常了解长链接与心跳包。事实上，大部分的android应用的推送也确实是通过长链接来实现的。因为android系统的开放性，APP是很容易做到自启动和后台长链接的，而心跳验证，就是始终保证长链接属于接通状态，然后由服务端直接推送消息。如果IOS开发者也采用这种思路，就十分困难了，在IOS中想要保持一个APP服务始终不被系统杀死，我只能说太难了。通过上面的流程图，对比android的推送思路，我们很容易明白，IOS中其实也始终有一个长链接，那就是系统本身，这个长链接始终与APNS服务器相连，然后统一管理所有应用程序的推送。

#### 3、这是IOS推送机制的优势？

下面的这些，只是我个人的一些看法。系统并无优劣，优劣在于个人喜好。

1、因为推送的服务端是appleID的验证用户，推送可靠性会高。

2、所有推送消息由APNS统一管理，效率高。

3、在客户端只需系统维护一个长链接，节省了用户流量消耗和手机的性能消耗，并且提高了安全性，使得有恶意推送和流氓软件的几率降低。

### 三、分分钟让你的APP收到远程推送

#### 1、工欲善其事、必先利其器——创建推送证书

(1)请求CSR文件

在MAC应用程序中找到钥匙串访问，打开它。

点击选项栏中的钥匙串访问中的证书助理：

![](http://static.oschina.net/uploads/space/2015/0511/132801_6tXU_2340880.png)

选择从证书颁发机构申请证书:

![](http://static.oschina.net/uploads/space/2015/0511/133114_kdP0_2340880.png)

填写电子邮件和名称，选择储存到磁盘，然后继续。

这时，我们存储的地方有了这样一个文件：CertificateSigningRequest.certSigningRequest。

(2)导出密钥文件

打开钥匙串，会发现多了一对密钥，名字就是上面你填写的常用名称。

我们选择专用密钥进行导出，然后设置一个我们自己的密码：

![](http://static.oschina.net/uploads/space/2015/0511/142351_iETv_2340880.png)

这时候我们又有了一个后缀名为.p12的文件。

(3)创建AppId

到https://developer.apple.com的member Center：

![](http://static.oschina.net/uploads/space/2015/0511/133637_9huq_2340880.png)

用你付过费的开发者appleID登陆后，选择Certificates:

![](http://static.oschina.net/uploads/space/2015/0511/133853_elU8_2340880.png)

![](http://static.oschina.net/uploads/space/2015/0511/143005_JNx9_2340880.png)

如果你的项目已经创建了APP id，则可以不用重新创建，但是你创建的APP id必须要支持远程推送。如果还没有创建，点击加号，创建一个：

![](http://static.oschina.net/uploads/space/2015/0511/143212_ykpm_2340880.png)

之后的界面中APP ID有两种类型：Explicit和Wildcard，分别是特殊的和通配的，我们需要推送功能，这个ID不能是通配的，所以我们选择第一个。

![](http://static.oschina.net/uploads/space/2015/0511/143428_IMMo_2340880.png)

这里需要填的的Bundle ID必须和我们App中的一致：

![](http://static.oschina.net/uploads/space/2015/0511/144041_bDCu_2340880.png)

在APP ID的服务设置中，将Push Notification勾选上，点击continue。

![](http://static.oschina.net/uploads/space/2015/0511/144446_vOpN_2340880.png)

之后点击submit，最后点击Done。这时我们的APP IDs列表中会出现我们刚才创建的APP ID。

(4)创建证书

点击我们刚才创建的APP ID，你会看到Push Notification一行为未设定的。我们点击Edit。

![](http://static.oschina.net/uploads/space/2015/0511/145201_c9sx_2340880.png)

在Push Notifications设置里是如下界面，development是开发证书，Production是产品证书，我们现在需要测试，所以用Development证书，上线时要使用Production证书。点击Create Certificate。

![](http://static.oschina.net/uploads/space/2015/0511/150742_prpH_2340880.png)

接着点击continue，如下界面会让我们选择一个CSR文件，我们第一步创建的文件在这里派上用场了，选择那个文件，点击Generate。

![](http://static.oschina.net/uploads/space/2015/0511/151025_FvPV_2340880.png)

将创建好的证书下载到电脑中：

![](http://static.oschina.net/uploads/space/2015/0511/151826_V0Gx_2340880.png)

至此，我们已经有了三个文件了，分别是CSR文件，.p12文件，.cer文件。要将这三个文件放在同一个目录下。.cer文件分为测试和产品两个，需要哪个自行选择。写了这么多，我们的准备工作可算是做完了，不要灰心，其实你的推送工作基本上也就做完了。只是申请过程麻烦了一些，但工程的代码，我们几乎不用怎么配置。

#### 2、兵马未动、粮草先行——服务端进行信息推送的设置

(1)处理证书

打开终端cd到我们上面得到的三个文件所在的目录。

在终端执行如下命令：

```
$ openssl x509 -in aps_development.cer -inform der -out PushCert.pem
```

aps_development.cer是刚才生成的.cer文件的文件名。会在当前文件夹中生成一个pem文件，这是我们服务端对应的证书。

再执行如下命令：

```
$ openssl pkcs12 -nocerts -out PushKey.pem -in key.p12
```

key.p12是上面生成的.p12文件的文件名。这时终端会让输入密码，这里的密码就是上面我们设置的密钥的密码。输入密码后回车，如果密码正确，会让我们输入新密码(一定切记)，输入两次后，终端会提示成功创建PushKey.pem文件。

最后一步，将我们生成的两个pem文件和成为一个：

```
$ cat PushCert.pem PushKey.pem > ck.pem
```

(2)测试证书是否可用

在终端执行下面的命令：

```
$ telnet gateway.sandbox.push.apple.com 2195
```

等一小会，如果终端显示下面的情形，则证书正常。

![](http://static.oschina.net/uploads/space/2015/0511/155534_99ff_2340880.png)

然后执行如下命令：

```
openssl s_client -connect gateway.sandbox.push.apple.com:2195 -cert PushChatCert.pem -key PushKey.pem
```

输入密码后回车显示如下的结果则连接成功：

![](http://static.oschina.net/uploads/space/2015/0512/094740_LjCL_2340880.png)

#### 3、天涯海角、一步之遥——应用程序中的配置

在我们项目的AppDelegate中添加如下代码：

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
double version = [[UIDevice currentDevice].systemVersion doubleValue];//判定系统版本。
if(version>=8.0f){
        UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:(UIRemoteNotificationTypeBadge|UIRemoteNotificationTypeSound|UIRemoteNotificationTypeAlert) categories:nil];
        [[UIApplication sharedApplication] registerUserNotificationSettings:settings];
        
    }else{
        UIRemoteNotificationType myTypes = UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeSound;
        [[UIApplication sharedApplication] registerForRemoteNotificationTypes:myTypes];
    }
}

- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo{ // 处理推送消息
    NSLog(@"userinfo:%@",userInfo);
    NSLog(@"收到推送消息:%@",[[userInfo objectForKey:@"aps"] objectForKey:@"alert"]);
}
- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *) error {
    NSLog(@"Registfail%@",error);
}
-(void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken{
    NSLog(@"%@",deviceToken);//这里的Token就是我们设备要告诉服务端的Token码
}
```

下面是网上搜的PHP服务端的代码：

```
<?php
//这里填写设备的Token码
$deviceToken = '74314cc9e8f747e2fa96c2c1585c830cdf994de6b453ce9fa1c09ba396b2f9e9';
//这里是密钥密码
$passphrase = 'abcabc';
//推送的消息
$message = '这是一条推送消息';

////////////////////////////////////////////////////////////////////////////////

$ctx = stream_context_create();
stream_context_set_option($ctx, 'ssl', 'local_cert', 'ck.pem');//ck文件
stream_context_set_option($ctx, 'ssl', 'passphrase', $passphrase);

// Open a connection to the APNS server
$fp = stream_socket_client(
    'ssl://gateway.sandbox.push.apple.com:2195', $err,
    $errstr, 60, STREAM_CLIENT_CONNECT|STREAM_CLIENT_PERSISTENT, $ctx);

if (!$fp)
    exit("Failed to connect: $err $errstr" . PHP_EOL);

echo 'Connected to APNS' . PHP_EOL;

// Create the payload body
$body['aps'] = array(
    'alert' => $message,
    'sound' => 'default'
    );

// Encode the payload as JSON
$payload = json_encode($body);

// Build the binary notification
$msg = chr(0) . pack('n', 32) . pack('H*', $deviceToken) . pack('n', strlen($payload)) . $payload;

// Send it to the server
$result = fwrite($fp, $msg, strlen($msg));

if (!$result)
    echo 'Message not delivered' . PHP_EOL;
else
    echo 'Message successfully delivered' . PHP_EOL;

// Close the connection to the server
fclose($fp);
    
?>
```

把上面的PHP文件和我们的ck文件放在同一目录下。在终端的当前目录下，执行如下命令：

```
$php push.php
```

如果我们的设备王略正常，就可收到推送的消息了：

![](http://static.oschina.net/uploads/space/2015/0512/095131_WNRR_2340880.png)

![](http://static.oschina.net/uploads/space/2015/0512/095142_Ooai_2340880.png)

### 四、几点注意

1、如果终端发送信息时提示密钥不可访问之类的错误，请检查是否cd到了当前目录，如果还存在问题，将密钥部分从新生成一次。

2、注意PHP代码中的字符为英文字符。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592

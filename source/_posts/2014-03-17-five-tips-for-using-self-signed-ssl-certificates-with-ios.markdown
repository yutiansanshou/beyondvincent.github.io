---
layout: post
title: "在iOS上使用自签名的SSL证书"
date: 2014-03-17 14:15
comments: true
categories: iOS探索
published: true

---

![](/images/2014/03/09.png)

<!--more-->

本文译自：[Five Tips for Using Self Signed SSL Certificates with iOS](http://blog.httpwatch.com/2013/12/12/five-tips-for-using-self-signed-ssl-certificates-with-ios/)

小引：

上周苹果release了iOS 7.1，用户升级至此版本，去下载企业级应用时，如果应用不是用https部署的，那么会提示服务器上的证书无效，如下图所示：

![](/images/2014/03/11.png)

小引:

在iOS 7.1中需要将plists文件的url路径设置为https才能下载安装，如果之前是用http部署的，这就需要对服务器做一些修改。网上流传着([Enterprise app deployment doesn't work on iOS 7.1](http://stackoverflow.com/questions/20276907/enterprise-app-deployment-doesnt-work-on-ios-7-1/22325916#22325916))把plist放到公开的具有https功能的文件服务器上即可(dropbox或SkyDrive)，经试验，都是可行的。不过如要企业内部的网络无法访问外网，这又如何是好呢？网上也流传需要受信任的证书才行，这样一来对于某些企业的内部部署就比较麻烦了(局域网、主机名变动等都会引起很麻烦的事情)，各区域可能还要用不同的证书(对应不同的host name)。

经查苹果的相关文档(2014年2月份)，是可以使用http或者https的，后来打电话咨询苹果技术支持人员，告知文档还没来得及更新，不过自签名的证书也可以使用，最好首先要把证书(最好是CA根证书)通过mail或者配置工具安装到iOS设备上，然后根据这个CA构建web 服务器上用到的证书，这样当iOS设备访问web服务器来安装app时，就可以正常进行了。这样的话，我们只需要根据不同的服务器制作对应的自签名证书即可，有问题也方便修改跟进。

下面这篇文章就是介绍如何在iOS上面使用自签名的SSL证书。

先来看看目录：

1. 在iPhone或者iPad上的Safari中不要接受自签名证书
2. 像安装iOS的Configuration Profile一样安装自签名证书
3. 不要用IIS创建自签名证书
4. 利用OpenSSL创建自签名证书
5. 创建自己的证书颁发机构(CA)

虽说SSL证书的购买价格不是太贵，不过有时候我们自行创建的话要更加方便一些，例如我们需要在开发环境配置SSL时、我们的测试服务器有不同的主机名或者开发的系统只有本地局域网才能访问。

实际上我们可以免费的创建[自签名SSL证书](http://en.wikipedia.org/wiki/Self-signed_certificate)，并不需要给证书颁发机构(CA)支付任何费用，也不需要遵守任何审计要求。

使用自签名SSL证书的缺点就是浏览器不会自动的信任相关的站点。如果用iPhone或者iPad上的Safari打开相关站点，会看到如下提示界面：

![](/images/2014/03/01.png)

这里的HttpWatch iOS app还会有更详细的提示内容：

![](/images/2014/03/02.png)

本文剩余内容将介绍如何对iOS做配置，以避免上面遇到的问题，以及如何方便的创建和管理自签名证书。

##在iPhone或者iPad上的Safari中不要接受自签名证书##

当在Safari中第一次访问自签名证书的站点时，会有如下提示(Continue或Details->Accept)：

![](/images/2014/03/03.png)

如果你照着选择的话，是可以在Safari中打开站点的，不过这样会有两个明显的缺点：

1. 在Safari中接受证书只会添加一个`SSL例外`：防止Safari对此站点做出的警告提示。实际上并不会将证书安装到iOS中，以成为可信任的证书。同台设备的其它程序在连接该站点时仍然会失败。
2. 一旦将`SSL例外`添加到iOS 7中了，就无法从iOS 7中移除。在之前的版本还可以通过Settings->Safari，选中"Clear Cookies and Data"进行删除。在iOS 7中这好像没有效果。------除非`General > Reset > Reset Settings`。

当然，苹果官方也提供了一个工具[iPhone configuration utility for Mac and PC](http://support.apple.com/downloads/#iphone%20configuration%20utility)，通过该工具可以把证书安装至设备中。如果邮件不可用，或者要批量安装的话，这是一个好方法。


##像安装iOS的Configuration Profile一样安装自签名证书##

要想把SSL证书在iOS设备上安装为可信任的，可以通过发邮件到设备上，并将证书以附件的形式带在邮件中：

![](/images/2014/03/04.png)

选择邮件中的附件，以将证书安装至iOS设备中。选择附件之后，选择`Install`来安装证书。完成安装之后，再在Safari或者其它iOS应用中使用该证书时，就不会有相关提醒了。

并且这与Safari添加的`SSL例外`有一个很大的区别，你可以在任意时刻通过`Settings->General->Profiles`来查看相关证书，如果有必要的话，还可以将其移除：

![](/images/2014/03/05.png)

当然，苹果也提供了一个工具[iPhone configuration utility for Mac and PC](http://support.apple.com/downloads/#iphone%20configuration%20utility)，通过该工具可以把证书安装到设备中，如果邮件不可用，或者要批量安装证书的话，可以用这个工具。

##不要用IIS创建自签名证书##

在IIS中创建自签名证书非常的简单。只需要选择菜单中的`Create Self-Signed Certificate`即可：

![](/images/2014/03/06.png)

不过IIS简单的使用计算机名称当做证书的主机名：

![](/images/2014/03/07.png)

大多数时候，计算机名称与主机名是不匹配的，这样的话自签名的证书永远都不会受信任——即使已经安装至iOS设备中：

![](/images/2014/03/08.png)

要修复这个问题可以安装并运行IIS 6 Toolkit中的SelfSSL。不过，要是使用OpenSSL则非常简单，下面我们就来看看吧。

##利用OpenSSL创建自签名证书##

创建自签名证书最简单的一种方法就是使用[OpenSSL命令行工具](http://www.openssl.org/related/binaries.html)，这个工具在许多平台上面都可以使用，并且在Mac OSX上面是默认安装好了的。

首先，创建一个私钥文件：

```
openssl genrsa -out myselfsigned.key 2048
```

然后创建自签名证书：

```
openssl req -new -x509 -key myselfsigned.key -out myselfsigned.cer -days 365
-subj /CN=www.mysite.com
```

上面的命令中，关于私钥和证书(cer)的文件名可以是任意的。其中`CN`参数需要设置为主机名(例如https://www.mysite.com)。而`days`参数则指定证书从创建开始的有效天数。

在Apache服务器上面可以直接使用私钥和证书文件(做相关的SSL配置即可)。在IIS中需要一个PFX文件，通过该文件，可以将证书导入至IIS的Server Certificates中。当然通过OpenSSL可以创建PFX文件：

```
openssl pkcs12 -export -out myselfsigned.pfx -inkey myselfsigned.key
-in myselfsigned.cer
```

##创建自己的证书颁发机构(CA)##

使用自签名证书会有这样的问题：需要为每台设备中用到的每个证书设置相关的信任关系。有一个解决办法就是创建自己的证书颁发机构(CA)根证书，然后基于该根证书创建别的证书。

这样一来就是自己扮演着CA，取代了商业性质的CA。这样做的好处就是自己的CA证书只需要在每台设备上安装一次即可。之后，设备会自动的信任基于CA根证书创建的证书。

创建CA证书只需要两个步骤即可，首先是创建私钥文件(跟之前的一样)：

```
openssl genrsa -out myCA.key 2048
```

然后是创建证书：

```
openssl req -x509 -new -key myCA.key -out myCA.cer -days 730
-subj /CN="My Custom CA"
```

上面创建的证书(myCA.key)可以公开发布出去，并安装在iOS或者其它OS上，以此当做内置的受信任根CA。自制的CA证书存储在`General->Settings->Profile`：

![](/images/2014/03/10.png)

其中私钥文件(myCA.key)只用是再创建新的SSL证书时使用。

上面的CA创建好之后，我们可以基于该证书创建许多证书。注意，这里多了一个步骤：必须创建一个CSR(客户端证书请求文件)——就像购买商业的SSL证书一样。

首先需要创建一个私钥文件：

```
openssl genrsa -out mycert1.key 2048
```

然后是创建CSR:

```
openssl req -new -out mycert1.req -key mycert1.key -subj /CN=www2.mysite.com
```

接着用这个CSR创建证书：

```
openssl x509 -req -in mycert1.req -out mycert1.cer -CAkey myCA.key
-CA myCA.cer -days 365 -CAcreateserial -CAserial serial
```

这里创建的证书(mycert.cer)可以安装在一台web服务器上，并且已经安装了相关CA证书的iOS设备都可以访问该web服务器。

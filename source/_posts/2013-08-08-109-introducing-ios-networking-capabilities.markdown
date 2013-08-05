---
layout: post
title: "Professional iOS Network Programming翻译第一章：iOS网络功能简介"
date: 2013-08-08 00:41
comments: true
categories: iOS翻译
published: false

---
{% img /images/2013/07/simple_social_network.png 400 300 %}


## **小引**

在iOS开发中，网络是非常重要的功能，针对iOS开发者来说，掌握好iOS中网络功能的开发也是必不可少的。最近在看一本书:[`Professional iOS Network Programming`](http://www.amazon.com/Professional-iOS-Network-Programming-Connecting/dp/1118362403)，网络编程方面介绍比较全面，非常适合iOS开发者去读一读，由于精力有限，我在这里可能会挑选一部分内容进行翻译，也有可能全书翻译。

<!--more-->

###**目录**
* Professional iOS Network Programming介绍
* 第一章：iOS网络功能简介
	* 了解网络框架
	* iOS网络APIs
		* NSURLConnection
		* Game Kit
		* Bonjour
		* NSStream
		* CFNetwork
		* BSD Sockets
	* Run Loops
		* Run Loop模型
	* 小结


##**Professional iOS Network Programming介绍**

这本书主要介绍在iOS中的网络编程知识，主要包含如下内容：

	在客户端和服务器端之间进行HTTP请求
	管理客户端和服务器端之间进行的数据负载
	处理HTTP请求中的错误
	网络通信中的安全
	加强网络通信的性能
	socket级别的通信
	推送通知的实现
	同一设备中两个程序间的通信
	不同设备中两个程序间的通信

##第一章：**iOS网络功能简介**

本章的内容

	了解iOS网络框架
	开发者可用的重要网络APIs
	在程序中有效的利用run Loop

优秀的iOS程序需要简单和直观的用户界面，同样，具有与web service通信功能的优秀程序需要一个良好架构的网络层。在设计应用程序架构时，必须考虑到程序的灵活性，以适应经常变化的需求，并能够正确的处理不断变化的网络条件，同时还需要保持核心设计原则：可维护性和可扩展性。

当在设计移动应用程序的架构时，必须熟悉一些相关的核心概念，例如run loop，可用的网络APIs，以及这些APIs是如何与run loop整合起来实现一个具有响应式的网络应用程序框架。本章详细的探讨了run loop，以及如何在程序中对其有效的使用。同样，也对关键的APIs做了一个概述，并且介绍了什么时候应该使用什么APIs。

###**了解网络框架**

在开发iOS应用程序(与网络交互的程序)之前，首先必须理解Objective-C中的网络层是如何组成的，如图1-1所示：

{% img /images/2013/07/networking_layers_in_OC.jpg %}

图1-1

如上图所示，每个iOS应用程序都都位于由四层组成的网络框架之上。最上面的是Cocoa层，该层包含了一些Objective-C写的APIs：`URl加载`，`Bonjour`和`Game Kit`。Cocoa之下是Core Foundation，该层包含了一组C语言写的APIs：`CFNetwork`，该层中的代码是大量应用层中网络代码的基础。CFNetwork在`CFStream`和`CFSocket`之上，提供了一个简单的网络接口。CFStream和CFSocket对BSD socket做了轻量级的封装，BSD是基于硬件上面的一层，它于无线通信相关硬件设备最接近。BSD socket是严格使用C实现的，通过BSD，开发者对网络中的任意通信(远程设备或者服务)拥有绝对控制权。

在上图中，越往下层走，会获得更高的控制权，但是相对于上一层来说易用性更差。苹果建议使用CFNetwork层以及之上的。在BSD层中的raw socket不能访问系统的VPN，也不能激活Wi-Fi或蜂窝无线模块，这些功能是由CFNetwork处理的。

在设计应用程序的网络层之前，开发者必须理解各种可用的APIs。下一节中会介绍iOS中关键的网络框架，并简短的解释一下如何使用它们。在本书后面的章节中，会详细介绍每个APi。


###**iOS网络APIs**

在框架中的每层里面，都有一套关键的APIs提供给开发者相关的功能以及控制权。在图1-1中每层相对于下一层，会有更多的封装。不过封装之后，会失去一些控制权。本节就来大概的预览一下iOS中网络层关键的APIs，并探讨一下什么时候使用它们。

#####NSURLConnection

NSURLConnection是Cocoa中的一个API，它提供了一个简单的方法来请求URL，可以与web service进行交互，以获取一个图片或者视频，或者简单的获取一个HTML文档。NSURLConnection构建于NSStream之上，它支持4种通用的URI schemes：file，HTTP，HTTPS和FTP。虽然NSURLConnection限制了可以使用的协议，但是它封装了大量底层API必须要做的任务：对缓冲区进行读写，另外还内置支持认证(authentication)，并提供了一个健壮的缓存引擎。

实际上NSURLConnection本身提供的接口比较少，主要依赖于`NSURLConnectionDelegate`协议，通过该协议，应用程序可以与网络连接生命周期中的多个点进行交互。NSURLConnection的请求默认是异步的；不过也提供了一个同步请求方法。由于同步请求会阻塞当前调用的线程，所以必须根据具体情况来设计应用程序。在第三章(发起请求)中会详细介绍NSURLConnection，并提供了一些示例。

######Game Kit

在iOS程序中，Game Kit提供了另外一种点对点(peer-to-peer)网络通讯的方法。在传统的网络配置中，Game Kit是构建于Bonjour之上的；不过Game Kit并不需要网络基础设施提供的功能。它可以创建ad-hoc Bluetooth Personal Area Networks(PAN)，通过该PAN可以让设备在邻近范围内或网络条件不允许的情况下，进行通讯，

Game Kit只需要一个会话id(session identifier)，显示的名称(display name)，以及配置网络时的连接模式。不需要配置socket，或其它任意底层网络通讯的连接。Game Kit使用GKSessionDelegate协议进行通讯。在第12章(利用Game Kit进行设备间的通讯)中介绍了将Game Kit集成到我们的程序中。

######Bonjour

Bonjour是苹果实现的零配置联网。Bonjour提供了这样一种机制：发现并连接到设备或者网络中的服务，这些过程中我们并不需要知道设备的网络地址，相反，Bonjour涉及到元祖名称，服务类型和域。Bonjour封装了底层网络接口需要的multicast DNS(mDNS)，以及基于DNS服务的发现(DNS-SD)。

在Cocoa层，NSNetService API提供了相关接口用来发布和解决Bonjour服务的地址信息。我们可以使用NSNetServiceBrowser API来发现网络中可用的服务。为了通信，发布一个Bonjour服务，即使是使用Cocoa层的API，也需要明白Core Foundation中对socket的配置。在第13章"Ad-Hoc Networking with Bonjour"中，深入介绍了零配置联网(Bonjour)，并给出了一个示例介绍如何实现一个基于Bonjour的服务。

######NSStream

NSStream是Cocoa层里面的API，构建于CFNetwork之上，是NSURLConnection的基础部分，并且还适用于较底层的网络任务。就像NSURLConnection，NSStream提供了一种与远程服务或者本地文件通信的机制。另外，还NSStream还可以在别的一些一些上进行通信，例如`telnet`，`SMTP`，NSURLConnection并不支持这些协议。

NSStream还提供了额外的一些控制功能，不过这是要付出代价的。NSStream并没有内置支持处理HTTP/S响应状态码的处理，也不支持认证功能。它是用C缓存器进行数据的发送和接收的，这跟Objective-C还有点区别。它也不能管理多个请求，如果需要相应的功能，需要在其子类中添加功能。NSStream是异步的，它通过NSStreamDelegate进行通信。在第8章中“Low-Level Networking”，以及第13章中“Ad-Hoc Networking with Bonjour”，都不同程度的实现了NSStream。


######CFNetwork

######BSD Sockets

###**Run Loops**


####Run Loop模型


###**小结**

对于iOS开发者来说，理解iOS网络框架中的各层，以及应用程序如何与run loop交互是非常重要的。一个优秀的网络架构层会给应用程序提供难以置信的灵活度。如果网络架构层的设计非常糟糕，那么这是很难获得成功和扩展能力的。

本章预览了一下各个网络APIs，并做了一些比较。在这里只是简单的介绍了一下，在后续章节中，会深入讨论。


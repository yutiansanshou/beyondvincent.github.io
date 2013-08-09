---
layout: post
title: "Professional iOS Network Programming翻译第一章：iOS网络功能简介"
date: 2013-08-09 16:52
comments: true
categories: iOS翻译
published: ture

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

`Bonjour`是苹果实现的零配置联网。Bonjour提供了这样一种机制：发现并连接到设备或者网络中的服务，这些过程中我们并不需要知道设备的网络地址，相反，Bonjour涉及到元祖名称，服务类型和域。Bonjour封装了底层网络接口需要的multicast DNS(mDNS)，以及基于DNS服务的发现(DNS-SD)。

在Cocoa层，NSNetService API提供了相关接口用来发布和解决Bonjour服务的地址信息。我们可以使用NSNetServiceBrowser API来发现网络中可用的服务。为了通信，发布一个Bonjour服务，即使是使用Cocoa层的API，也需要明白Core Foundation中对socket的配置。在第13章"Ad-Hoc Networking with Bonjour"中，深入介绍了零配置联网(Bonjour)，并给出了一个示例介绍如何实现一个基于Bonjour的服务。

######NSStream

`NSStream`是Cocoa层里面的API，构建于CFNetwork之上，是NSURLConnection的基础部分，并且还适用于较底层的网络任务。就像NSURLConnection，NSStream提供了一种与远程服务或者本地文件通信的机制。另外，还NSStream还可以在别的一些一些上进行通信，例如`telnet`，`SMTP`，NSURLConnection并不支持这些协议。

NSStream还提供了额外的一些控制功能，不过这是要付出代价的。NSStream并没有内置支持处理HTTP/S响应状态码的处理，也不支持认证功能。它是用C缓存器进行数据的发送和接收的，这跟Objective-C还有点区别。它也不能管理多个请求，如果需要相应的功能，需要在其子类中添加功能。NSStream是异步的，它通过NSStreamDelegate进行通信。在第8章中“Low-Level Networking”，以及第13章中“Ad-Hoc Networking with Bonjour”，都不同程度的实现了NSStream。


######CFNetwork

`CFNetwork API`构建于BSD socket之上，被用于NSStream、URL加载系统、Bonjour和Game Kit APIs的实现中。CFNetwork中默认支持一些上层协议，例如HTTP和FTP。CFNetwork和BSD socket最关键的区别就是集成了run loop。如果在程序中使用了CFNetwork，输入(input)和输出(output)事件会在线程的run loop中被调度。如果输入和输出事件发生于非主线程上，那么我们需要负责在这个线程中以适当的模式启动run loop。本章后面的“Run Loops”小节会有相关介绍。

CFNetwork提供的配置选项要比URL加载系统更多，这有好的一面，也有不好的一面。当利用CFNetwork创建一个HTTP请求时，这些配置选项是可见的。在创建请求的时候必须手动添加所有的HTTP header，以及cookies，然后与请求一起提交。而使用NSURLConnection时，标准的header和cookie jar中的任意cookies都自动的添加好了。

CFNetwork下面还有来自Core Foundation层中的CFSocket和CFStream APIs。CFNetwork中有一些用于特定协议的APIs，例如用于与FTP服务通讯的CFFTP，用于收发HTTP消息的CFHTTP，以及用于发布和浏览Bonjour服务的CFNetServices。第八章中将详细介绍CFNetwork，而在13章中会简要介绍一下Bonjour。

######BSD Sockets

在网络架构中，`BSD Sockets`为网络通信提供了最基础的服务，也是最底层的一个APIs。BSD Socket是用C语言实现的，不过完全可以用在Objective-C代码中。一般不建议直接使用BSD Socket API，因为它在操作系统中没有任何hook。例如，BSD Socket既不走系统中的VPN通道，也没相关的API来自动激活已经关闭掉的Wi-Fi或蜂窝无线设备。苹果建议编程时使用CFNetwork或更高的层中的API。第8章中详细介绍了BSD Sockets以及CFNetwork，并提供了一个示例介绍了如何将它们集成到程序中。下一节将讨论run loop——从操作系统中检测网络事件，这些事件会被用于我们的程序中。


###**Run Loops**
Run loop对应的类是`NSRunLoop`，它其实是线程中的一个基础组件，有了run loop之后，操作系统就能够唤醒休眠中的线程，以对即将到来的事件进行管理。一个run loop是一个循环配置的用来调度任务，并在一个时钟周期内处理即将到来的事件。在iOS程序中的每个线程中最多能有一个run loop。主线程中的run loop在程序启动的时候就默认开启了，并且当程序的delegate applicationDidFinishLaunchingWithOptions:被调用之后，我们就可以对其进行访问了。

在非主线程中，如果需要使用run loop，需要开发者明确的开启run loop。在非主线程中启动之前，必须添加一个输入源(input source)或者timer；否则run loop会立即退出。run loop给开发者提供了与线程交互的能力，不过并不是总是需要它的。例如有时候线程在处理大量数据时，并不不需要进行任何交互，此时就不需要启动run loop了。如果线程需要跟网络进行交互，此时就需要启动run loop。

Run loop接收的事件有两种源类型：输入源`(input sources)`和`计时器(timers)`。在输入源中一般要么是基于端口的，要么就是自定义的，这些事件通过异步的方式派发到程序中。这两种类型源的最大区别就是基于端口的内核信号源是自动的，而自定义的源必须在不同线程中手动管理相关信号。在创建自定义输入源时，可以通过CFRunLoopSourceRef实现多个回调函数。

计时器则是这样一种机制：基于时间进行通知应用程序在未来某个特定时间点执行某个特定的任务的。计时器事件也是通过异步的方式派发到程序中的，不过它还与特定的模式相关(下一节将介绍相关模式)。如果并不是当前监听的特定模式，这个计时器时间会被忽略，而线程也不会受到通知，直到run llop运行在相应的模式中。

####Run Loop模型

这里的相关内容没有翻译。相关更多资料请看这里：[`RunLoopManagement`](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html)

###**小结**

对于iOS开发者来说，理解iOS网络框架中的各层，以及应用程序如何与run loop交互是非常重要的。一个优秀的网络架构层会给应用程序提供难以置信的灵活度。如果网络架构层的设计非常糟糕，那么这是很难获得成功和扩展能力的。

本章预览了一下各个网络APIs，并做了一些比较。在这里只是简单的介绍了一下，在后续章节中，会深入讨论。


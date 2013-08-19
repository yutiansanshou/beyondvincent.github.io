---
layout: post
title: "Professional iOS Network Programming翻译第二章：设计服务端的架构"
date: 2013-08-17 16:52
comments: true
categories: iOS翻译
published: false

---
{% img /images/2013/07/simple_social_network.png 400 300 %}


## **小引**

（）

<!--more-->

###**目录**
* 第二章：设计服务端的架构
	* 远程外观模式(Remote Facade Pattern)
		* 服务器端示例
		* 客户端示例
	* 服务的版本
		* 版本服务示例
		* 使用版本服务的客户端示例
	* 服务定位器
	* 小结



##第二章：**设计服务端的架构**

本章的内容

	实现一个远程外观模式
	利用服务定位器寻找endpoint
	利用服务版本功能实现对老版本程序的支持

本章代码下载

可以在[这里](http://www.wrox.com/WileyCDA/WroxTitle/Professional-iOS-Network-Programming-Connecting-the-Enterprise-to-the-iPhone-and-iPad.productCd-1118362403.html)的Download Code选项中可以下载到本章相关代码。本章共有两个代码示例，一个是示例工程，另外一个是web services：

* Facade Tester.zip
* Facade PHP.zip

可以说，Web service是iOS网络应用程序的命脉，Web service设计的灵活性和鲁棒性对于用户体验有非常重要的影响。对于优秀设计的web service APIs，内部变动了，并不会影响到使用这些APIs的程序。而服务定位器功能则可以让程序能够动态的发现并使用新的serviceendpoint，其中并不需要重新编译和提交程序到商店中。当需要重新提交程序时，在程序过渡和升级过程中，web service需要保持对老版本程序的支持。service API对各个版本的支持是非常重要的，这样当service升级时，老版本的程序并不需要重新升级程序版本。本章将对相关内容进行介绍。


###**远程外观模式(Remote Facade Pattern)**

在为程序设计服务端的架构时，远程外观模式简化了应用程序对web service的集成，并允许多个客户端共享相同的业务逻辑。外观模式对内部复杂的子系统进行封装，并供客户端更加容易的使用该子系统提供的功能。例如，邮政系统包含成千上万的邮递员、卡车、飞机、配送中心和邮局，然而对于用户来说，需要将其中复杂和简单的信件和包裹的收发细节进行隐藏，用户并不需要知道一封信件是如何从纽约到达旧金山的，用户只需要支付邮费，然后等待信件到达即可。同理，程序中的API也可能需要将多个数据库的查询、以及后端系统请求封装为一个外部可以访问的方法——返回操作的结果。只要保持向别的程序暴露的外部API约定不变，那么底层系统可以进行变更、升级或者完全删除不会影响客户端的任意内容。



####服务器端示例

####客户端示例


###**服务的版本**

####版本服务示例

####使用版本服务的客户端示例

###**服务定位器**


###**小结**


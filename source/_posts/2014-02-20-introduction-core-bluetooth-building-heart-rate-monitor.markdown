---
layout: post
title: "Core Bluetooth介绍：构建一个心率监测器"
date: 2014-02-20 20:20
comments: true
categories: iOS探索
published: false

---

![](/images/2014/02/1.png)

<!--more-->

* 蓝牙中的中心和外设
* 中心如何与外设通讯
* 外设数据的结构
* CBPeripheral CBService 和 CBCharacteristic
* 开始
* 相关Delegate
* 用户界面的构建
* 利用Bluetooth框架
* 实现Delegate方法
	* 添加centralManagerDidUpdateState:central
	* 添加didDiscoverPeripheral:peripheral:
	* 添加centralManager:central:peripheral:
	* 添加peripheral:didDiscoverServices:
	* 添加peripheral:didDiscoverCharacteristicsForService:
	* 添加peripheral:didUpdateValueForCharacteristic:
	* 添加getHeartRateBPM:(CBCharacteristic *)characteristic error:(NSError *)error
	* 添加getManufacturerName:(CBCharacteristic *)characteristic
	* 添加getBodyLocation:(CBCharacteristic *)characteristic
* 让心跳再快一点
* 何去何从？

在iOS和Mac程序开发中，通过Core Bluetooth可以与低功耗的蓝牙设备(BLE)通信。BLE包括心率监测器、数字温度计等。

Core framework 根据[蓝牙4.0的规范](https://www.bluetooth.org/en-us/specification/adopted-specifications)做了封装，并定义了一套易于使用的protocol——用于与BLE设备通讯。

通过本文，可以学习关于Core Bluetooth框架的关键内容，以及如何利用框架搜索和连接至BLE设备，并获取到相关数据。本文我将构建一个心率监测程序，让其与蓝牙心率监测器通讯。

本文使用的心率监测器是[Polar H7 Bluetooth Smart Heart Rate Sensor](http://amzn.to/1h6RjLY)。如果你没有这种蓝牙设备的话，也可以继续学习本文，只不过在自己使用的时候，有些代码需要微调一下。

#蓝牙中的中心和外设

在所有的Bluetooth Le通讯中又两个主要的角色：中心和外设：

* `中心`可以理解为boss。他希望从一帮工人手里获取到一些信息，已完成特定的任务。
* `外设`可以理解为worker。它负责收集和发布数据。

下面这个图可以看到它们之间的关系：

![](/images/2014/02/3.png)

在上面的场景中，一台iOS设备(中心)与心率监测器(外设)通讯，以获取到相关数据，并显示到程序界面上。

#中心如何与外设通讯



#外设数据的结构

#CBPeripheral CBService 和 CBCharacteristic
#开始

#相关Delegate

#用户界面的构建

#利用Bluetooth框架

#实现Delegate方法

##添加centralManagerDidUpdateState:central

##添加didDiscoverPeripheral:peripheral:

##添加centralManager:central:peripheral:

##添加peripheral:didDiscoverServices:

##添加peripheral:didDiscoverCharacteristicsForService:

##添加peripheral:didUpdateValueForCharacteristic:

##添加getHeartRateBPM:(CBCharacteristic *)characteristic error:(NSError *)error

##添加getManufacturerName:(CBCharacteristic *)characteristic

##添加getBodyLocation:(CBCharacteristic *)characteristic

#让心跳再快一点

#何去何从？

至此，你已经学习了Core Bluetooth LE相关内容，以及如何使用其来连接到低功耗蓝牙外设，并获取到相关数据。

关于Core Bluetooth设备的另外一个示例是iBeacons。如果你感兴趣的话，可以看看[iOS 7 by Tutorials](http://www.raywenderlich.com/store/ios-7-by-tutorials)中的`What's New in Core Location`章节。

可以来这里下载本文涉及到的[完整示例工程](http://cdn4.raywenderlich.com/wp-content/uploads/2013/12/HeartMonitor-Final.zip)。









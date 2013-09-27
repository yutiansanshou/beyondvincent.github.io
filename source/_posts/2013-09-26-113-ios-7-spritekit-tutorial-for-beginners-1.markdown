---
layout: post
title: "Sprite Kit初级教程(1)"
date: 2013-09-26 11:45
comments: true
categories: iOS翻译
published: ture

---

{% img /images/2013/09/6.png %}
<!--more-->



注：本文译自[`Sprite Kit Tutorial for Beginners`](http://www.raywenderlich.com/42699/spritekit-tutorial-for-beginners)



###**目录**
* `Sprite Kit的优点和缺点`
* `Sprite Kit vs Cocos2D-iPhone vs Cocos2D-X vs Unity`
* `Hello, Sprite Kit!`
* 横屏显示
* 移动怪兽
* 发射炮弹
* 碰撞检测: 概述
* 碰撞检测: 实现
* 收尾
* 何去何从?




在iOS 7中内置了一个新的Sprite Kit框架，该框架主要用来开发2D游戏。目前已经支持的内容包括：精灵、很酷的特效(例如视频、滤镜和遮罩)，并且还集成了物理库等许多东西。

iOS 7中附带了一个非常棒的Sprite Kit示例工程，名字叫做Adventure。不过这个示例工程稍微有点复杂，不太适合初学者。本文的目的就是做一个关于Sprite Kit使用的初级教程。

通过本文，你可以从头到尾的学习到如何为你的iPhone创建一个简单又有趣的2D游戏。如果你看过我们之前的教程:[Simple Cocos2D game教程](http://www.raywenderlich.com/25736/how-to-make-a-simple-iphone-game-with-cocos2d-2-x-tutorial)，你会发现非常的相似。

在开始之前，请确保已经安装了最新版本的Xcode(5.X)，里面支持Sprite Kit以及iOS 7。


###** Sprite Kit的优点和缺点**

首先，我想指出在iOS中开发2D游戏Sprite Kit并不是唯一的选择，下面我们先来看看Sprite Kit的一些优点和缺点。

Sprite Kit的优点：

 1、它是内置到iOS中的，因此并不需要下载额外的库或者其它一些外部依赖。并且它是由苹果开发的，所以对于它的支持和更新我们可以放心。
 
 2、它内置的工具支持纹理和粒子。
 
 3、它可以让你做一些其它框架很难做到的事情，例如把视频当做精灵一样处理，或者使用很酷的图形效果和遮罩。
 
Sprite Kit的缺点：

 1、如果使用了Sprite Kit，那么你将被iOS生态圈所绑架，导致你无法很容易对你开发的游戏移植到Android上。
 
 2、Sprite Kit现在还处于初始阶段，此时提供的功能还没有别的框架丰富，例如Cocos2D。最缺的东西应该是暂不支持写自定义的OpenGL代码。
 

###** Sprite Kit vs Cocos2D-iPhone vs Cocos2D-X vs Unity**

此时，你可能在想“我该选择使用哪个2D框架呢？”

这取决于你的实际情况，下面是我的一些想法：

 1、如果你是一个初学者，并且只关注于iOS，那么就使用内置的Sprite Kit吧，它非常容易学习，并且完全可以把工作做好。
 
 2、如果需要写自己的OpenGL代码，那么还是使用Cocos2D，或者其它框架吧，目前Sprite Kit并不支持自定义OpenGL代码。
 
 3、如果要进行跨平台开发，那么选择Cocos2D-X或者Unity。Cocos2D-X非常出色，可以用它来构建2D游戏。Unity则更加的灵活(例如，如果有需要的话，你可以在游戏中添加一些3D效果)。
	
看到这里，如果你还想要继续了解Sprite Kit的话，请继续往下读吧。


###** Hello，Sprite Kit！**

下面我们就开始利用Xcode 5内置的Sprite Kit模板来构建一个简单的Hello World工程吧。

启动Xcode，选择`File\New\Project`，接着选中`iOS\Application\SpriteKit Game`模板，然后单击`Next`：

{% img /images/2013/09/7.png %}

输入Product Name为`SpriteKitSimpleGame`，Devices选择iPhone，接着单击`Next`：

{% img /images/2013/09/8.png %}

选择工程保存的路径，然后点击`Create`。然后点击Xcode中的播放按钮来运行工程。稍等片刻，可以看到如下运行画面：

{% img /images/2013/09/9.png %}

跟Cocos2D类似，Sprite Kit也是按照`场景(scenes)`来构建的，这相当于游戏中的"levels"和"screens"。例如，你的游戏中可能会有一个主游戏区的场景，以及一个世界地图的一个场景。

如果你观察一下创建好的工程，会发现SpriteKit Game模板已经创建好了一个默认的场景`MyScene`。现在打开`MyScene.m`，里面已经包含了一些代码，其中将一个lable放到屏幕中，并且添加了：当tap屏幕时，会在屏幕上新增一个旋转的飞船。

在本教程中，我们主要在MyScene中写代码。不过在开始写代码之前，需要进行一个小调整——让程序以横屏的方式运行。

……Sprite Kit初级教程(1)结束……

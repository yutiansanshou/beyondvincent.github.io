---
layout: post
title: "iOS 7 教程：浅析Text Kit"
date: 2013-11-12 12:55
comments: true
categories: iOS7
published: true

---

![](/images/2013/11/22.png)

<!--more-->



注：本文由破船原创，并投稿至[Tiny4Cocoa iOS开发杂志](http://tiny4cocoa.com/thread/show/57/)。

Text Kit是iOS 7中引入的一个新功能，非常值得开发者使用，下面先看看本文的目录结构：

1. [什么是Text Kit](#1)
2. [Text Kit架构](#2)
3. [Text Kit特点](#3)
4. [Text Kit功能概述](#4)
5. [Text Kit中重要的一些对象](#5)
6. [Text Kit示例](#6)
7. [小结](#7)
8. [推荐Text Kit学习资源](#8)


###<a id="1"></a>什么是Text Kit

在iOS7中，苹果引入了`Text Kit`——Text Kit是一个快速而又现代化的文字排版和渲染引擎。Text Kit在UIKit framework中的定义了一些类和相关协议，它最主要的作用就是为程序提供文字排版和渲染的功能。在程序中，通过Text Kit可以对文字进行存储(store)、布局(lay out)，以及用最精细的排版方式(例如文字间距、换行和对齐等)来显示文本内容。
苹果引入Text Kit的目的并非要取代已有的Core Text，Core Text的主要作用也是用于文字的排版和渲染中，它是一种先进而又处于底层技术，如果我们需要将文本内容直接渲染到图形上下文(Graphics context)时，从性能和易用性来考虑，最佳方案就是使用Core Text。而如果我们直接利用苹果提供的一些控件(例如UITextView、UILabel和UITextField等)对文字进行排版，无疑就是借助于UIkit framework中Text Kit提供的API。


###<a id="2"></a>Text Kit架构

下面，我们通过图1(此图来自WWDC2013 Session 210)来了解一下Text Kit的架构。图1是基于iOS 7绘制的，从图中，我们可以看到Text Kit是基于Core Text构建的，它通过Core Text与Core Graphics进行交互。而UI控件(UILabel、UITextField和UITextView)则构建于Text Kit之上，可见这些文本控件可以利用Text Kit提供的API来对文字进行排版和渲染处理。
从图中我们也可以看到SDK提供的UIWebView是基于WebKit的，它不能使用Text Kit提供的功能。

![图1 Text Kit在iOS 7 SDK中的位置](/images/2013/11/23.jpg)

我们再来看看图1中的相关组件在iOS6里面是如何对应的，如图2所示，可以看出在iOS 6中是没有Text Kit，并且UILabel、UIText和UITextView是基于String Drawing和WebKit构建的。其中String Drawing是与Core Graphics直接通讯。

![图2 在iOS 6中并没有Text Kit](/images/2013/11/24.jpg)



###<a id="3"></a>Text Kit特点

从上面的介绍中，我们可以了解到Text Kit在UIKit中的作用非常重要。Text Kit在实际开发中具有如下特点：

* 在UI控件中Text Kit完全掌控着文字的排版和渲染
* UITextView、UITextField和UILabel是构建于Text Kit之上的
* 能够与动画、UICollectionView和UITableView做到无缝集成
* Text Kit具有这样一些能力：Subclassing、Delegation和Notifcation。

###<a id="4"></a>Text Kit功能概述

下面我们看看通过Text Kit，都能实现那些功能(这里列出了是一些常用和重要功能)：

* 对文字进行分页或多列排版
* 支持文字的换行、折叠和着色等处理
* 可以调整字与字之间的距离、行间距、文字大小、指定特定的字体
* 支持富文本编辑，可以自定义文字截断
* 支持凸版印刷效果(letterpress)
* 支持数据类型的检测(例如链接、附件等)

如图3，是利用Text Kit对文字做的分页排版

![图3 利用Text Kit做的分页排版效果](/images/2013/11/25.jpg)

再看图4，是利用Text Kit做的换行处理，其中对某个路径范围做了排除。

![图4 利用Text Kit做的换行处理效果](/images/2013/11/26.jpg)

再来看看利用Text Kit做的凸版印刷效果，如图5所示

![图5 利用Text Kit做的凸版印刷效果](/images/2013/11/27.jpg)


###<a id="5"></a>Text Kit中重要的一些对象

下面我们来看看Text Kit中重要的几个对象。

![图6 Text Kit中重要的几个对象](/images/2013/11/28.jpg)

如图6所示，Text Kit中主要有4个重要的对象。

* Text View是用来显示文本内容的控件，主要包括UILabel、UITextView和UITextField。
* Text containers对应着NSTextContainer类。NSTextContainer定义了文本可以排版的区域。一般来说，都是矩形区域，当然，也可以根据需求，通过子类化NSTextContainer来创建别的一些形状，例如圆形、不规则的形状等。NSTextContainer不仅可以创建文本可以填充的区域，它还维护着一个数组——该数组定义了一个区域，排版的时候文字不会填充该区域，因此，我们可以在排版文字的时候，填充非文本元素(例如图片，如图4所示)。
* Layout manager对应着NSLayoutManager类。该类负责对文字进行编辑排版处理——通过将存储在NSTextStorage中的数据转换为可以在视图控件中显示的文本内容，并把统一的字符编码映射到对应的字形(glyphs)上，然后将字形排版到NSTextContainer定义的区域中。
* Text storage对应着NSTextStorage类。该类定义了Text Kit扩展文本处理系统中的基本存储机制。NSTextStorage继承自NSmutableAttributedString，主要用来存储文本的字符和相关属性。另外，当NSTextStorage中的字符或属性发生了改变，会通知NSLayoutManager，进而做到文本内容的显示更新。

通常情况下，NSTextStorage、NSLayoutManager和NSTextContainer是一一对应的。如图7所示关系：

![图7 普通排版](/images/2013/11/29.jpg)

当然，如果需要将文字显示为多列，或多页，可以按照如图8所示关系——使用多个NSTextContainer。

![图8 多页或者多列排版](/images/2013/11/30.jpg)

如果针对不同的排版方式，则可以使用多个NSLayoutManager，如图9所示

![图9 不同的排版方式](/images/2013/11/31.jpg)

如图10所示，通过形象的方式，对UITextView的组成做了分解。通常，我们在设备上只能看到最右边的文本显示界面，而内部的NSTextStorage、NSLayoutManager和NSTextContainer是看不出来的。通常由NSLayoutManager从NSTextStorage中读取出文本数据，然后根据一定的排版方式，将文本排版到NSTextContainer中，再由NSTextContainer结合UITextView将最终效果显示出来。

![图10 UITextView的分解](/images/2013/11/32.jpg)


###<a id="6"></a>Text Kit示例

前面对Text Kit做了一些介绍，下面我们配合一个例子(图文排版)，来进一步加深对Text Kit的认识。具体实现步骤如下

1. 打开Xcode 5，新建一个Single View Application模板的程序，将工程命名为ExclusionPath。
2. 打开Main.storyboard文件，然后再默认View Controller的View里面分别添加一个UITextView和UIImageView。并将这两个控件连接到ViewController.h中(名称分别为textView何imageView)。然后给textView设置一些字符串，imageView设置一个图片。
3. 打开ViewController.m文件，找到viewDidLoad方法，用如下代码替换该方法：

```objc
- (void)viewDidLoad
{
    [super viewDidLoad];
    
    //创建一个平移手势对象，该对象可以调用imagePanned：方法
    UIPanGestureRecognizer *panGes = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(imagePanned:)];
    [self.imageView addGestureRecognizer:panGes];
    
    self.textView.textContainer.exclusionPaths = @[[self translatedBezierPath]];
}
```

在上面的代码中，给imageView添加了一个平移手势。另外通过调用translatedBezierPath方法，给textView的textContainer设置exclusionPaths属性值。表示需要排除的区域（也就是图片在排版中显示的位置）。

4. 下面来看一下translatedBezierPath方法的实现，如下代码所示

```objc
- (UIBezierPath *)translatedBezierPath
{
    CGRect butterflyImageRect = [self.textView convertRect:self.imageView.frame fromView:self.view];
    UIBezierPath *newButterflyPath = [UIBezierPath bezierPathWithRect:butterflyImageRect];
    
    return newButterflyPath;
}
```

在上面的代码中，利用imageView的frame属性创建了一个UIBezierPath，然后将该值返回。
5. 还记得第3步中创建的平移手势吗。里面有一个action需要实现imagePanned:，下面来看看这个方法的实现：

```objc
- (void)imagePanned:(id)sender
{
    if ([sender isKindOfClass:[UIPanGestureRecognizer class]]) {
        UIPanGestureRecognizer *localSender = sender;
        
        if (localSender.state == UIGestureRecognizerStateBegan) {
            self.gestureStartingPoint = [localSender translationInView:self.textView];
            self.gestureStartingCenter = self.imageView.center;
        } else if (localSender.state == UIGestureRecognizerStateChanged) {
            CGPoint currentPoint = [localSender translationInView:self.textView];
            
            CGFloat distanceX = currentPoint.x - self.gestureStartingPoint.x;
            CGFloat distanceY = currentPoint.y - self.gestureStartingPoint.y;
            
            CGPoint newCenter = self.gestureStartingCenter;
            
            newCenter.x += distanceX;
            newCenter.y += distanceY;
            
            self.imageView.center = newCenter;
            
            self.textView.textContainer.exclusionPaths = @[[self translatedBezierPath]];
        } else if (localSender.state == UIGestureRecognizerStateEnded) {
            self.gestureStartingPoint = CGPointZero;
            self.gestureStartingCenter = CGPointZero;
        }
    }
}

```

在上面的代码中首先根据平移的距离来设置imageView的位置，然后利用translatedBezierPath方法重新计算了一下排除区域。
6. 至此代码编写完毕，下面来运行程序，看看实际效果。如图11所示：

![图11 运行效果](/images/2013/11/33.gif)

7. 点击下图，下载代码

[![](/images/2013/11/34.jpg)](https://github.com/BeyondVincent/iOS7-new-feature/tree/master/code/TextKit/ExclusionPath)


###<a id="7"></a>小结

实际上，上面的示例，只是揭秘了Text Kit功能的冰山一角。从iOS7及以后的版本中，Text Kit在UIKit framework里面占据重要的地位，Text Kit在文字处理方面，具有非常强大的功能，并且开发者可以对Text Kit进行定制和扩展。据悉，苹果利用了2年的时间来开发Text Kit，相信这对许多开发者来说都是福音。


###<a id="8"></a>推荐Text Kit学习资源

更多关于Text Kit的学习资料，请参考下面的内容：

wwdc视频:

* Introducing Text Kit
* Advanced Text Layouts and Effects with Text Kit
* Using Fonts with Text Kit

苹果官方参考文档：

* Text Programming Guide for iOS.pdf
* NSLayoutManager Class Reference for iOS.pdf
* NSLayoutManagerDelegate Protocol Reference for iOS.pdf
* NSTextContainer Class Reference for iOS.pdf
* NSTextStorage Class Reference for iOS.pdf
* NSTextStorageDelegate Protocol Reference for iOS.pdf

苹果官方示例：

* IntroToTextKit


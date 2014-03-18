---
layout: post
title: "提高Interface Builder高效工作的8个技巧"
date: 2014-03-19 00:10
comments: true
categories: iOS探索
published: true

---

![](/images/2014/03/21.png)

<!--more-->

本文译自：[8 Tips for working effectively with Interface Builder](http://codesheriff.blogspot.com/2014/03/8-tips-for-working-effectively-with.html)(需翻墙)

先来看看目录：

0. 介绍
1. 使view的Size与view中的Content相适应 
2.  按住option键—观察所选中view与另外view边缘之间的距离
3. Editor -> Embed In View, Unembed:
4. 在不影响subview的位置时给view自由的添加padding
5. 对不在最前端的view进行移动
6. IBOutletCollection排序
7. 使用自定义属性
8. MoarFonts——字体定制：所见即所得

##介绍##

在JoyTunes工作期间，我们在开发最新一版的钢琴应用程序，对程序的UI做了大量的重新设计，因而也在Interface Builder上花费了许多时间，对于图片和view的缩放操作，非常的让人不省心。不过在开发过程中，我们发现了许多非常不错的IB使用技巧，我寻思着这必须要跟大家分享，所以成就了这篇文章。


免责声明：
在JoyTune工作期间，我们使用的是.xib文件(不是storyboards)，并且没有使用Auto Layout。实际上这主要是历史原因导致的。所以，这里介绍的一些技巧可能稍微有点不同(如果你使用storyboard或Auto Layout)，不过大多数都是一样的。

##1. 使view的Size与view中的Content相适应##


很惭愧的是最近才发现这个功能——能节约大量时间。
选中任意的一个view，然后Editor->Size to Fit Content，或者简单的按 ⌘=
接着就会按照下面的规则对选中view的Size做出与之Content对应的适应。



   * ImageView/Button的size会设置为图像的原始size(最常见的用法)：
   
![](/images/2014/03/12.gif)

   * Label/Button的size会被设置为与当前text内容相当的尺寸：
   
![](/images/2014/03/13.gif)

   * parent container view会与其subviews的frames相适应。

![](/images/2014/03/14.gif)


##2.  按住option键—观察所选中view与另外view边缘之间的距离##

按住option键之后，选择一个view，然后将鼠标悬停在别的一些view上，会看到一些距离——选中view与别的view边缘之间的距离。

![](/images/2014/03/15.gif)

##3. Editor -> Embed In View, Unembed:##

你是不是对此素手无策呢：你希望将已有的一些subviews放入到不同的parent view中，甚至是不同的.xib文件中，但是当你把一些view重新设置之后，它们为自动的位于新的parent view中心？

现在好了，我们有一个解决办法，如下图所示：

![](/images/2014/03/16.gif)

##4. 在不影响subview的位置时给view自由的添加padding##

当试图给view添加padding时，默认情况下subview的x和y是不会改变的，但是有时候我们并不希望是这样的结果。我发现一个最好的方法，就是在按住⌘时拖动view的边缘：

![](/images/2014/03/17.gif)

##5. 对不在最前端的view进行移动##

刚开始我还以为要想移动不在最前端的view是不可行的。

有一种方法就是先将非最前端的view临时设置到最前端，移动好位置之后，在设置回去。

另外一种方法就是使用右边panel中的size inspector，不过有时候要想设置一个好的位置，需要不断的猜测和修正。

另外我发现一种方法：使用键盘上的上下左右键来移动view——这还不用把view设置为最前端：

   * 在document outline中选中view
   * 为了获得view的焦点：单击root view的frame
   * 利用箭头进行移动
   
![](/images/2014/03/18.gif)

提醒：
获得view的焦点还有一个更好的方法：在document outline上双击view，就可以用箭头移动view了。

##6. IBOutletCollection排序##

有时候IBOutletCollection里面元素的顺序对我们来说非常重要：我们希望按某个顺序对其进行迭代。

有一种方法：在代码里面利用x/y/tag对其做排序处理，然后在迭代。

实际上，没必要这么做。IBOutletCollection的顺序取决于我们dragged connection的顺序，可以通过^+单击 File's Owner来查看当前的顺序：

![](/images/2014/03/19.gif)

##7. 使用自定义属性##

可能这个功能是IB中很少被使用的：使用Identity inspector中的User Defined Runtime Attributes(用户自定义运行时属性)在view上设置自定义属性：

![](/images/2014/03/20.gif)

在此我定义了一个JTLabel类，我们可以设置它的stroke color和width，这样一来我们就不用在代码里面设置相关属性了。

利用这个功能很好的一例子就是[Canvas](http://canvaspod.io/)，通过它不用写一行代码就能定义相关的动画。

##8. MoarFonts——字体定制：所见即所得##

在Interface Builder中字体的定制是个非常麻烦的事情。IB并没有内置该功能，我用过比较好的解决办法就是使用自定义属性——就像Canvas一样，或者使用字体替换技术——例如[IBCustomFonts](https://github.com/deni2s/IBCustomFonts)。这些都是有效的方案，不过他们有一个致命的缺点——它们不能给我们一种WYSIWYG(所见即所得)的体验，当然，这也是为什么我们会第一时间使用Interface Builder的原因。

为了知道给label设置的自定义字体是否合适，我们必须要运行程序才能知道结果——这有点让人不能接受。

最近我发现了一个新的解决办法：使用[MoarFonts](http://pitaya.ch/moarfonts/)。卖价10美元，没有demo，没有试用——不过请相信，这非常值得购买！它的使用方法非常简单：将MoarFonts当做script build phase，然后build app，接着重启Xcode，就可在Interface Builder中看到定制的字体。

打完收工！希望这些技巧对你能有所帮助。
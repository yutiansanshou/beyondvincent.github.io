---
layout: post
title: "Xcode 5中的Interface Builder更有利于团队协作开发"
date: 2013-09-04 11:45
comments: true
categories: iOS翻译
published: ture

---


## **小引**

在iOS开发中，开发者有各种理由选择用代码来构建界面，其中最多的理由就是Interface Builder绘制的代码不利于团队间协作(代码的可读性和合并)。虽然Interface Builder来绘制界面有诸多优势，但是由于致命的缺陷，许多开发团队不得不远离它，当然也有一些开发团队为了在协同开发时也使用IB来绘制界面，他们尽量确保同一时期只由一个人来操作某个xib文件，以此避免提交代码的时候需要合并xib文件。然而，在Xcode 5中，苹果的开发团队已经对xib文件格式做了大量的简化工作。本文就来简要的看看相关内容。

<!--more-->

注：本文译自[`Xcode 5 Finally Makes Interface Builder a Viable Option for Teams`](http://nilsou.com/blog/2013/08/07/xcode-5-finally-makes-interface-builder-a-viable-option-for-teams/)

###**目录**
* Xcode 4中xib文件的格式
* Xcode 5中xib文件的格式
* 小结


上周我将代码merge到我的working branch时，注意到以前没曾见过的提示：

{% img /images/2013/09/1.png%}

Git会自动合并**xib文件**！我在想，Xcode 5中肯定对xib文件的格式做了修改，所以我准备深入研究一下。

原来由Xcode 5生成的xib文件是一种全新的格式。看起来苹果这次对xib文件格式的变更将有助于开发团队开始考虑使用IB来绘制界面。


###*** Xcode 4中xib文件的格式**

为了演示xib文件格式在Xcode 4和Xcode 5中的差异，我首先在Xcode 4中创建了一个新的xib文件，并添加了一些view进去：一个UIScrollView，该scrollview中包含一些UIButton、UILable和TextView等。

{% img /images/2013/09/2.png%}

然后我在工程导航窗口中右键单击该文件，并选择`Open As > Source Code`。下面的链接中是看到的内容：

[Xcode 4生成的.xib文件](https://gist.github.com/nilsou/6057457)(需要点击“File suppressed. Click to show.”)

可以看到，这么简单的一个view居然有`1108行`代码！这太多了。

对开发者来说它的可读性非常的糟糕。这是开发者为什么不喜欢xib文件的主要原因(由此放弃使用interface builder)。

更糟糕的是这个xib文件的格式还具有不确定性。也就是说如果我在Interface Builder创建相同的UI界面，但是我们看到的文件内容并不是相同的。这就导致xib文件的合并非常的困难，甚至不可能进行合并。这事开发者不使用xib文件的另外一个重要原因。


###*** Xcode 5中xib文件的格式**

接着，我在Xcode 5中打开同一个工程。当在Xcode 5中打开用Xcode 4创建的xib文件时，会提示将文件升级到新的格式。这里需要注意的是升级之后的文件只能在Xcode 5中打开，这种新格式的文件不能在老版本中的Xcode中打开。所以，如果是团队协作开发，那么升级的时候，需要确保所有的开发者都使用Xcode 5。

{% img /images/2013/09/3.png%}

我点击`Upgrade`，然后再次打开xib文件的source code，看看有什么变化。如下链接中所示：

[Xcode 5生成的.xib文件](https://gist.github.com/nilsou/6057474)(需要点击“File suppressed. Click to show.”)

`133行`！这与Xcode 4中创建的xib文件相差约10倍。可见苹果的开发团队已经对xib文件格式做了大量的简化。

再看看里面的具体内容，可以看出它的可读性也加强了。xib文件中的源代码现在也能够反应出view的层次(Interface Builder左边看到的内容结构！)，等熟悉之后，开发者可以直接对这个xml代码进行编写。

最重要的一点，可以看出Xcode 5生成的xib文件内容源码位置是确定的。这非常利于文件的合并。

###**小结**

这种新的格式带来的最大好处不仅仅是增强开发者对xib文件的可读性，另外在大多数情况下，git还可以对xib文件进行自动合并，不用开发者手动进行。

现在如果还有开发者告诉你他不想用xib文件，那么请把这篇文章发给他看看吧，我相信已经没有太多理由不使用xib文件了。

其实在Xcode 5中不仅对Interface Builder进行了改善，还有其它一些功能也做了改进，例如自动布局约束的设置已经没有以前痛苦了。




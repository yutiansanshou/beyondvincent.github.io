---
layout: post
title: "iOS 7 教程：让程序同时支持iOS 6和iOS 7"
date: 2013-11-19 22:18
comments: true
categories: iOS7
published: true

---

![](/images/2013/11/35.png)

<!--more-->

注：本文由破船译自[Itty Bitty Labs](http://blog.ittybittyapps.com/blog/2013/11/08/working-with-ios-6-and-7/)。


1. [iOS 7中的布局问题](#1)
2. [iOS 6运行异常](#2)
3. [Xcode 4编译错误](#3)
4. [UILabel不一致的background](#4)
5. [全屏时隐藏状态栏](#5)
6. [UIToolbar barStyle](#6)
7. [更多](#7)

由于各种原因，我们的程序需要同时支持iOS 7以及之前的版本(例如iOS 6)，也就是说开发者不得不同时在iOS 7和iOS 6之间进行开发。实际上开发者对此是比较讨厌的。

###<a id="1"></a>iOS 7中的布局问题

下面是非常简单的一个程序，运行在iOS 6中的界面：

![](/images/2013/11/36.png)

而要是运行在iOS 7的模拟器中，会看不到label了：

![](/images/2013/11/37.png)

这是为什么呢？我们对其reveal一下看看吧：

![](/images/2013/11/38.png)

从上图可以看出，实际上label躲在NavigationBar后面了。在iOS 7中，苹果引入了一个新的属性，叫做`[UIViewController setEdgesForExtendedLayout:]`，它的默认值为`UIRectEdgeAll`。当你的容器是navigation controller时，默认的布局将从navigation bar的顶部开始。这就是为什么所有的UI元素都往上漂移了44pt。

修复这个问题的快速方法就是在方法`- (void)viewDidLoad`中添加如下一行代码：

```objc
self.edgesForExtendedLayout = UIRectEdgeNone;
```

这样问题就修复了。

![](/images/2013/11/39.png)


###<a id="2"></a>iOS 6运行异常

现在如果在iOS 6中运行程序，会遇到下面这样的运行时异常错误：

```objc
[LAViewController setEdgesForExtendedLayout:]: unrecognized selector sent to instance 0x778a210
```

所有只能在iOS 7中运行的API需要重新封装一下，如下代码所示：

```objc
if ([self respondsToSelector:@selector(setEdgesForExtendedLayout:)])
{
    self.edgesForExtendedLayout = UIRectEdgeNone;
}
```


###<a id="3"></a>Xcode 4编译错误

有些机器可能还在使用Xcode 4.6，当用4.6来编译代码时，会遇到下面的编译错误：

```objc
Property 'edgesForExtendedLayout' not found on object of type 'LAViewController *'
Use of undeclared identifier 'UIRectEdgeNone'
```

为了避免这个错误，可以创建下面的这个宏：

```objc
#if __IPHONE_OS_VERSION_MAX_ALLOWED >= 70000
#define IOS7_SDK_AVAILABLE 1
#endif
```
然后在需要的地方将iOS 7的代码包装一下即可：

```objc
#ifdef IOS7_SDK_AVAILABLE
...
#endif
```

###<a id="4"></a>UILabel不一致的background

对于UILabel，在iOS 7中它的background颜色默认是clearColor，而在iOS 6中默认的是白色。所以，我们最好在代码中对label的background颜色进行明确的设置：

```objc
view.backgroundColor = [UIColor clearColor];
```

###<a id="5"></a>全屏时隐藏状态栏

在iOS 6中，当调用`presentViewController`时，默认的modal screen将是全屏(`UIModalPresentationFullScreen`)。为了在iOS 7中也能获得相同的效果，我们可以在modal controller中添加如下代码：

```objc
- (BOOL)prefersStatusBarHidden
{
  return YES;
}
```

###<a id="6"></a>UIToolbar barStyle

有时候，我们会将UIToolbar与系统键盘结合起来使用。而在iOS 6中的键盘是黝黑色的，此时toolbar的style一般也是类似的，如下代码所示：

```objc
self.barStyle = UIBarStyleBlack;// or UIBarStyleBlackTranslucents
```

而在iOS 7中，键盘变为了亮色，因此我们需要根据不同的iOS 版本，设置不同的bar style。

```objc
if ([[[UIDevice currentDevice] systemVersion] compare:@"7.0" options:NSNumericSearch] != NSOrderedAscending)
{
    self.barStyle = UIBarStyleDefault;
}
else
{
    self.barStyle = UIBarStyleBlack;//or UIBarStyleBlackTranslucent
}
```

###<a id="7"></a>更多

上面这些技巧是我目前在开发中遇到的，肯定还有更多的技巧，大家要是知道的话可以告诉我。

最后送大家一个图，看看相关差异吧：


![](/images/2013/11/40.png)

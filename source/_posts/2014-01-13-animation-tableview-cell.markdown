---
layout: post
title: "给tableview cell添加动画"
date: 2014-01-13 17:30
comments: true
categories: iOS探索
published: true

---

![](/images/2014/01/5.png)

<!--more-->

###小引###

本文介绍如何利用给tableview cell添加动画。其实只需要很少的代码量就可以。本文参考[Animating UITableView cells](http://www.thinkandbuild.it/animating-uitableview-cells/)

下面先来看看最终的效果：

<iframe height=498 width=510 src="http://player.youku.com/embed/XNjYxMTgxOTQw" frameborder=0 allowfullscreen></iframe>

从上面的视频中，可以看出，当cell显示出来的时候，是在Y和Z轴上进行3D旋转。

下面来看看是如何实现的：

首先假设你已经能够熟练使用UITableView了。那么我们只需要实现UITableViewDelegate中的tableView:WillDisplayCell:ForRowAtIndexPath:即可。当cell显示之前，会先调用该方法，因此给cell添加动画，在这个方法里面即可。

如下代码所示：

```objc 
-(void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath{
    // 1. 配置CATransform3D的内容
    CATransform3D transform;
    transform = CATransform3DMakeRotation( (90.0*M_PI)/180, 0.0, 0.7, 0.4);
    transform.m34 = 1.0/ -600;
    
    // 2. 定义cell的初始状态
    cell.layer.shadowColor = [[UIColor blackColor]CGColor];
    cell.layer.shadowOffset = CGSizeMake(10, 10);
    cell.alpha = 0;
    
    cell.layer.transform = transform;
    cell.layer.anchorPoint = CGPointMake(0, 0.5);
    
    // 3. 定义cell的最终状态，并提交动画
    [UIView beginAnimations:@"transform" context:NULL];
    [UIView setAnimationDuration:0.5];
    cell.layer.transform = CATransform3DIdentity;
    cell.alpha = 1;
    cell.layer.shadowOffset = CGSizeMake(0, 0);
    cell.frame = CGRectMake(0, cell.frame.origin.y, cell.frame.size.width, cell.frame.size.height);
    [UIView commitAnimations];
}
```

第一步：使用CATransform3D在Y和Z轴上做旋转设置。

第二步：定义cell的初始状态，添加了一些阴影，并将第一步中的transform设置给cell中layer的transform matrix。然后将anchor设置为0.0, 0.5，也就是说让cell围绕着左边进行旋转。

第三步：通过动画，将cell设置为原始状态。此处利用了UIView的beginAnimations:context方法来更新cell中layer的值。当然还有别的方法来执行动画，不过这种方法比较简单，我们可以设置持续时间。代码里面将transform设置为CATransform3DIdentity。

这样通过第二步和第三步的状态就能够引导动画，以此完成最终效果。

完整代码工程下载地址：
[BVTableViewAnimation](https://github.com/BeyondVincent/BVTableViewAnimation)

下面是网上看到的两个内容，可以参考：

[From RW：Table View Animations Tutorial: Drop-In Cards](http://www.raywenderlich.com/49311/advanced-table-view-animations-tutorial-drop-in-cards)[`@bluesea哈哈哈`](http://weibo.com/522056706)推荐本链接

[Library Allowing You To Create Table Views With Wacky Highly Detailed Ripple Cell Animations
](http://maniacdev.com/2013/05/library-allowing-you-to-create-table-views-with-wacky-highly-detailed-ripple-cell-animations)

[Drop-In Open Source Library For Creating Wacky Animated UITableViews](http://maniacdev.com/2012/05/drop-in-open-source-library-for-creating-wacky-animated-uitableviews)

希望上面介绍对你有帮助，如果有问题，可以在下面的回复我。

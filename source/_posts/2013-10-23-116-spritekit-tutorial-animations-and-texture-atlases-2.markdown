---
layout: post
title: "Sprite Kit教程：动画和纹理图集 2"
date: 2013-10-16 23:40
comments: true
categories: iOS翻译
published: true

---

{% img /images/2013/10/3.png %}
<!--more-->



注：本文译自[`Sprite Kit Tutorial: Animations and Texture Atlases`](http://www.raywenderlich.com/45152/sprite-kit-tutorial-animations-and-texture-atlases)



###**目录**
* 创建一个工程
* 纹理图集和熊
* 一个简单的动画
* [改变动画运动的方向](#gbfx)
* [在屏幕上让熊移动](#ydx)
* [何去何从?](#hqhc)


###<a id="gbfx"></a>改变动画运动的方向

看起来不错哦！下面我们就来看看如何通过触摸屏幕上的点来控制熊的运动方向。在`MyScene.m`文件中做如下改动：

```objc
// Add these new methods
-(void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event 
{
 
    CGPoint location = [[touches anyObject] locationInNode:self];
    CGFloat multiplierForDirection;
 
    if (location.x <= CGRectGetMidX(self.frame)) {
        //walk left
        multiplierForDirection = 1;
    } else {
        //walk right
        multiplierForDirection = -1;
    }
 
    _bear.xScale = fabs(_bear.xScale) * multiplierForDirection;
    [self walkingBear];
}
 
-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event 
{    
}
```

上面的代码会根据tap的位置，让`touchesEnded`方法判断tap处于屏幕正中间的左边还是右边。通过该方法，决定熊的朝向。熊的方向是通过Sprite Kit来改变的(通过负值乘以xScale就可以让熊朝向左边。)

编译并运行程序，一切正常的话，当你在屏幕上点击时，会发现熊的朝向发生了改变。

![](/images/2013/10/12.png)


###<a id="ydx"></a>在屏幕上让熊移动

下面我们让熊可以移动到屏幕的各个位置。

在`MyScene.m`文件中做如下改动：

```objc
// Comment out the call to start the bear walking
//[self walkingBear];
 
-(void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event 
{
    //Stuff from below!
}
 
-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event 
{    
}
 
//add this method
-(void)bearMoveEnded
{
    [_bear removeAllActions];
}
```

如上所示，移除了`touchesEnded`方法中的所有代码。下面我们一步一步的往里面添加代码。

当想要停止动画的时可以调用方法`bearMoveEnded`。

下面就从`touchesEnded`方法开始吧：

`1) 确定触摸的位置并定义一个变量代表熊的朝向`

```objc
CGPoint location = [[touches anyObject] locationInNode:self];
CGFloat multiplierForDirection;
```

如上代码，利用常见的一个方法将触摸的位置转换为node坐标系中的位置。

`2) 设置速度`

```objc
CGSize screenSize = self.frame.size;
float bearVelocity = screenSize.width / 3.0;
```

如上，定义了熊移动的速度。可知熊从移动长度为屏幕宽度这么长时，需要3秒钟。由于不同设备的屏幕宽度可能会不同，所以在这里使用了self.frame.size，所以熊的速度应该是屏幕宽度/3秒。

`3) 计算出熊在X和Y轴中移动的量`

```objc
 CGPoint moveDifference = CGPointMake(location.x - _bear.position.x, location.y - _bear.position.y);
```

通过简单的利用触摸位置减去熊的位置，计算出熊在X和Y轴上应该移动的距离。

`4) 计算出实际的移动距离`

```objc
float distanceToMove = sqrtf(moveDifference.x * moveDifference.x + moveDifference.y * moveDifference.y);
```

上面的代码是计算出熊实际移动的直线距离(一个直角三角形的斜边：熊当前的位置和触摸位置)。关于游戏中涉及到的数学知识可以看看这本书：[Trigonometry for Game Programming](http://www.raywenderlich.com/35866/trigonometry-for-game-programming-part-1)。

`5) 计算出移动实际距离所需要花费的时间`

```objc
float moveDuration = distanceToMove / bearVelocity;
```

通过移动的实际距离除以移动速度计算出需要花费的时间。

`6) 需要的话对动画做翻转(Flip)处理`

```objc
if (moveDifference.x < 0) {
    multiplierForDirection = 1;
} else {
    multiplierForDirection = -1;
}
_bear.xScale = fabs(_bear.xScale) * multiplierForDirection;
```

上面的代码：确定熊往左还是往右移动。如果小于0，则往左移动，否则往右移动。

在这里，你的第一直觉可能是利用图片编辑器创建并使用对应另一个方向的图片。不过，之前我们学习过了如果通过乘法来改变sprite的xScale，进而改变sprite的方向。

`7) 运行一些action`

```objc
if ([_bear actionForKey:@"bearMoving"]) {
    //stop just the moving to a new location, but leave the walking legs movement running
    [_bear removeActionForKey:@"bearMoving"];
} //1
 
if (![_bear actionForKey:@"walkingInPlaceBear"]) {
    //if legs are not moving go ahead and start them
    [self walkingBear];  //start the bear walking
} //2
 
SKAction *moveAction = [SKAction moveTo:location duration:moveDuration];  //3
SKAction *doneAction = [SKAction runBlock:(dispatch_block_t)^() {
        NSLog(@"Animation Completed");
        [self bearMoveEnded];
}]; //4
 
SKAction *moveActionWithDone = [SKAction sequence:@[moveAction,doneAction]]; //5
 
[_bear runAction:moveActionWithDone withKey:@"bearMoving"]; //6
```

1. 停止已有的移动action(因为要准备告诉熊移动到别的地方)。这里使用的key可以开始和停止以此命名的动画的运行。
2. 如果熊还没有准备移动腿，那么就让熊的腿开始移动，否则它该如何走到新的位置呢。这里使用了我们之前使用过的方法，这个方法可以确保不启动一个已经运行着的动画(以key命名)。
3. 创建一个移动action，并制定移动到何处，以及需要花费的时间。
4. 创建一个done action，当熊到达目的地后，该action利用一个block调用一个方法来停止动画。
5. 将上面的两个action设置为一个顺序action链，就是说让这两个action按照先后顺序运行(第一个运行完之后，再运行第二个)。
6. 让熊开始运行action，并制定一个key为："bearMoving"。记住，这里的key用来判断熊是否需要移动到新的位置。

注意：Sprite Kit支持两种action：`sequential`和`grouped`。`sequential` action表示action按照顺序执行。如果想要action同时运行，那么就使用`grouped`。

当然，也可以在sequential action中包含grouped action，反之亦然。更多相关内容请看[Sprite Kit Programming Guide](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsAnimation/Conceptual/SpriteKit_PG/Introduction/Introduction.html#//apple_ref/doc/uid/TP40013043)中的`Adding Actions to Nodes`章节。

当动画执行完毕之后，`bearMoveEnded`会被调用，所有的动画都将被停止，并等待下一个移动方位。

搞定了！

现在编译并运行程序，一切正常的话，那么当点击屏幕时，熊会跟着移动。

![](/images/2013/10/13.png)

###<a id="hqhc"></a>何去何从?

这里是本文涉及到的[工程示例](http://cdn5.raywenderlich.com/wp-content/uploads/2013/09/AnimatedBear.zip)。

下面这些想法可以让动画更加有趣：

* 尝试在方法`walkingBear`中增加或者减慢运动的速度，看看效果
* 试着在屏幕上同时显示多个熊。提示：创建多个sprite node，并赋予action。

至此，你应该已经知道如何使用动画了。

如果你希望学习更多相关Sprite Kit内容，可以看看这本书：[iOS Games by Tutorials](http://www.raywenderlich.com/store/ios-7-by-tutorials)。本书会告诉你需要知道的内容——从物理特性，到磁贴地图，以及粒子系统，甚至是制作自己的关卡编辑器。

……Sprite Kit教程：动画和纹理图集 2 结束……

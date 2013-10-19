---
layout: post
title: "Sprite Kit教程：如何拖放Sprites"
date: 2013-10-20 23:44
comments: true
categories: iOS翻译
published: true

---

{% img /images/2013/10/14.png %}

<!--more-->



注：本文译自[`Sprite Kit Tutorial: How To Drag and Drop Sprites`](http://www.raywenderlich.com/44270/sprite-kit-tutorial-how-to-drag-and-drop-sprites)



###**目录**

* [开始](#ks)
* [用触摸的方式选中sprite](#xzjs)
* [用触摸的方式移动sprite和layer](#ydjl)
* [在Sprite Kit中如何使用手势识别](#sssb)
* [何去何从](#hqhc)


本文中，你可以学到如下内容：

* 利用触摸来拖放sprite的基本知识
* 利用触摸滚动view
* How to keep coordinates straight in your head
* 如何在Sprite Kit中使用手势识别

为了让本文有趣一点，这里提供了一些可爱的动物图片。

本文假设你已经了解了Sprite Kit的一些基本知识。如果还不了解的话，先看看下面的文章吧：

[`Sprite Kit教程：初学者 1`](http://beyondvincent.com/blog/2013/10/12/114-spritekit-tutorial-for-beginners-3/)
[`Sprite Kit教程：初学者 2`](http://beyondvincent.com/blog/2013/09/29/113-spritekit-tutorial-for-beginners-2/)
[`Sprite Kit教程：初学者 3`](http://beyondvincent.com/blog/2013/09/26/113-spritekit-tutorial-for-beginners-1/)

英文原文在这里：[`Sprite Kit Tutorial for Beginners`](http://www.raywenderlich.com/42699/spritekit-tutorial-for-beginners)

下面我们就开始吧。

###<a id="ks"></a>开始

在实现触摸处理之前，我们先来创建一个基本的Sprite Kit工程，并在scene中显示出一些sprite(动物)和背景。

打开Xcode，选择`File\New Project\Application\SpriteKit Game`，然后单击`Next`。

![](/images/2013/10/15.png)

将工程命名为`DragDrop`，devices选择`iPhone`，然后单击`Next`，把工程保存到磁盘中。

![](/images/2013/10/16.png)

跟[`Sprite Kit教程：初学者 1`](http://beyondvincent.com/blog/2013/10/12/114-spritekit-tutorial-for-beginners-3/)一样，我们希望这个程序只支持横屏显示(landscape)。所以在`Project Navigator`中选中`DragDrop`工程，然后选择`DragDrop` target，在弹出的画面中，只需要勾选上`Landscape Left`和`Landscape Right`。如下图所示：

![](/images/2013/10/17.png)

打开`ViewController.m`文件，并用下面的代码替换`viewDidLoad`方法(代码跟之前的一样)：

```objc
- (void)viewWillLayoutSubviews
{
    [super viewWillLayoutSubviews];
 
    // Configure the view.
    SKView * skView = (SKView *)self.view;
    if (!skView.scene) {
      skView.showsFPS = YES;
      skView.showsNodeCount = YES;
 
      // Create and configure the scene.
      SKScene * scene = [MyScene sceneWithSize:skView.bounds.size];
      scene.scaleMode = SKSceneScaleModeAspectFill;
 
      // Present the scene.
      [skView presentScene:scene];
    }
}
```

接着来这里下载本文需要用到的[图片资源](http://d1xzuxjlafny7l.cloudfront.net/downloads/DragDropImages.zip)。下载并解压之后，将所有的文件拖到工程中，其中把`Copy items into destination group’s folder (if needed)`勾选上，然后单击`Finish`。

![](/images/2013/10/18.png)

完成上面的步骤之后，打开`MyScene.m`文件，并在`@implementation`上面添加一个class extension，并声明两个属性，如下所示：


```objc
@interface MyScene () 
 
@property (nonatomic, strong) SKSpriteNode *background;
@property (nonatomic, strong) SKSpriteNode *selectedNode;
 
@end
```

稍后会用到上面的这两个属性来存储背景图片，已经当前选中的node/sprite。接着在@interface前面添加如下这行代码：

```objc
static NSString * const kAnimalNodeName = @"movable";
```

稍后将会用这个字符串来标示可移动的node。接着找到`initWithSize:`方法，并用下面的代码替换里面的内容：

```objc
- (id)initWithSize:(CGSize)size {
    if (self = [super initWithSize:size]) {
 
        // 1) Loading the background
        _background = [SKSpriteNode spriteNodeWithImageNamed:@"blue-shooting-stars"];
        [_background setName:@"background"];
        [_background setAnchorPoint:CGPointZero];
        [self addChild:_background];
 
        // 2) Loading the images
        NSArray *imageNames = @[@"bird", @"cat", @"dog", @"turtle"];
        for(int i = 0; i < [imageNames count]; ++i) {
          NSString *imageName = [imageNames objectAtIndex:i];
          SKSpriteNode *sprite = [SKSpriteNode spriteNodeWithImageNamed:imageName];
          [sprite setName:kAnimalNodeName];
 
          float offsetFraction = ((float)(i + 1)) / ([imageNames count] + 1);
          [sprite setPosition:CGPointMake(size.width * offsetFraction, size.height / 2)];
          [_background addChild:sprite];
        }
    }
 
    return self;
}
```

我们来看看上面的代码都干了什么。

1) 加载背景图片

上面方法中的第一部分代码是为scene加载背景图片(blue-shooting-stars.png)。并将该note的anchor设置为图片的左下角(0, 0)。

在Sprite Kit中，设置一个node的位置时，实际上是设置它的anchor。默认情况下，node的anchor被设置为node的正中间。在此，将anchor设置为左下角。

方法中，并没有设置背景图片的position，所以背景图的的位置默认为(0,0)。最终，图片的左下角位置是(0,0)，并向右边延伸。

2) 加载小动物

函数中接下来的代码是循环遍历列表中的图片，并将其加载到scene中。为了好的布局，其中各个node根据屏幕的长度来定位，另外还将这些node的名字设置为`kAnimalNodeName`。

之后将创建好的node添加到`_background`中。

OK！编译并运行程序，会看到屏幕中已经显示出了一些可爱的动物了。

![](/images/2013/10/19.png)



###<a id="xzjs"></a>用触摸的方式选中sprite

下面我们来实现一下根据用户当前触摸的位置判断出哪个sprite应该被选中。

用下面的代码替换`touchesBegan:withEvent:`：

```objc
-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
    UITouch *touch = [touches anyObject];
    CGPoint positionInScene = [touch locationInNode:self];
    [self selectNodeForTouch:positionInScene];
}
```

首先从touches set中获得touch。然后将touch的位置转换到一个指定node中的位置，上面的代码中使用了scene。让后将获得的方法传递给`selectNodeForTouch:`方法，该方法是一个新方法，下面我们就来看看这个方法的实现。

```objc
- (void)selectNodeForTouch:(CGPoint)touchLocation {
   //1
   SKSpriteNode *touchedNode = (SKSpriteNode *)[self nodeAtPoint:touchLocation];
 
      //2
	if(![_selectedNode isEqual:touchedNode]) {
		[_selectedNode removeAllActions];
		[_selectedNode runAction:[SKAction rotateToAngle:0.0f duration:0.1]];
 
		_selectedNode = touchedNode;
		//3
		if([[touchedNode name] isEqualToString:kAnimalNodeName]) {
			SKAction *sequence = [SKAction sequence:@[[SKAction rotateByAngle:degToRad(-4.0f) duration:0.1],
													  [SKAction rotateByAngle:0.0 duration:0.1],
													  [SKAction rotateByAngle:degToRad(4.0f) duration:0.1]]];
			[_selectedNode runAction:[SKAction repeatActionForever:sequence]];
		}
	}
 
}
```

这是一个helper方法，它主要做三件不同的事情：

1. 通过scene(self)获得touchLocation位置对应的node。
2. 获得匹配的node之后，检查一下这个node与上一次选中的node是否相同，如果相同的话，在这里直接就返回了。如果是一个新选中的node，或者还没有选中过，这个node会有一点小小的挪动动画，以此可以看出哪个node被选中了。不过在开始动画之前，需要移除当前已经选中node上的所有running actions，并在这个node上运行一个action：`rotateToAngle:duration:`。这样可以确保只有一个node在做动画，而另外的node恢复到原样。
3. 这个if语句用来判断一下选中的node是否可以进行动画(只需要检查一下node的name就可以做出判断——还记得在`initWithSize:`方法中设置的这个属性值吗？)。如果选中的node可以做动画处理，那么就创建一个sequence action——是一个动画效果，就像在主屏幕中重排/删除程序那样的效果，然后在选中的node上运行这个sequence。为了避免动画运行完毕之后会停止，在这里运行了一个一直重复的action。

下面将helper函数`degToRad`添加到文件的底部：

```objc
float degToRad(float degree) {
	return degree / 180.0f * M_PI;
}
```

由于Sprite Kit是利用弧度来做旋转效果的，所以上面这个方法将角度转换为弧度。

编译并运行程序，现在可以在屏幕上tap一个动物，当选中某个动物时，该动物会做出相应的动画效果，以表示被选中！

![](/images/2013/10/20.png)



###<a id="ydjl"></a>用触摸的方式移动sprite和layer

下面来看看如何移动这些动物！基本思路是这样的：实现`touchesMoved:withEvent:`方法，计算出距离上一次触摸移动了多远，如果有动物被选中，动物将被移动相应的距离，如果没有选中动物，那么就移动整个layer，这样用户可以从左向右的滚动layer。

在添加代码之前，我们先来探讨一下在Sprite Kit中，一个node是如何滚动的。

看看下面的图片：

![](/images/2013/10/21.png)

如上图所示，我们已经初始化了一个背景，所以背景的anchor点是(0, 0)，并且向右边扩展。黑色框中的区域表示当前的可视区域(window的大小)。

如果希望将图片往右边滚动100 points，可以通过将整个node往左边移动100 points，如第二幅图看到的效果一样。

当然，也可能希望不要移动太远。例如，不应该让layer可以往右边移动，否则会看到空白的点。

下面来看看相应的代码！将如下方法添加到文件的底部：

```objc
- (CGPoint)boundLayerPos:(CGPoint)newPos {
    CGSize winSize = self.size;
    CGPoint retval = newPos;
    retval.x = MIN(retval.x, 0);
    retval.x = MAX(retval.x, -[_background size].width+ winSize.width);
    retval.y = [self position].y;
    return retval;
}
 
- (void)panForTranslation:(CGPoint)translation {
    CGPoint position = [_selectedNode position];
    if([[_selectedNode name] isEqualToString:kAnimalNodeName]) {
        [_selectedNode setPosition:CGPointMake(position.x + translation.x, position.y + translation.y)];
    } else {
        CGPoint newPos = CGPointMake(position.x + translation.x, position.y + translation.y);
        [_background setPosition:[self boundLayerPos:newPos]];
    }
}
```

第一个方法`boundLayerPos:`是为了确保不会将layer移动到背景图片范围之外。在这里传入一个需要移动到的位置，然后该方法会对位置做适当的判断处理，以确保不会移动太远。

接着方法`panForTranslation:`首先判断一下_selectedNode是否为动物node，如果是的话，根据传入的参数来为node设置新的位置。如果是background layer，同样也会设置一个新的位置，只不过新的位置需要调用`boundLayerPos:`方法获得。

完成上面之后，可以实现`touchesMoved:withEvent:`方法了：

```objc
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event {
	UITouch *touch = [touches anyObject];
	CGPoint positionInScene = [touch locationInNode:self];
	CGPoint previousPosition = [touch previousLocationInNode:self];
 
	CGPoint translation = CGPointMake(positionInScene.x - previousPosition.x, positionInScene.y - previousPosition.y);
 
	[self panForTranslation:translation];
}
```

跟`touchesBegan:withEvent:`一样，先获得touch，然后将它的位置转换为scene中的相应位置。为了计算出移动的距离，需要上一次触摸的位置。

通过当前位置减去上一次的位置就可以计算出需要移动的距离了。最后调用`panForTransaltion:`方法，并将移动距离传入即可。

搞定！编译并运行程序，现在可以通过拖放的方式移动sprite(以及layer)了！

![](/images/2013/10/22.png)



###<a id="sssb"></a>在Sprite Kit中如何使用手势识别

在Sprite Kit中还可以使用手势识别来处理触摸！

手势识别可以识别不同的手势，如tap，double tap，swipe或pan。

通过手势识别，我们可以不用写大量的代码来识别不同的手势（如tap，double tap，swipe或pan），只需要创建一个手势识别对象并将其添加到view中，即可进行手势识别。当有手势发生，会有一个回调。

下面就来看看如何在Sprite Kit中使用手势识别。

首先，注释掉触摸处理方法：`touchesBegan:withEvent:`和`touchesMoved:withEvent:`(因为要使用不同的处理方法啦)。

然后添加如下方法：

```objc
- (void)didMoveToView:(SKView *)view {
    UIPanGestureRecognizer *gestureRecognizer = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(handlePanFrom:)];
    [[self view] addGestureRecognizer:gestureRecognizer];
}
```

当scene第一次显示出来时会调用这个方法。在上面的方法中创建了一个pan手势识别器，并用当前的scene来对其做初始化，另外还传入一个callback：`handlePanFrom:`。接着把这个手势识别器添加到scene中的view里面。

注意：可能你会问为什么要在这里添加识别器，而不是在scene的init方法中。答案很简单：`SKScene`有一个view属性，保存着SKView——该view用来显示scene，不过只有scene显示到屏幕中时这个属性才会被初始化，所以在init方法被调用时该属性是nil的。此处的`didMoveToView:`类似于UIKit中的`viewDidAppear:`，当scene显示出来时，`didMoveToView:`会被调用。

接着，将下面的代码添加到`MyScene.m`文件底部：

```objc
- (void)handlePanFrom:(UIPanGestureRecognizer *)recognizer {
	if (recognizer.state == UIGestureRecognizerStateBegan) {
 
        CGPoint touchLocation = [recognizer locationInView:recognizer.view];
 
        touchLocation = [self convertPointFromView:touchLocation];
 
        [self selectNodeForTouch:touchLocation];
 
 
    } else if (recognizer.state == UIGestureRecognizerStateChanged) {
 
        CGPoint translation = [recognizer translationInView:recognizer.view];
        translation = CGPointMake(translation.x, -translation.y);
        [self panForTranslation:translation];
        [recognizer setTranslation:CGPointZero inView:recognizer.view];
 
    } else if (recognizer.state == UIGestureRecognizerStateEnded) {
 
        if (![[_selectedNode name] isEqualToString:kAnimalNodeName]) {
            float scrollDuration = 0.2;
            CGPoint velocity = [recognizer velocityInView:recognizer.view];
            CGPoint pos = [_selectedNode position];
            CGPoint p = mult(velocity, scrollDuration);
 
            CGPoint newPos = CGPointMake(pos.x + p.x, pos.y + p.y);
            newPos = [self boundLayerPos:newPos];
            [_selectedNode removeAllActions];
 
            SKAction *moveTo = [SKAction moveTo:newPos duration:scrollDuration];
            [moveTo setTimingMode:SKActionTimingEaseOut];
            [_selectedNode runAction:moveTo];
        }
 
    }
}
```

当手势开始、改变(例如用户持续drag)，以及结束时，上面这个callback函数都会被调用。该方法会进入不同的case，以处理不同的情况。

当手势开始时，将坐标系统转换为node坐标系(注意这里没有便捷的方法，只能这样处理)。然后电泳之前写的helper方法`selectNodeForTouch:`。

当手势发生改变时，需要计算出手势移动的量。还在手势识别器已经为我们存储了手势移动的累计量(translation)！不过考虑到效果的差异，我们需要在UIKit坐标系和Sprite Kit坐标系中对坐标进行转换。

平移(pan)之后，需要把手势识别器上的translation设置为0，否则该值会继续被累加。

当手势结束之后，上面的函数中有一些有趣的代码！UIPanGestureRecognizer可以为我们提供一个移动的速度。通过这个速度可以对node做一个动画——滑动一小点，这样用户可以对node做一个快速的摇动，就像table view上的那种效果一样。

所以，在这里包含的代码用来计算基于速度移动的一个point，然后运行一个moveTo action(为了更加好看，附带`SKActionTimingEaseOut`效果)。

接着添加如下一个方法到文件中：

```objc
CGPoint mult(const CGPoint v, const CGFloat s) {
	return CGPointMake(v.x*s, v.y*s);
}
```

上面这个方法是将滚动的时间乘以速度。

编译并运行程序，现在应该可以用手势识别器滑动和移动动物了。

![](/images/2013/10/23.png)


###<a id="hqhc"></a>何去何从

本文的代码工程在[这里](http://cdn2.raywenderlich.com/downloads/DragDropSpriteKit.zip)。

至此，你应该知道如何在Sprite Kit程序中使用touch来移动node，以及如何在Sprite Kit中使用手势识别器。

现在，你也可以尝试利用别的手势识别器对上面的工程做扩展处理，例如pinch或rotate手势识别器——可以让猫长大哦！

如果你希望学习更多相关Sprite Kit内容，可以看看这本书：[iOS Games by Tutorials](http://www.raywenderlich.com/store/ios-7-by-tutorials)。本书会告诉你需要知道的内容——从物理特性，到磁贴地图，以及粒子系统，甚至是制作自己的关卡编辑器。


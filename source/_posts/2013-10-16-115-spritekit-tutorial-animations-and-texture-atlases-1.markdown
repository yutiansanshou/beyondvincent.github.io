---
layout: post
title: "Sprite Kit教程：动画和纹理图集 1"
date: 2013-10-16 12:40
comments: true
categories: iOS翻译
published: ture

---

{% img /images/2013/10/2.png %}

<!--more-->



注：本文译自[`Sprite Kit Tutorial: Animations and Texture Atlases`](http://www.raywenderlich.com/45152/sprite-kit-tutorial-animations-and-texture-atlases)



###**目录**
* [创建一个工程](#cjgc)
* [纹理图集和熊](#wltj)
* [一个简单的动画](#jddh)
* 改变动画运动的方向
* 在屏幕上让熊移动
* 何去何从?


从本文，可以学习到如何使用iOS 7中的Sprite Kit框架创建一个简单的动画：在屏幕上行走的熊。

另外还可以学习到如何使用纹理图集来制作动画效果，如何在触摸事件发生时让熊移动，以及改变熊运动的方向。

学习本文之前，最好先看看下面的文章：
[`Sprite Kit教程：初学者 1`](http://beyondvincent.com/blog/2013/10/12/114-spritekit-tutorial-for-beginners-3/)
[`Sprite Kit教程：初学者 2`](http://beyondvincent.com/blog/2013/09/29/113-spritekit-tutorial-for-beginners-2/)
[`Sprite Kit教程：初学者 3`](http://beyondvincent.com/blog/2013/09/26/113-spritekit-tutorial-for-beginners-1/)

英文原文在这里：[`Sprite Kit Tutorial for Beginners`](http://www.raywenderlich.com/42699/spritekit-tutorial-for-beginners)

下面我们就开始吧。

###<a id="cjgc"></a>创建一个工程

我们先创建好一个工程架子——选择`File\New Project…`，在`iOS Application`中选择`Sprite Kit Game`，如下图所示：

{% img /images/2013/10/4.png %}

选择`Next`，并将工程命名为`AnimatedBear`，把Class Prefix中的内容清除掉，并将Devices选择为`iPad`，如下图所示：

{% img /images/2013/10/5.png %}

接着选择`Next`，将工程保存到磁盘中。

现在编译并运行程序的话，当点击屏幕时，可以看到在屏幕中有一个自动旋转的飞船。如下图所示：

{% img /images/2013/10/6.png %}

这样工程架子就准备好了，下面我们去寻找一些熊的动画资源——从这里下载即可：[BearImages Art](http://cdn5.raywenderlich.com/wp-content/uploads/2013/08/BearImages.zip)。如下图所示：

![熊的示例图片](/images/2013/10/7.jpg)

上面下载到的图片有所需要的最大分辨率——iPad retina显示(2X)和non-retina版本(1x)。这些文件的命名方式为bear1..n@2x~ipad和bear1..n~ipad.png。

在这里，构建一个动画，你可以只需要将这些图片直接添加到Sprite Kit工程中即可。不过，还有另外一种更加方便的方法来构建动画——使用纹理图集。

###<a id="wltj"></a>纹理图集和熊

如果之前你没使用过纹理图集，那你可以把它想象为一副很大的图片，其中包括动画中需要使用到的各种图片。这个图集可以看做是一个文件，它指定了每个sprite的边界范围，当在代码中需要使用时，可以将这些sprite取出来。

使用纹理图集是因为Sprite Kit和图形引擎会对其做相应的优化处理。`后面这段话暂时不知道什么意思:`

```
	If you use sprites within a texture atlas properly, rather than making one OpenGL ES draw call per sprite it just makes one per texture atlas sheet.
```

简而言之——使用纹理图集会非常的快，特别是有大量sprite的时候！

Xcode会自动的生成这个纹理图集文件，并指定好每个sprite的边界范围，这样当在代码中需要用到某个sprite的时候，可以方便取出来。这一切都会自动处理，开发者不用亲力为之。

```
注意：当纹理图集有问题时(例如错误的图片等)，那么建议clean一下工程(Product\Clean)——这样可以强制让纹理图集重新构建。
```

为纹理图集创建一个文件夹，并将图片文件放置到该文件夹中，然后在文件夹名称尾部添加`.atlas`。这样Xcode就能识别出.atlas扩展名，进而自动的将图片合并为一个纹理图集。

之前下载的图片资源中有一个名为`BearImages.atlas`的文件夹，里面包含了各种分辨率的图片(是其它两个文件夹中的图片合集)。

将名为`BearImages.atlas`的文件夹拖拽到程序中，如下图所示：

![](/images/2013/10/8.png)

当释放鼠标时，会看到如下图片中的对话框：是关于如何添加到工程中的。确保选中这三项：`Copy items into destination group’s folder`, `Create groups for any added folder`, 和 `the AnimatedBear`，然后点击`Finish`：

![](/images/2013/10/9.png)

在Xcode中展开这个文件夹`BearImages.atlas`，会看到如下内容：

![](/images/2013/10/10.png)

下面，是时候让熊动起来了！

###<a id="jddh"></a>一个简单的动画

这里我们先把熊显示在屏幕中间，并开启永久循环动画。

此处主要都是在`MyScene.m`中写代码。打开这个文件，并用下面的代码替换之：

```objc
#import <AVFoundation/AVFoundation.h>
#import "MyScene.h"
 
@implementation MyScene 
{
 
    SKSpriteNode *_bear;
    NSArray *_bearWalkingFrames;
 
}
 
-(id)initWithSize:(CGSize)size 
{
    if (self = [super initWithSize:size]) {
        /* Setup your scene here */
 
        self.backgroundColor = [SKColor blackColor];
 
        // TODO...
 
    }
    return self;
}
 
-(void)update:(CFTimeInterval)currentTime {
    /* Called before each frame is rendered */
}
 
@end
```

上面的代码很简单，只是定义了几个稍后会用到的变量。编译并运行一下，确保没有错误——会看到屏幕是黑色的。

接下来要让熊动起来，有5步需要处理，我们就来看看吧。

记得将下面的代码添加到`initWithSize`方法的`TODO`位置。

`1) 构建一个用于保存行走帧(walking frame)`

```objc
NSMutableArray *walkFrames = [NSMutableArray array];
```
`2) 加载纹理图集`

```objc
SKTextureAtlas *bearAnimatedAtlas = [SKTextureAtlas atlasNamed:@"BearImages"];
```
上面的代码会从程序bundle的数据区中创建一个图集。Sprite Kit会根据设备的寻找对应分辨率的图片文件，在iPad retina上会使用BearImages@2x~ipad.png。

`3) 构建帧列表`

```objc
int numImages = bearAnimatedAtlas.textureNames.count;
for (int i=1; i <= numImages/2; i++) {
    NSString *textureName = [NSString stringWithFormat:@"bear%d", i];
    SKTexture *temp = [bearAnimatedAtlas textureNamed:textureName];
    [walkFrames addObject:temp];
}
_bearWalkingFrames = walkFrames;
```

上面的代码根据图片名称从图集中循环获取到一个帧列表(这些图片的命名为bear1.png->bear8.png)，注意到`numImages`这个变量了吗？它为啥要除以2呢？

这是因为：纹理图集包含了所有分辨率的图片文件(non-retina和retina)。共有16个文件，每种分辨率有8个文件。要想加载某种分辨率的图片，就需要除以2。这样通过名称和计数器，就能获取到正确的分辨率图片。

`4) 创建sprite，并将其位置设置为屏幕中间，然后将其添加到场景中`

```objc
SKTexture *temp = _bearWalkingFrames[0];
_bear = [SKSpriteNode spriteNodeWithTexture:temp];
_bear.position = CGPointMake(CGRectGetMidX(self.frame), CGRectGetMidY(self.frame));
[self addChild:_bear];
[self walkingBear];
```

利用帧列表的第一帧构建一个sprite，然后将其放置到屏幕正中间。最后调用walkingBear方法，让熊开始走动。

`5) 在initWithSize方法后面添加一个新的方法walkingBear`

```objc
-(void)walkingBear
{
    //This is our general runAction method to make our bear walk.
    [_bear runAction:[SKAction repeatActionForever:
                      [SKAction animateWithTextures:_bearWalkingFrames
                                       timePerFrame:0.1f
                                             resize:NO
                                            restore:YES]] withKey:@"walkingInPlaceBear"];
    return;
}
```

上面的这个action会以0.1秒的间隔开始播放各帧。如果你的代码再次调用这个方法使动画重新开始的话，`walkingInPlaceBear`这个key会强制移除动画。这对于确保动画不相互干扰非常重要。`withKey`参数还提供了一个钟方法对动画进行检查，来判断其是否通过名称运行的。

这个action是永久重复的，内部的actionan `imateWithTextures`会按顺序动画播放帧列表中的图片。

`完工!`

现在编译并运行程序，一切正常的话，会在屏幕中看到一个会动的熊，如下图所示：

![](/images/2013/10/11.png)


……Sprite Kit教程：动画和纹理图集 1 结束……

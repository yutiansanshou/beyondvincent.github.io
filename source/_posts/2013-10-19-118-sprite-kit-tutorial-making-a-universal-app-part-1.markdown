---
layout: post
title: "Sprite Kit教程：制作一个通用程序 1"
date: 2013-10-27 20:30
comments: true
categories: iOS翻译
published: true

---

{% img /images/2013/10/24.png %}

<!--more-->

注：本文译自[`Sprite Kit Tutorial: Making a Universal App: Part 1`](http://www.raywenderlich.com/49695/sprite-kit-tutorial-making-a-universal-app-part-1)

###**目录**

* [UI规划：概述](#msgs)
* [UI规划：小结](#msxj)
* [开始](#ks)
* [纹理图集](#wltj)
* [背景设置](#bjsb)
* [鼹鼠的放置](#asfz)
* [弹出鼹鼠](#tcas)
* [何去何从](#hqhc)


本文将介绍如何制作一个通用程序(打鼹鼠的游戏)——可以在iPhone和iPad上运行(包括retina显示的支持。)

学习本文之前，需要掌握以下知识：

[`Sprite Kit教程：初学者 1`](http://beyondvincent.com/blog/2013/10/12/114-spritekit-tutorial-for-beginners-3/)
[`Sprite Kit教程：初学者 2`](http://beyondvincent.com/blog/2013/09/29/113-spritekit-tutorial-for-beginners-2/)
[`Sprite Kit教程：初学者 3`](http://beyondvincent.com/blog/2013/09/26/113-spritekit-tutorial-for-beginners-1/)

英文原文在这里：[`Sprite Kit Tutorial for Beginners`](http://www.raywenderlich.com/42699/spritekit-tutorial-for-beginners)

[`Sprite Kit教程：动画和纹理图集 1`](http://beyondvincent.com/blog/2013/10/16/115-spritekit-tutorial-animations-and-texture-atlases-1/)
[`Sprite Kit教程：动画和纹理图集 2`](http://beyondvincent.com/blog/2013/10/16/116-spritekit-tutorial-animations-and-texture-atlases-2/)

英文原文在这里：[`Sprite Kit Tutorial: Animations and Texture Atlases`](http://www.raywenderlich.com/45152/sprite-kit-tutorial-animations-and-texture-atlases)

[`Sprite Kit教程：如何拖放Sprites`](http://beyondvincent.com/blog/2013/10/20/117-spritekit-tutorial-how-to-drag-and-drop-sprites/)

英文原文在这里：[`Sprite Kit Tutorial: How To Drag and Drop Sprites`](http://www.raywenderlich.com/44270/sprite-kit-tutorial-how-to-drag-and-drop-sprites)

如果还没有看上面的这些文章(或者相关的知识)，建议你先去看一下。

本文会有两篇文章。第一篇，会先创建一个基本的游戏——可爱的小鼹鼠聪洞里面弹出来。为了让游戏在iPhone和iPad(支持retina显示)上看起来很优美，本文还花了大量的时间来考虑如何做游戏的美术规划和坐标。

###<a id="msgs"></a>UI规划：概述

我们希望程序可以在iPhone 3.5英寸，4英寸(iPhone 5)和iPad上良好的运行，所以在开始之前，我们需要认真的做好UI规划。

为了搞明白需要什么样的UI尺寸，我们先来看看下面的相关内容：

* Retina显示
* 4英寸iPhone显示
* iPad和iPhone长宽比

下面开始吧！

####`Retina显示`

在iPhone中，non-retina和retina在显示上的最大区别就是retina的分辨率是non-retina的2倍。所以在non-retina上面分辨率为 480 * 320(landscape)，而retina则是960 * 640.

![](/images/2013/10/26.jpg)

同样iPad也分为non-retina和retina，它们的分辨率相差也为2倍，non-retina显示的分辨率是1024 * 768像素，而retina上面则是2048 * 1536像素！

![](/images/2013/10/27.jpg)

稍等，你可能在想：双倍分辨率岂不是打乱了所有已经写好的程序，例如iPhone上的480 * 320和iPad的1024 * 768？这是有可能的，除非是在Sprite Kit中设置尺寸或者坐标，此时实际上是在UIKit中进行设置，并且设置的尺寸单位叫做`points`，而不是像素(pixels)。

在non-retina显示上，无论是iPhone火iPad，一个point代表一个pixel，而在retina上面，一个point代表2个pixels。所以将位置设置为(10,10)point时，non-retina上将是(10,10)，而retina上则是(20,20)，所以它们依然会显示在相同的偏移量上。不错吧！

当使用苹果提供的控件或者Core Graphics时，苹果已经写好了相关代码，让它们在retina显示起来很好看。

唯一需要注意的就是关于使用的图片。比如在iPhone或iPad程序中又一个200 * 200d 图片。如果什么事情都不做的话，在retina上面会自动的将这个图片放大两倍——这看起来不是太好，因为我们并没有提供相关分辨率的图片。

![](/images/2013/10/28.jpg)

因此针对retina显示我们需要提供所有图片的另外一个版本，也就是说需要一个普通的版本，以及另外2倍分辨率的一个版本。如果将2倍分辨率图片命名为"@2x"后缀，那么当利用[SKSpriteNode spriteNodeWithImageNamed:...]或者类似的APIs加载sprite时，它会自动的将@2x图片加载到retina显示上。

所以在开发针对retina显示的Sprite Kit游戏时也很简单——只需要添加@2x的图片，基本上就搞定了。

####`4英寸iPhone显示`

iPhone 5设备在屏幕上显示的分辨率比以前的更大了，对于游戏显示上来说，这非常的好。本文中的处理很简单，只需要将背景图片做一个扩展延伸即可。

iPhone 5的分辨率是1136 * 640——宽高比为16:9。用point来衡量的话则是568 * 320.

###`iPad和iPhone的宽高比`

上面我们已经看到要处理retina显示很容易，但是要想创建一个通用的程序呢(可以运行在iPhone和iPad设备上)。

其实要想创建一个通用的程序还真有一个麻烦的事情——iPhone和iPad的宽高比不一样！

iPhone的比例是1.5(480 * 320 或960 * 640)，而iPad是1.33(768 * 1024或1536 * 2048)。

由于比例不同，如果一副能够在non-retina iPad(768×1024)上完整显示，你希望将其在iPhone上重用，那么不会完整的匹配上，如果将其缩放，按照宽度进行适配(乘以0.9375)，会得到720×960的尺寸，这样就会把高度剪切掉一部分。

![](/images/2013/10/29.jpg)

发生这种情况会让人比较烦恼，我们不仅需要处理背景图片的问题，不同的宽高比导致不同设备间使用相同的坐标比较困难。

下面是我了解到的一些对应的处理方法：

* 在3.5英寸的iPhone retina显示屏幕正中间确定一个`可玩区域`。这样剩下的区域可以用一个背景图片来覆盖，不要让玩家关注这一剩下的区域。这样一来在不同设备间进行左边的转换和重用要非常方便。本文将利用这种方法。
* 让iPad显示内容的宽高比设置为跟iPhone一样：左右留出32points的边距，上下留出64point的边距，此时在正中间的区域就是1024×768 points。这样只要让游戏程序的内容显示在这1024×768的范围内，就可以在每个设备间对图片进行缩放了。
* 由于pixel和point是有区别的。那么我们就创建出iPad retina显示的图片(536×2048 px)，然后将该图片除以2，这样也就可以用于non-retina iPad显示了！

###`iOS 模拟器选项`

下面这些模拟器可以运行iOS 7：

* iPhone Retina (3.5-inch) – iPhone 4 和 4S
* iPhone Retina (4-inch) – iPhone 5, 5C, 和 5S
* iPad – iPad 1, 2, 和 Mini
* iPad Retina – iPad 3 和 4

注意：这里并没有non-retina iPhone——因为没有任何一台no-retina iPhone或iPod touch可以运行iOS 7。

另外由于Sprite Kit是在iOS 7中才引入的，所以就不用考虑no-retina iPhone或iPod touch设备了。

###<a id="msxj"></a>UI规划：小结

基于上面的一些讨论，下面是本文的相关计划：

* UI设计的范围(可玩区域)在960×640范围内，在retina iPhone(3.5英寸)中全屏显示，4英寸iPhone，iPad和retina iPad中居中显示。
* 可用的UI元素放置在纹理图集文件中。@2x表示用于iPad retina显示的图片。
* 由于需要全屏显示，所以背景图片是一个特例。创建一个1024x768 point尺寸的图片，这样可以完全显示在iPad中。并且这个图片可以缩放显示在3.5英寸的iPhone上，只不过背景图中的有些内容不能显示出来，但是这关系并不太大。
* 4英寸的iPhone将通过代码来使用`-568`的纹理图集，并将其`可玩区域`居中。
* iPad和iPad retina通过代码使用`-ipad`的纹理图集，并将其坐标转换到`可玩区域`中，另外在使用适当的字体大小等。

来这里可以下载到本文的[UI资源](http://cdn4.raywenderlich.com/downloads/WhackAMoleSKArt.zip)。解压出下载到的文件，可以看到如下一些内容：

* 在TextureAtlases中又3个文件夹。每个文件夹中的UI元素针对不同的显示(3.5英寸iPhone, 4英寸iPhone, 和 iPad)。
* iPad纹理图集文件夹中包含的图片是针对non-retina和retina iPad的。
* 在foreground文件夹中，有两个前景图片(下部和上部的图片)。被分为了两部分，这样可以将鼹鼠放置在下部和上部，看起来鼹鼠就像到地下了一样。
* 4英寸的iPhone是另外一个特列。因此在这里另外构建了新的前景图，以利用上更多的空间。
* 在background文件夹中，虽然iPad的宽高比是1.33，但这里做的背景图宽高比为一半,这样做是因为背景图片基本上可以忽略不计（只是3个鼹鼠洞）。所以不值得在这上面耗费，因为只需要用小纹理的图片替代即可，在需要放大的时候放大一下。
* 在sprite文件夹中，所有的sprite尺寸都适合显示在960×640大小的可玩区域中。注意，这里有一个鼹鼠和两个相关动画(鼹鼠笑和被打)。

上面搞了这么多，现在终于可以开始了！

###<a id="ks"></a>开始

打开Xcode，选择File > New > Project…，然后选中Sprite Kit Game并单击Next。将工程命名为WhackAMole，devices选中universal，接着再单击Next。选择一个路径来保存工程，然后单击Create。

当工程打开之后，应该能看到Project Navigator中的工程文件已经被选中了，如果没有选中，那么将其选中，然后在target中选中WhackAModle，以及选中顶部的General，在Deployment info里面可以看到一些设备朝向的勾选框。在这里我们的游戏是landscape的，所以勾选上iPhone和iPad的Landscape Left和Landscape Right。

![](/images/2013/10/30.png)

另外，为了让朝向正确，还需要对代码做一些修改。打开ViewController.m文件并用下面的viewWillLayoutSubviews:方法替换viewDidLoad方法：

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

为什么要这样做呢？默认情况下View Controller views是以竖直的方式加载，所以横屏模式下，当viewDidLoad被调用的时候不能保证尺寸是正确的，不过当viewWillLayoutSubviews被调用的时候view的size将是正确的。如上代码所示，大多数代码与viewdidLoad中的相同。需要关注的就是if语句中关于skView.scene的配置。当然在这里需要判断一下skView.scene是否已经存在(viewWillLayoutSubviews方法可能会被多次调用)。

###<a id="wltj"></a>纹理图集

纹理图集的配置非常简单。首选创建一个文件夹并且文件名已`.atlas`结尾。接着将那些UI元素拷贝到这个文件夹里面。然后在Xcode工程中添加这个文件夹即可！

简单吧！当在编译程序的时候，Xcode会把`.atlas`结尾的文件夹中的图片生成纹理图集。

注意：添加到`.atlas`文件夹中的图片尺寸不能超过2048×2048 pixels，否则会出错——2048×2048 pixels是自动生成纹理图集的最大尺寸。

下面看看具体如何做。找到之前下载的压缩文件，在压缩文件中有一个名为TextureAtlases的文件夹。这个文件夹中包含了3中设备类型的UI元素(iPad, iPhone, 和 WidescreeniPhone)。这些文件家中都包含有`.atlas`文件夹。我们将TextureAtlases文件夹拖至工程中，确保勾选上`Copy items into destination group’s folder (if needed)`。

![](/images/2013/10/31.png)

本文中为了让一切变得简单点，我们为每种类型的设备准备了一套纹理图集(iPhone 3.5-inch, iPhone 4-inch 和 iPads)。在iPhone 4英寸中可以重用iPhone3.5英寸中的一些纹理图集，


###<a id="bjsb"></a>背景设置

在开始修改scene中显示内容之前，我们需要添加一个宏以及一个helper方法。打开MyScene.m文件，并在文件的头部添加如下一行代码(在#import下面)：

```objc
#define IS_WIDESCREEN ( fabs( ( double )[ [ UIScreen mainScreen ] bounds ].size.height - ( double )568 ) < DBL_EPSILON )
```

上面这个宏可以判断程序是否允许在4英寸的屏幕中，该宏将被用在helper方法中，如果要了解上面宏的详细内容，[看这里](http://stackoverflow.com/questions/12446990/how-to-detect-iphone-5-widescreen-devices)。

接着添加一个helper方法——为运行程序的设备获取正确的SKTextureAtla。这个方法接收一个文件名，并在文件名尾部添加一个正确的标示符，然后返回正确的一个SKTextureAtla。

```objc
- (SKTextureAtlas *)textureAtlasNamed:(NSString *)fileName
{
    if (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPhone) {
 
        if (IS_WIDESCREEN) {
            // iPhone Retina 4-inch
            fileName = [NSString stringWithFormat:@"%@-568", fileName];
        } else {
            // iPhone Retina 3.5-inch
            fileName = fileName;
        }
 
    } else {
        fileName = [NSString stringWithFormat:@"%@-ipad", fileName];
    }
 
    SKTextureAtlas *textureAtlas = [SKTextureAtlas atlasNamed:fileName];
 
    return textureAtlas;
}
```

上面的代码做了些什么？

* 首选判断设备是否为一台iPhone。
* 如果是一台iPhone，然后利用之前定义的IS_WIDESCREEN宏判断是否为4英寸显示屏。如果是的话，就在文件名尾部添加`-568`。
* 如果设备是iPad或iPad retina，那么在文件尾部添加`-ipad`。
* 根据文件名创建并返回一个新的SKTextureAtlas。

接着找到initWithSize:方法。移除掉设置背景颜色和创建Hell World lable的6行代码，然后用下面的代码替换之：

```objc
// Add background
SKTextureAtlas *backgroundAtlas = [self textureAtlasNamed:@"background"];
SKSpriteNode *dirt = [SKSpriteNode spriteNodeWithTexture:[backgroundAtlas textureNamed:@"bg_dirt"]];
dirt.scale = 2.0;
dirt.position = CGPointMake(CGRectGetMidX(self.frame), CGRectGetMidY(self.frame));
dirt.zPosition = 0;
[self addChild:dirt];
 
// Add foreground
SKTextureAtlas *foregroundAtlas = [self textureAtlasNamed:@"foreground"];
SKSpriteNode *upper = [SKSpriteNode spriteNodeWithTexture:[foregroundAtlas textureNamed:@"grass_upper"]];
upper.anchorPoint = CGPointMake(0.5, 0.0);
upper.position = CGPointMake(CGRectGetMidX(self.frame), CGRectGetMidY(self.frame));
upper.zPosition = 1;
[self addChild:upper];
 
SKSpriteNode *lower = [SKSpriteNode spriteNodeWithTexture:[foregroundAtlas textureNamed:@"grass_lower"]];
lower.anchorPoint = CGPointMake(0.5, 1.0);
lower.position = CGPointMake(CGRectGetMidX(self.frame), CGRectGetMidY(self.frame));
lower.zPosition = 3;
[self addChild:lower];
 
// Add more here later...
```
我们来看看上面的代码都做了什么。

* `Add background` 这部分代码使用之前的helper方法创建一个背景纹理图集。接着从背景纹理图集中构建一个dirt sprite。最后将其缩小2倍，并将其添加到scene正中间。将其缩小的目的是为了节省空间。
* `Add foreground` 这部分代码跟上面background中的十分相似，只不过这两个foreground sprite在同一个纹理图集中罢了。这里用了一种方便的方法来放置图片：设置anchor point(顶部图片设置middle/bottom，底部图片设置middle/top)。这种方法不需要做复杂的数学运算，就能在所有的设备上做出正确的显示。另外需要注意的是iPhone上的背景图片有一部分将不会显示出来，不过在这里并没有太大的影响。另外需要留意的是设置了图片的zPosition值，这样可以确保图片的正确排序。
* `SKSpriteNode的zPosition属性` 这个数学用来决定每个sprite在scene所处层次的位置。可以将其看做一个蛋糕，其中dirt sprite处于最底层，所以使用最小的一个值。添加别的层时增加相应的值，所以上半部分前景图设置为1，而下半部分设置为3，那么2呢？这个值是留给鼹鼠的——因为鼹鼠将出现在上部前景图上面，而在下部前景图后面。

在运行程序之前，再做一点清理工作。找到touchesBegan:方法，并将其删除掉。

编译并运行程序，现在可以看到屏幕上显示出了背景图和前景图！并且在iPhone和iPad模拟器中运行，也能正确的显示！如下图所示：

![](/images/2013/10/32.png)

###<a id="asfz"></a>鼹鼠的放置

在这个游戏中，我们将添加3个鼹鼠到scene中——上图中的每个洞放一个。鼹鼠默认是在地下的，偶尔会弹出来，当弹出来时，我们可以打击它们。

首先我们先将鼹鼠放到每个洞中。为了确保鼹鼠位置的正确，最好先把鼹鼠显示在最上面，等调好位置之后，在将其放到后台去。

打开MyScene.h文件，并按照如下代码进行修改：

```objc
#import <SpriteKit/SpriteKit.h>
 
@interface MyScene : SKScene
 
@property (strong, nonatomic) NSMutableArray *moles;
@property (strong, nonatomic) SKTexture *moleTexture;
 
@end
```
上面的代码添加了一个SKTexture和一个数组。创建鼹鼠的时候会用到SKTexture，创建好的每个鼹鼠会被添加到数组中，这样方便之后循环获得每个鼹鼠。

在添加鼹鼠之前，首先定位到MyScene.m的顶部，并将下面这行代码添加到`@implementation MyScene`之前。

```objc
const float kMoleHoleOffset = 155.0;
```

这是一个float类型的常量，用来对鼹鼠进行定位。

接着，将如下代码添加到initWithSize:方法最后面：

```objc
// Load sprites
self.moles = [[NSMutableArray alloc] init];
SKTextureAtlas *spriteAtlas = [self textureAtlasNamed:@"sprites"];
self.moleTexture = [spriteAtlas textureNamed:@"mole_1.png"];
 
 
float center = 240.0;
 
if (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPhone && IS_WIDESCREEN) {
    center = 284.0;
}
 
SKSpriteNode *mole1 = [SKSpriteNode spriteNodeWithTexture:self.moleTexture];
mole1.position = [self convertPoint:CGPointMake(center - kMoleHoleOffset, 85.0)];
mole1.zPosition = 999;
mole1.name = @"Mole";
mole1.userData = [[NSMutableDictionary alloc] init];
[self addChild:mole1];
[self.moles addObject:mole1];
 
SKSpriteNode *mole2 = [SKSpriteNode spriteNodeWithTexture:self.moleTexture];
mole2.position = [self convertPoint:CGPointMake(center, 85.0)];
mole2.zPosition = 999;
mole2.name = @"Mole";
mole2.userData = [[NSMutableDictionary alloc] init];
[self addChild:mole2];
[self.moles addObject:mole2];
 
SKSpriteNode *mole3 = [SKSpriteNode spriteNodeWithTexture:self.moleTexture];
mole3.position = [self convertPoint:CGPointMake(center + kMoleHoleOffset, 85.0)];
mole3.zPosition = 999;
mole3.name = @"Mole";
mole3.userData = [[NSMutableDictionary alloc] init];
[self addChild:mole3];
[self.moles addObject:mole3];
```

上面的代码首先创建并加载一个SKTextureAtlas。接着根据sprite纹理图集中的mole_1.png 创建一个SKTexture，这将用来创建3个鼹鼠。Texture的重用性可以让Sprite Kit处理和渲染sprite更加高效。

接下来的这个值用来设置center。如果设备是4英寸的iPhone，那么这个center值将反映出额外的尺寸。

接着为每个鼹鼠创建对应的sprite，并将它们放置到scene中，还把它们添加到鼹鼠数组中。注意，每个鼹鼠的位置是利用center位置和文件头部定义的常量决定的。针对iPhone 3.5英寸的设备，鼹鼠位置处在480×320的可玩区域，而如何是iPad，相关位置需要做转换，所以下面写了一个helper方法convertPoint。

将下面这个方法添加到initWithSize:方法后面：

```objc
- (CGPoint)convertPoint:(CGPoint)point
{
    if (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPad) {
        return CGPointMake(32 + point.x*2, 64 + point.y*2);
    } else {
        return point;
    }
}
```

上面这个方法将可玩区域的point转换到iPad上适当的位置。记住：

* iPad的屏幕尺寸更大，所有的point都是双倍。
* 由于是将960×640的区域转换为1024×786的iPad区域，所以需要将左右边距分别设置为32point，而上下边距则各位64。

就这样，上面的方法就是简单的给出iPad中正确的位置。

编译并运行程序，可以看到scene中有3个鼹鼠，它们的位置已经设置正确！你最好在iPhone 3.5-inch, iPhone 4-inch, iPad, 和 iPad Retina设备上都运行一下，以确保位置的正确。

![](/images/2013/10/33.png)

###<a id="tcas"></a>弹出鼹鼠

至此，我们已经把鼹鼠放置好了，下面我们添加一些代码让鼹鼠从洞里面跳出来吧。

首先，将这些sprite(鼹鼠)的zPosition从999设置为2，这样就可以把鼹鼠藏起来了。

然后，将下面的代码添加到update:方法中：

```objc
for (SKSpriteNode *mole in self.moles) {
    if (arc4random() % 3 == 0) {
        if (!mole.hasActions) {
            [self popMole:mole];
        }
    }
}
```
需要注意的是每帧的显示都会调用update方法。该方法被调用的时候我们都会尝试着弹出一些鼹鼠。在代码中循环遍历处理了每个鼹鼠，并给每个鼹鼠1/3的机会从洞中弹出来。不过记住我们只能弹出那么还没有弹出来的鼹鼠——很简单的一个判断方法就是检查一下sprite的属性hasActions返回的值，如果还有action在运行，那么hasActions将返回YES。

接着，实现一下popMole方法：

```objc
- (void)popMole:(SKSpriteNode *)mole
{
	SKAction *easeMoveUp = [SKAction moveToY:mole.position.y + mole.size.height duration:0.2f];
    easeMoveUp.timingMode = SKActionTimingEaseInEaseOut;
    SKAction *easeMoveDown = [SKAction moveToY:mole.position.y duration:0.2f];
    easeMoveDown.timingMode = SKActionTimingEaseInEaseOut;
    SKAction *delay = [SKAction waitForDuration:0.5f];
 
    SKAction *sequence = [SKAction sequence:@[easeMoveUp, delay, easeMoveDown]];
    [mole runAction:sequence];
}
```

上面的代码使用了Sprite Kit中的一些action，让鼹鼠弹出洞来，并暂停半秒钟，然后在弹回去。我们来细看一下上面代码的意思：

1. 创建一个action来将鼹鼠沿着Y轴移动鼹鼠高度的距离，这样就能将鼹鼠正好放置到洞上面。
2. 为了让移动行为看起来更自然一点，将action的timingMode设置为SKActionTimingEaseInEaseOut。这样可以让action在开始和结束时速度慢一点，这样鼹鼠看起来会有加速和减速的效果，看起来就会更自然一点了。
3. 创建一个action将鼹鼠移回原处，这个action跟上一个类似，只不过使用鼹鼠当前Y轴的位置。
4. 创建一个action，该action会让鼹鼠停留在洞口半秒钟。
5. 按顺序运行这些action：move up，delay和move down。

搞定！编译并运行程序，可以看到鼹鼠会从它们的洞口弹出来！

![](/images/2013/10/34.png)

###<a id="hqhc"></a>何去何从

本文的代码工程在[这里](http://cdn3.raywenderlich.com/downloads/WhackAMoleSK1.zip)。

下一篇文章`Sprite Kit教程：制作一个通用程序 1`中会给鼹鼠添加一些可爱的动画(笑和被击中)，并添加一个玩法——打击鼹鼠，并赚取点数，并添加一些音效。


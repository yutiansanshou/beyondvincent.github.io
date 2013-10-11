---
layout: post
title: "Sprite Kit教程：初学者 3"
date: 2013-10-12 00:06
comments: true
categories: iOS翻译
published: ture

---

{% img /images/2013/09/18.png %}
<!--more-->



注：本文译自[`Sprite Kit Tutorial for Beginners`](http://www.raywenderlich.com/42699/spritekit-tutorial-for-beginners)



###**目录**
* Sprite Kit的优点和缺点
* Sprite Kit vs Cocos2D-iPhone vs Cocos2D-X vs Unity
* Hello, Sprite Kit!
* 横屏显示
* 移动怪兽
* 发射炮弹
* [碰撞检测和物理特性: 概述](#pzjcgs)
* [碰撞检测和物理特性: 实现](#pzjcsx)
* [收尾](#sw)
* [何去何从?](#hqhc)




###<a id="pzjcgs"></a>碰撞检测和物理特性: 概述

至此我们已经可以让炮弹任意的发射了——现在我们要让忍者利用炮弹来消灭这些怪物。下面就添加一些代码来给炮弹与怪物相交做检测。

Sprite Kit内置了一个物理引擎，这非常的棒！该物理引擎不仅可以模拟现实运动，还能进行碰撞检测。

下面我们就在游戏中使用Sprite Kit的物理引擎来检测炮弹与怪物的碰撞。首先，我们来看看需要做些神马事情：

* `物理世界的配置`。物理世界是一个模拟的空间，用来进行物理计算。默认情况下，在场景(scene)中已经创建好了一个，我们可以对其做一些属性配置，例如重力感应。
* `为精灵(sprite)创建对应的物体(physics bodies)`。在Sprite Kit中，为了碰撞检测，我们可以为每个精灵创建一个相应的形状，并设置一些属性,这就称为`物体(physics body)`。注意：图文的形状不一定跟精灵的外形一模一样。一般情况，这个形状都是简单的、大概的(而不用精确到像素级别)——毕竟这已经足以够大多数游戏使用了。
* `将精灵分类`。在物体(physics body)上可以设置的一个属性是`category`，该属性是一个位掩码(bitmask)。通过该属性可以将精灵分类。在本文的游戏中，有两个类别——一类是炮弹，另一类则是怪物。设置之后，当两种物体相互碰撞时，就可以很容易的通过类别对精灵做出相应的处理。
* `设置一个contact(触点) delegate`。还记得上面提到的物理世界吗？我们可以在物理世界上设置一个`contact delegate`，通过该delegate，当两个物体碰撞时，可以收到通知。收到通知后，我们可以通过代码检查物体的类别，如果是怪物和炮弹，那么就做出相应的动作！

上面大致介绍了一下游戏策略，下面就来看看如何实现！


###<a id="pzjcsx"></a>碰撞检测和物理特性: 实现

首先在`MyScene.m`文件顶部添加如下两个常量：

```objc

static const uint32_t projectileCategory     =  0x1 << 0;
static const uint32_t monsterCategory        =  0x1 << 1;

```

上面设置了两个类别，记住需要用位(bit)的方式表达——一个用于炮弹，另一个则是怪物。

`注意:`看到上面的语法你可能感到奇怪。在Sprite Kit中category是一个32位整数，当做一个位掩码(bitmask)。这种表达方法比较奇特：在一个32位整数中的每一位表示一种类别(因此最多也就只能有32类)。在这里，第一位表示炮弹，下一位表示怪兽。

接着，在`initWithSize`中，将下面的代码添加到位置：添加player到场景涉及代码的后面。

```objc
self.physicsWorld.gravity = CGVectorMake(0,0);
self.physicsWorld.contactDelegate = self;
```
上面的代码将物理世界的重力感应设置为0，并将场景设置位物理世界的代理（当有两个物体碰撞时，会受到通知）。

在`addMonster`方法中，将如下代码添加创建怪兽相关代码后面：

```objc
monster.physicsBody = [SKPhysicsBody bodyWithRectangleOfSize:monster.size]; // 1
monster.physicsBody.dynamic = YES; // 2
monster.physicsBody.categoryBitMask = monsterCategory; // 3
monster.physicsBody.contactTestBitMask = projectileCategory; // 4
monster.physicsBody.collisionBitMask = 0; // 5
```
来看看上面代码意思：

1. 为怪兽创建一个对应的物体。此处，物体被定义为一个与怪兽相同尺寸的矩形(这样与怪兽形状比较接近)。
2. 将怪兽设置位`dynamic`。这意味着物理引擎将不再控制这个怪兽的运动——我们自己已经写好相关运动的代码了。
3. 将categoryBitMask设置为之前定义好的`monsterCategory`。
4. `contactTestBitMask`表示与什么类型对象碰撞时，应该通知contact代理。在这里选择炮弹类型。
5. `collisionBitMask`表示物理引擎需要处理的碰撞事件。在此处我们不希望炮弹和怪物被相互弹开——所以再次将其设置为0。

接着在`touchesEnded:withEvent:`方法中设置炮弹位置的代码后面添加如下代码。

```objc

projectile.physicsBody = [SKPhysicsBody bodyWithCircleOfRadius:projectile.size.width/2];
projectile.physicsBody.dynamic = YES;
projectile.physicsBody.categoryBitMask = projectileCategory;
projectile.physicsBody.contactTestBitMask = monsterCategory;
projectile.physicsBody.collisionBitMask = 0;
projectile.physicsBody.usesPreciseCollisionDetection = YES;

```

在上面的代码中跟之前的类似，只不过有些不同，我们来看看：
1. 为了更好的效果，炮弹的形状是圆形的。
2. `usesPreciseCollisionDetection`属性设置为YES。这对于快速移动的物体非常重要(例如炮弹)，如果不这样设置的话，有可能快速移动的两个物体会直接相互穿过去，而不会检测到碰撞的发生。

接着，添加如下方法，当炮弹与怪物发生碰撞时，会被调用。注意这个方法是不会被自动调用，稍后会看到我们如何调用它。

```objc
- (void)projectile:(SKSpriteNode *)projectile didCollideWithMonster:(SKSpriteNode *)monster {
    NSLog(@"Hit");
    [projectile removeFromParent];
    [monster removeFromParent];
}
```

当怪物和炮弹发生碰撞，上面的代码会将他们从场景中移除。很简单吧！

下面该实现contact delegate方法了。将如下方法添加到文件中：

```objc
- (void)didBeginContact:(SKPhysicsContact *)contact
{
    // 1
    SKPhysicsBody *firstBody, *secondBody;
 
    if (contact.bodyA.categoryBitMask < contact.bodyB.categoryBitMask)
    {
        firstBody = contact.bodyA;
        secondBody = contact.bodyB;
    }
    else
    {
        firstBody = contact.bodyB;
        secondBody = contact.bodyA;
    }
 
    // 2
    if ((firstBody.categoryBitMask & projectileCategory) != 0 &&
        (secondBody.categoryBitMask & monsterCategory) != 0)
    {
        [self projectile:(SKSpriteNode *) firstBody.node didCollideWithMonster:(SKSpriteNode *) secondBody.node];
    }
}
```

还记得之前给物理世界设置的`contactDelegate`吗？当两个物体发生碰撞之后，就会调用上面的方法。

在上面的方法中，可以分为两部分来理解：

1. 该方法会传递给你发生碰撞的两个物体，但是并不一定符合特定的顺序(如炮弹在前，或者炮弹在后)。所以这里的代码是通过物体的category bit mask来对其进行排序，以便后续做出正确的判断。注意，这里的代码来自苹果提供的Adventure示例。
2. 最后，检测一下这两个碰撞的物体是否就是炮弹和怪物，如果是的话就调用之前的方法。

最后一步，为了编译器没有警告，确保private interface 中添加一下`SKPhysicsContactDelegate`：
```objc
@interface MyScene () <SKPhysicsContactDelegate>
```
现在编译并运行程序，可以发现，当炮弹与怪物接触时，他们就会消失！


###<a id="sw"></a>收尾

现在，本文的游戏快完成了。接下来我们就来为游戏添加音效和音乐，以及一些简单的游戏逻辑吧。

苹果提供的Sprite Kit里面并没有音频引擎(Cocos2D中是有的)，不过我们可以通过action来播放音效，并且可以使用AVFoundation播放后台音乐。

在工程中我已经准备好了一些音效和很酷的后台音乐，在本文开头已经将resources添加到工程中了，现在只需要播放它们即可！

首先在`ViewController.m`文件顶部添加如下import：

```objc
@import AVFoundation;
```

上面的语法是iOS 7中新的modules功能 —— 只需要使用新的关键字@import，就可以框架的头文件和库文件添加到工程中，这功能非常方便。要了解更多相关内容，请看到[iOS 7 by Tutorials](http://www.raywenderlich.com/store/ios-7-by-tutorials)中的第十章内容中的：What’s New with Objective-C and Foundation。

接着添加一个新的属性和private interface：

```objc
@interface ViewController ()
@property (nonatomic) AVAudioPlayer * backgroundMusicPlayer;
@end
```
接着将下面的代码添加到`viewWillLayoutSubviews`方法中(在`[super viewWillLayoutSubviews]`后面)：

```objc
NSError *error;
NSURL * backgroundMusicURL = [[NSBundle mainBundle] URLForResource:@"background-music-aac" withExtension:@"caf"];
self.backgroundMusicPlayer = [[AVAudioPlayer alloc] initWithContentsOfURL:backgroundMusicURL error:&error];
self.backgroundMusicPlayer.numberOfLoops = -1;
[self.backgroundMusicPlayer prepareToPlay];
[self.backgroundMusicPlayer play];
```

上面的代码会开始无限循环的播放后台音乐。

下面我们来看看如何处理音效。切换到`MyScene.m`文件中，并将下面这行代码添加到`touchesEnded:withEvent:`方法的顶部：

```objc
[self runAction:[SKAction playSoundFileNamed:@"pew-pew-lei.caf" waitForCompletion:NO]];

```

如上，一行代码就可以播放音效了，很简单吧！

下面，我们创建一个新的创建和layer，用来显示`你赢了(You Win)`或`你输了(You Lose)`。用模板`iOS\Cocoa Touch\Objective-C class`创建一个新的文件，将其命名为`GameOverScene`，并让其继承自`SKScene`，然后点击`Next`和`Create`。

接着用如下代码替换`GameOverScene.h`中的内容：

```objc
#import <SpriteKit/SpriteKit.h>
 
@interface GameOverScene : SKScene
 
-(id)initWithSize:(CGSize)size won:(BOOL)won;
 
@end
```

在上面的代码中导入了Sprite Kit头文件，并声明了一个特定的初始化方法，该方法的第一个参数用来定位显示的位置，第二个参数won用来判断用户是否赢了。

接着用下面的代码替换`GameOverLayer.m`中的内容：

```objc
#import "GameOverScene.h"
#import "MyScene.h"
 
@implementation GameOverScene
 
-(id)initWithSize:(CGSize)size won:(BOOL)won {
    if (self = [super initWithSize:size]) {
 
        // 1
        self.backgroundColor = [SKColor colorWithRed:1.0 green:1.0 blue:1.0 alpha:1.0];
 
        // 2
        NSString * message;
        if (won) {
            message = @"You Won!";
        } else {
            message = @"You Lose :[";
        }
 
        // 3
        SKLabelNode *label = [SKLabelNode labelNodeWithFontNamed:@"Chalkduster"];
        label.text = message;
        label.fontSize = 40;
        label.fontColor = [SKColor blackColor];
        label.position = CGPointMake(self.size.width/2, self.size.height/2);
        [self addChild:label];
 
        // 4
        [self runAction:
            [SKAction sequence:@[
                [SKAction waitForDuration:3.0],
                [SKAction runBlock:^{
                    // 5
                    SKTransition *reveal = [SKTransition flipHorizontalWithDuration:0.5];
                    SKScene * myScene = [[MyScene alloc] initWithSize:self.size];
                    [self.view presentScene:myScene transition: reveal];
                }]
            ]]
        ];
 
    }
    return self;
}
 
@end
```

上面的代码可以分为4部分内容，我们来分别看看：

1. 将背景色设置为白色(与主场景一样颜色)。
2. 根据`won`参数，将信息设置为"You Won"或"You Lose"。
3. 这里的代码是利用Sprite Kit将一个文本标签显示到屏幕中。如代码所示，只需要选择一个字体，并设置少量的参数即可，也非常简单。
4. 设置并运行有个有两个action的sequence。为了看起来方便，此处我将它们放到一块(而不是为每个action创建单独的一个变量)。首先是等待3秒，然后是利用`runBlock`action来运行一些代码。
5. 演示了在Sprite Kit中如何过渡到新的场景。首先可以选择任意的一种不同的动画过渡效果，用于场景的显示，在这里选择了翻转效果(持续0.5秒)。然后是创建一个想要显示的场景，接着使用self.view的方法`presentScene:transition:`来显示出场景。

OK，万事俱备，只欠东风了！现在只需要在主场景中，适当的情况下加载game over scene就可以了。

首先，在`MyScene.m`中导入新的场景：

```objc
#import "GameOverScene.h"
```

然后，在`addMonster`中，用下面的代码替换最后一行在怪物上运行action的代码：

```objc
SKAction * loseAction = [SKAction runBlock:^{
    SKTransition *reveal = [SKTransition flipHorizontalWithDuration:0.5];
    SKScene * gameOverScene = [[GameOverScene alloc] initWithSize:self.size won:NO];
    [self.view presentScene:gameOverScene transition: reveal];
}];
[monster runAction:[SKAction sequence:@[actionMove, loseAction, actionMoveDone]]];
```

上面创建了一个"lose action"，当怪物离开屏幕时，显示game over场景。

在这里为什么`loseAction`要在`actionMoveDone`之前运行呢？
原因在于如果将一个精灵从场景中移除了，那么它就不在处于场景的层次结构中了，也就不会有action了。所以需要过渡到lose场景之后，才能将精灵移除。不过，实际上actionMoveDone永远都不会被调用——因为此时已经过渡到新的场景中了，留在这里就是为了达到教学的目的。

现在，需要处理一下赢了的情况。在private interface中添加一个新的属性：

```objc
@property (nonatomic) int monstersDestroyed;
```

然后将如下代码添加到`projectile:didCollideWithMonster:`的底部：

```objc
self.monstersDestroyed++;
if (self.monstersDestroyed > 30) {
    SKTransition *reveal = [SKTransition flipHorizontalWithDuration:0.5];
    SKScene * gameOverScene = [[GameOverScene alloc] initWithSize:self.size won:YES];
    [self.view presentScene:gameOverScene transition: reveal];
}
```

编译并运行程序，尝试一下赢了和输了会看到的画面！

###<a id="hqhc"></a>何去何从?

至此`Sprite Kit教程：初学者`结束！这里可以下到[完整的代码](http://cdn2.raywenderlich.com/downloads/SpriteKitSimpleGame2.zip)。

希望本文能帮助你学习Sprite Kit，并写出你自己的游戏！

如果你希望学习更多相关Sprite Kit内容，可以看看这本书：[iOS Games by Tutorials](http://www.raywenderlich.com/store/ios-7-by-tutorials)。本书会告诉你需要知道的内容——从物理，到tile map，以及特定的系统，甚至是制作自己的关卡编辑器。

……Sprite Kit教程：初学者 3 结束……

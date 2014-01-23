---
layout: post
title: "如何自定义iOS中的控件"
date: 2014-01-20 17:00
comments: true
categories: iOS探索
published: true

---

![](/images/2014/01/18.png)

<!--more-->

本文译自[How to build a custom control in iOS](http://www.thinkandbuild.it/how-to-build-a-custom-control-in-ios/)。大家要是有什么问题，可以直接在[twitter](https://twitter.com/bitwaker)上联系原作者，当然也可以在最后的评论中回复我。

下面先来看看目录：

1. 子类化UIControl
	* 绘制用户界面
		* 绘制背景
		* 绘制用户的可操作区域
		* 绘制手柄
	* 跟踪用户的操作
		* 开始跟踪
		* 持续跟踪
		* 结束跟踪
	* Target-Action模式
2. 如何使用自定义控件
3. 总结
4. 代码下载

在开发过程中，有时候UIKit的标准控件并不能满足我们的需求，例如你需要一个控件能支持用户方便的选择0-360°之间的一个角度值，此时就需要根据自己的需求自定义控件了。

对于选择角度值的控件可以这样实现：创建一个圆形的滑块，用户通过拖动手柄操作就能选择角度值。实际上这样的控件在别的一些平台中你可能看到过，但是在UIKit中并没有。

本文就实现一个选择角度值的控件来介绍控件的自定义。下面先来看看到底要做成什么样子：

![](/images/2014/01/12.png)


###1. 子类化UIControl###

`UIControl`是UIView的子类，它又是所有UIKit控件的父类(例如UIButton、UISlider和UISwitch等)。

UIControl的主要作用是创建相应的逻辑将action分发到对应的target，另外90%的情况下，它会根据自身的状态(例如Highlighted, Selected和Disabled等)来绘制用户界面。

通过UIControl，我们主要管理3个重要的任务：

* 绘制用户界面
* 跟踪用户的操作
* Target-Action模式

在本文的圆形滑块中，我们要做如下一些事情：

定制一个用户界面(圆形滑块本身)，通过该界面用户可以通过手柄进行界面交互。用户的交互操作会被转换为控件target对应的action(控件将滑块按钮的frame origin转换为0-360之间的一个值，并用于target/action上)。

建议在学习本文的时候从文章尾部的连接中下载完整的示例工程。

下面我将从上面列出的3个重要任务一一进行分解介绍。

这些步骤都是模块化的，所以如果你对界面的绘制不感兴趣，可以跳过`绘制用户界面`，直接学习后面的步骤。

打开工程文件中的`TBCircluarSlider.m`文件。然后开始学习下面的内容。

####1.1 绘制用户界面####

我比较喜欢使用Core Graphics，唯一用到UIKit的就是通过textfield来显示滑块的值。

`提醒`：此处需要用到一些`Core Graphics`知识，如果你不懂也没多大关系，我会尽量把代码做详细的讲解。

我们先来看看控件的不同组成部分，这样更有利于后面的学习。

首先，是用一个`黑色的圆环`当做滑块的背景。

![](/images/2014/01/13.png)

`可操作区域(active area)`是一个从蓝色到紫色的梯度渐变效果。

![](/images/2014/01/14.png)

用户通过拖拽下面的这个手柄按钮来选择值：

![](/images/2014/01/15.png)

最后，用于显示选中值的`TextField`。在下一版中，我计划让用户可以通过键盘输入角度值。

![](/images/2014/01/16.png)

控件界面的绘制主要使用drawRect函数，首选我们需要获取到当前使用的图形上下文，如下代码所示：

```objc
CGContextRef ctx = UIGraphicsGetCurrentContext();
```

#####1.1.1 绘制背景#####

背景是360°的，所以只要用CGContextAddArc给图形上下文添加正确的path，并设置正确的stroke即可。

下面的代码可以就可以完成背景的绘制：

```objc
//Add the arc path
CGContextAddArc(ctx, self.frame.size.width/2, self.frame.size.height/2, radius, 0, M_PI *2, 0);
 
//Set the stroke colour
[[UIColor blackColor]setStroke];
 
//set Line width and cap
CGContextSetLineWidth(ctx, TB_BACKGROUND_WIDTH);
CGContextSetLineCap(ctx, kCGLineCapButt);
 
//draw it!
CGContextDrawPath(ctx, kCGPathStroke);
```

`CGContextArc`函数的参数包括图形上下文，弧度的中心坐标点，以及半径(是一个私有变量)，接着是弧度开始和结束时的角度(在TBCircularSlider.m文件的头部可以看到一些关于数学计算的方法)，最后一个参数标示绘制的方向，0表示逆时针方向。

接下来的3行的代码是用来设置一些信息的，例如颜色和线条宽度等。最后使用`CGContextDrawPath`方法完成背景的绘制。

#####1.1.2 绘制用户的可操作区域#####

这部分需要利用一点小技巧才行。此处我们绘制一个线性渐变的掩码图片，下面看看原理：

![](/images/2014/01/17.png)

此处掩码图片的工作原理是可以看到原始渐变矩形框的一个孔。

在这里绘制的弧度有一个阴影，这是创建掩码图时使用了一点模糊的效果。

下面是创建掩码图的相关代码：

```objc
UIGraphicsBeginImageContext(CGSizeMake(320,320));
CGContextRef imageCtx = UIGraphicsGetCurrentContext();
 
CGContextAddArc(imageCtx, self.frame.size.width/2  , self.frame.size.height/2, radius, 0, ToRad(self.angle), 0);
[[UIColor redColor]set];
 
//Use shadow to create the Blur effect
CGContextSetShadowWithColor(imageCtx, CGSizeMake(0, 0), self.angle/20, [UIColor blackColor].CGColor);
 
//define the path
CGContextSetLineWidth(imageCtx, TB_LINE_WIDTH);
CGContextDrawPath(imageCtx, kCGPathStroke);
 
//save the context content into the image mask
CGImageRef mask = CGBitmapContextCreateImage(UIGraphicsGetCurrentContext());
UIGraphicsEndImageContext();
```

在上面的代码中首先创建了一个图形上下文，然后设置了一下阴影。通过`CGContextSetShadowWithColor`方法，我们可以设置如下内容：

* 上下文
* 偏移量(此处不需要)
* 模糊值(该值是通过参数控制的：使用当前的角度除以20，当用户与此控件交互时，以此获得一个简单的动画模糊值)
* 颜色

接着是根据当前的角度绘制一个相应的弧度。

例如，如果当前的角度变量是360°，那么就绘制一个圆弧，如果是90°，就绘制一个弧度为90°的一个弧。最后，利用`CGBitmapContextCreateImage`方法获取一张图片（刚刚绘制的弧）。这个图片就是我们所需要的掩码图了。

裁剪上下文：

现在我们已经有一个渐变的掩码图了。接着利用函数`CGContextClipToMask`对上下文进行裁剪——给该函数传入上面刚刚创建好的掩码图。代码如下所示：

```objc
CGContextClipToMask(ctx, self.bounds, mask);
```

最后我们来绘制渐变效果，代码如下所示：

```objc
//Define the colour steps
CGFloat components[8] = {
    0.0, 0.0, 1.0, 1.0,     // Start color - Blue
    1.0, 0.0, 1.0, 1.0 };   // End color - Violet
 
CGColorSpaceRef baseSpace = CGColorSpaceCreateDeviceRGB();
CGGradientRef gradient = CGGradientCreateWithColorComponents(baseSpace, components, NULL, 2);
 
//Define the gradient direction 
CGPoint startPoint = CGPointMake(CGRectGetMidX(rect), CGRectGetMinY(rect));
CGPoint endPoint = CGPointMake(CGRectGetMidX(rect), CGRectGetMaxY(rect));
 
    //Choose a colour space
CGColorSpaceRelease(baseSpace), baseSpace = NULL;    
 
//Create and Draw the gradient
CGContextDrawLinearGradient(ctx, gradient, startPoint, endPoint, 0);
CGGradientRelease(gradient), gradient = NULL;
```

绘制渐变效果需要很多处理，不过我们可以将其分为4部分：

* 定义颜色的变化范围
* 定义渐变的方向
* 选择颜色空间
* 创建并绘制渐变

最终的显示效果(看到渐变矩形框的一部分)要归功于之前创建的掩码图。

另外，为了在背景边框模拟光线反射，我添加了一些灯光效果。

#####1.1.3 绘制手柄#####

下面我们根据当前的角度值，在的正确位置绘制出手柄。

实际上，在绘制过程中，这一步非常简单，复杂一点的就是计算一下手柄所在的位置。

这里我们需要使用三角函数将一个`标量值(scalar number)`转换为`CGPoint`。不要担心有多复杂，只需要使用Sin和Cos函数就可以完成。代码如下所示：

```objc
-(CGPoint)pointFromAngle:(int)angleInt{
     
    //Define the Circle center
    CGPoint centerPoint = CGPointMake(self.frame.size.width/2 - TB_LINE_WIDTH/2, self.frame.size.height/2 - TB_LINE_WIDTH/2);
     
    //Define The point position on the circumference
    CGPoint result;
    result.y = round(centerPoint.y + radius * sin(ToRad(-angleInt))) ;
    result.x = round(centerPoint.x + radius * cos(ToRad(-angleInt)));
     
    return result;
}
```

上面的代码中，指定一个角度值，然后计算出在圆周上面的位置，当然，这里需要圆周的中心点和半径。

使用sin函数在使用sin函数时，需要一个Y坐标值，而cos函数则需要X坐标值。

需要注意的是此处每个函数返回的值都认为半径为1，所以需要将所得结果乘以我们指定的半径大小，并相对于圆周的中心做计算。

希望下面的公式对你的理解有所帮助：

```objc
point.y = center.y + (radius * sin(angle)); 
point.x = center.x + (radius * cos(angle));
```

通过上面的计算，现在我们已经知道手柄的具体位置了，所以，接下来就直接将手柄绘制到指定位置即可，如下代码所示：

```objc
-(void) drawTheHandle:(CGContextRef)ctx{
     
    CGContextSaveGState(ctx);
     
    //I Love shadows
    CGContextSetShadowWithColor(ctx, CGSizeMake(0, 0), 3, [UIColor blackColor].CGColor);
     
    //Get the handle position!
    CGPoint handleCenter =  [self pointFromAngle: self.angle];
     
    //Draw It!
    [[UIColor colorWithWhite:1.0 alpha:0.7]set];
    CGContextFillEllipseInRect(ctx, CGRectMake(handleCenter.x, handleCenter.y, TB_LINE_WIDTH, TB_LINE_WIDTH));
     
    CGContextRestoreGState(ctx);
}
```

具体操作步骤如下：

* 保存当前的上下文(当在一个单独的函数中进行绘制任务时，将上下文的状态进行保存是编程的一个好习惯)。
* 给手柄设置一些阴影效果
* 定义手柄的颜色，然后利用`CGContextFillEllipseInRect`将其绘制出来。

我们在drawRect函数的最后调用上面这个方法：

```objc
[self drawTheHandle:ctx];
```

至此，我们就完成了绘制部分的任务。

####1.2 跟踪用户的操作####

在UIControl的子类中，我们可以`override`3个特殊的方法来提供一个自定义的跟踪行为

#####1.2.1 开始跟踪#####

当在控件的bound内发生了一个触摸事件，首先会调用控件的`beginTrackingWithTouch`方法。

我们就看看如何`override`这个方法吧：

```objc
-(BOOL)beginTrackingWithTouch:(UITouch *)touch withEvent:(UIEvent *)event{
    [super beginTrackingWithTouch:touch withEvent:event];
 
    //We need to track continuously
    return YES;
}
```

该函数返回的BOOl值决定着：当触摸事件是dragged时，是否需要响应。在我们这里的自定义控件中，是需要跟踪用户的dragging，所以返回YES。

上面这个函数有两个参数：touch对象和事件。

#####1.2.2 持续跟踪#####

在上一个方法中我们指定了这里的自定义控件需要跟踪一个持续的事件，所以当用户进行drag时，会调用一个特殊的方法：`continueTrackingWithTouch`：

```objc
-(BOOL)continueTrackingWithTouch:(UITouch *)touch withEvent:(UIEvent *)event
```

该方法返回的BOOL值标示是否继续跟踪touch事件。

通过该方法我们可以根据touch位置对用户的操作进行过滤。例如，我们可以：仅当touch位置与手柄位置相交的时候才激活控件(activate control)。不过在这里我们的控制逻辑并不是这样的，我们希望用户点击任何位置都能对手柄做出相应的位置处理。

本文的该方法负责更新手柄的位置(在后面的一节中会看到我们把该位置信息传递给对应的target上)。

对上面这个方法的override代码如下所示：

```objc
-(BOOL)continueTrackingWithTouch:(UITouch *)touch withEvent:(UIEvent *)event{
    [super continueTrackingWithTouch:touch withEvent:event];
 
    //Get touch location
    CGPoint lastPoint = [touch locationInView:self];
 
    //Use the location to design the Handle
    [self movehandle:lastPoint];
 
        //We'll see this function in the next section:
    [self sendActionsForControlEvents:UIControlEventValueChanged];
 
    return YES;
}
```

上面的代码中，首先利用`locationInView`获取到touch的位置，然后将该位置传递给`moveHandle`方法，该方法会将传入的值转换为一个有效的手柄位置(a valid handle position)。

此处“a valid position”的意思是什么呢？

此控件的手柄只能在背景圆弧定义的边界范围内做移动，但是我们不希望强制要求用户必须在很小的圆弧内才可以移动手柄，如果非要这样的话，用户体验会非常的糟糕。

`moveHandle`的任务就是负责把任意的位置值转变为手柄可移动的值，另外，另外，在该函数中，还对指定的滑块角度值做了转换，代码如下所示：

```objc
-(void)movehandle:(CGPoint)lastPoint{
     
    //Get the center
    CGPoint centerPoint = CGPointMake(self.frame.size.width/2,
                                                                            self.frame.size.height/2);
     
    //Calculate the direction from the center point to an arbitrary position.
    float currentAngle = AngleFromNorth(centerPoint, 
                                                                                lastPoint, 
                                                                                NO);
    int angleInt = floor(currentAngle);
     
    //Store the new angle
    self.angle = 360 - angleInt;
 
    //Update the textfield 
    _textField.text =  [NSString stringWithFormat:@"%d", 
                                                                                                    self.angle];
     
    //Redraw
    [self setNeedsDisplay];
} 
```

上面代码中，实际上主要任务都是在`AngleFromNorth`方法中处理的：根据两个point，就会返回一个连接这两点对应的一个角度关系，`AngleFromNorth`方法的实现如下所示：

```objc
static inline float AngleFromNorth(CGPoint p1, CGPoint p2, BOOL flipped) {
    CGPoint v = CGPointMake(p2.x-p1.x,p2.y-p1.y);
    float vmag = sqrt(SQR(v.x) + SQR(v.y)), result = 0;
    v.x /= vmag;
    v.y /= vmag;
    double radians = atan2(v.y,v.x);
    result = ToDeg(radians);
    return (result >=0  ? result : result + 360.0);
}
```

提醒：`angleFromNorth`方法并不是我的原创，我是直接从苹果提供的OSX示例clockControl中拿过来用的。

在上面的代码中，获得了角度值以后，将其存储到`angle`中，然后更新一下textfield的值。

接着调用的`setNeedDisplay`是为了确保`drawRect`被调用，以尽快在界面上做出相应的更新。

#####1.2.3 结束跟踪#####

当跟踪结束的时候，会调用下面这个方法：

```objc
-(void)endTrackingWithTouch:(UITouch *)touch withEvent:(UIEvent *)event{
    [super endTrackingWithTouch:touch withEvent:event];
}
```

在本文中，我们并不需要override该方法。如果当用户完成控件的界面操作时，你希望做一些处理，那么该方法会非常有用。

####1.3 Target-Action模式####

至此，圆形滑块控件可以工作了，你可以drag手柄，并能看到textfield中值的改变。

发送action——控件事件

如果希望自己定制的控件与UIControl行为保持一致，那么当控件的值发生变化时，需要进行通知处理：使用`sendActionsForControlEvents`方法，并制定特定的事件类型，值改变对应的事件一般是`UIControlEventValueChanged`。

苹果已经预定义了许多事件类型(Xcode中，在UIControlEventValueChanged上`cmd + 鼠标单击`)。如果你的控件是继承自UITextField，那么你可能会对`UIControlEventEdigitingDidBegin`感兴趣，如果你要做一个touch Up action，那么可以使用UIControlTouchUpInside。

如果你注意的话，在本文前部分的continueTrackingWithTouch方法里面，我们调用了`sendActionsForControlEvents`方法：

```objc
[self sendActionsForControlEvents:UIControlEventValueChanged];
```

这样处理之后，当控件值发生变化时，每一个对象(观察者——注册该事件)都会收到响应的通知。

###2. 如何使用自定义控件###

到这里，我们的控件定制完毕，下面介绍如何在程序中使用自定义的控件。

打开文件`TBViewController.m`，看看`viewDidLoad`方法里面的代码：

```objc
- (void)viewDidLoad
{
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor colorWithRed:0.1 green:0.1 blue:0.1 alpha:1];
     
    //Create the Circular Slider
    TBCircularSlider *slider = [[TBCircularSlider alloc]initWithFrame:CGRectMake(0, 60, TB_SLIDER_SIZE, TB_SLIDER_SIZE)];
     
    //Define Target-Action behaviour
    [slider addTarget:self action:@selector(newValue:) forControlEvents:UIControlEventValueChanged];
     
    [self.view addSubview:slider];
}
```

在上面的代码中，给view设置了一个背景色，并通过调用`initWithFrame`方法实例化了一个圆形滑块(自定义的控件)。

注意：UIControl继承自UIView，所以它继承了UIView的所有方法。

接着定义了如何与该控件进行交互：使用`addTarget:action:forControlEvent:`方法。

该方法只是给控件的特定事件设置一下target-action。如果你还记得的话，上面层介绍过，每当用户移动手柄时，圆形滑块都会发送一个UIControlEventValueChanged事件。所以我们可以通过下面的代码为该事件注册一个action：

```objc
[slider addTarget:self action:@selector(newValue:) forControlEvents:UIControlEventValueChanged]; 
```

这样我们就可以创建一个`**newValue**`方法来处理值发生改变时的一些事情：

```objc
-(void)newValue:(TBCircularSlider*)slider{
    NSLog(@"Slider Value %d",slider.angle);
}
```

结合Target-Action，所以函数会受到action的发送者，此处是slider，通过这个slider，就能直接获取到角度值。

###3. 总结###

根据本文的具体步骤，你可以构建`任意你想要的控件`。

当然，也有其它一些方法来构建自定控件，不过本文基本上是按照苹果的建议来做的。

点击下图，下载代码

###4. 代码下载###
[![](/images/2013/11/34.jpg)](https://github.com/ariok/TB_CircularSlider)

本文由破船译自[How to build a custom control in iOS](http://www.thinkandbuild.it/how-to-build-a-custom-control-in-ios/)。大家要是有什么问题，可以直接在[twitter](https://twitter.com/bitwaker)上联系原作者，当然也可以在下面的评论中回复我。

---
layout: post
title: "Objective-C中的Literals"
date: 2014-01-23 15:30
comments: true
categories: iOS探索
published: true

---

![](/images/2014/01/19.png)

<!--more-->

本文译自[Guided tour through Objective-C Literals](http://www.thinkandbuild.it/guided-tour-through-objective-c-literals/)。大家要是有什么问题，可以直接在[twitter](https://twitter.com/bitwaker)上联系原作者，当然也可以在最后的评论中回复我。

苹果在2012年就已经把Literals加入到LLVM中，利用Literals，不仅可以方便快捷的创建某些特定数据类型，还可以简化代码量，加强代码的可读性。

下面先来看看目录：

1. NSNumber
2. NSArray
3. NSDictionary
4. Expressions

下面我们就让代码来说话吧。

###1. NSNumber###

曾经你是否一直这样来实例化`NSNumber`：

```objc
NSNumber *integer = [NSNumber numberWithInt:19];
```

是不是感觉比较麻烦，现在好了，通过Literal，只需要如下一行简洁的代码即可：

```objc
NSNumber *integer = @19
```

在上面的代码中，使用了`@`符号，这跟创建NSString一样(只是不用双引号吧了)，这样一来，就非常容易记住和使用啦。

不仅如此，我们还可以利用C语言中的后缀风格来定义NSNumber，如下代码所示：

```objc
NSNumber *unsignedInteger = @19U;   //Unsigned Integer
NSNumber *longInteger = @19L;       //Long Integer
NSNumber *floatNumber = @19.5493F;  //Float 
 
NSNumber *boolean = @YES; //        //BOOL
```

###2. NSArray###

有些编程语言创建数组是非常简单的，例如PHP。但是在引入Literal之前，Objective-C中创建数组的语法确不简单。如下代码所示：

```objc
NSArray *array = [NSArray arrayWithObjects: [NSNumber numberWithInt:10],
                                            @"A String!",
                                            [NSNumber numberWithFloat:10.654F],
                                             nil];
```

上面代码中不仅初始化对象复杂，还需要额外添加一个nil。但是要用Literal，看起来完全不一样了：

```objc
NSArray *array_l = @[@10, @"A string", @10.645F];
```

如上所示，利用Literal，可以通过`@[]`轻松的搞定数组初始化，并且省掉最后的`nil`。实际上编译器会把上面的代码替换为`[NSArray arrayWithObjects:count:]`。

在Literal之前，访问数组中的对象需要使用一个类似这样的方法`objectAtIndex`：

```objc
id obj = [array objectAtIndex:0]; 
```

而来到Literal的世界中，可以使用一对方括弧`[]`加对象对应的索引就可以访问到了：

```objc
id obj = array[0]; 
```

通过上面的语法，我们可以按照下面的方法来修改可变数组中的值：

```objc
NSMutableArray *mutableArray = [NSMutableArray arrayWithObject:@[@11,@76]];
mutableArray[0] = @51;
```

###3. NSDictionary###

在Literal引入之前，NSDictionary对象的实例化跟NSArray类似，看起来也很长，并且在最后需要`nil`，如下代码所示：

```objc
NSDictionary *dict = [NSDictionary dictionaryWithObjects:[NSArray arrayWithObjects: [NSNumber numberWithInt:10],
                                                                                    [NSNumber numberWithInt:20],
                                                                                    [NSNumber numberWithInt:30],
                                                                                    nil]
                                                 forKeys:[NSArray arrayWithObjects: @"first",
                                                                                    @"second",
                                                                                    @"third",
                                                                                    nil]];
```

上面的代码看起来着实有点过头了。如果要用Literal的话，就简洁明了多了：

```objc
NSDictionary *dicts = @{@"first":@10, @"second":@20, @"third":@30};
```

在上面NSDictionary实例化过程中，通过Literal，除了可以定义NSNumber和NSArray之外，还可以以可读的方式一一放置key和对应的值。相信这种方法大家都会喜欢。

从上面的介绍，你应该会喜欢上Literal，它确实可以让我们的代码更加容易读懂，并且不容易出错！！！

另外，我们还可以通过下面这样的方式访问字典中key对应的内容：(感谢[@谌启亮](http://weibo.com/u/2135198615)在评论中的提醒)

```objc
NSString *firstValue = dicts[@"first"]
```



// 注意：下面这一点内容我摘自[Objective-C-Literals-Boxed Expressions](http://clang.llvm.org/docs/ObjectiveCLiterals.html)

###4. Expressions###

Objective-C提供了一种新的语法对C表达式进行包装：`@( <expression> )`

它支持标量表达式(numeric, enumerated, BOOL)，以及C字符串指针类型：

```objc
// numbers.
NSNumber *smallestInt = @(-INT_MAX - 1);  // [NSNumber numberWithInt:(-INT_MAX - 1)]
NSNumber *piOverTwo = @(M_PI / 2);        // [NSNumber numberWithDouble:(M_PI / 2)]

// enumerated types.
typedef enum { Red, Green, Blue } Color;
NSNumber *favoriteColor = @(Green);       // [NSNumber numberWithInt:((int)Green)]

// strings.
NSString *path = @(getenv("PATH"));       // [NSString stringWithUTF8String:(getenv("PATH"))]
NSArray *pathComponents = [path componentsSeparatedByString:@":"];
```

关于Literals的更多详细内容可以参考：[Objective-C-Literals](http://clang.llvm.org/docs/ObjectiveCLiterals.html)


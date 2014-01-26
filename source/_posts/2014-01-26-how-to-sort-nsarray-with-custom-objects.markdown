---
layout: post
title: "对NSArray中自定义的对象进行排序"
date: 2014-01-26 16:00
comments: true
categories: iOS探索
published: true

---

![](/images/2014/01/20.png)

<!--more-->

本文译自[How to sort NSArray with custom objects](http://ios-blog.co.uk/tutorials/how-to-sort-nsarray-with-custom-objects/)。

我们开发的每个程序都会使用到一些数据，而这些数据一般被封装在一个自定义的类中。例如一个音乐程序可能会有一个Song类，聊天程序则又一个Friend类，点菜程序会有一个Recipe类等。有时候我们希望在程序中显示的列表数据是按照一定顺序进行排列的，本文我们就来看看在iOS中有哪些方法可以对NSArray中的对象进行排序。下面是目录：

* 小引
* 使用NSComparator进行排序
* 使用NSDescriptor进行排序
* 使用selector进行排序


###小引###

我们将要排序的对象是一个Persion类，如下定义：

```objc
@interface Person : NSObject
 
@property (nonatomic, copy) NSString *name;
@property (nonatomic, copy) NSString *surname;
@property (nonatomic, strong) NSDate *dateOfBirth;
 
@end
```

而数组中包含如下内容：

```objc
Smith John 03/01/1984
Andersen Jane 16/03/1979
Clark Anne 13/09/1995
Smith David 19/07/1981
Johnson Rose 22/02/1989
```

###使用NSComparator进行排序###

comparator实际上是用一个block对象作比较操作。它的定义如下所示：

```objc
typedef NSComparisonResult (^NSComparator)(id obj1, id obj2);
```

上面的参数(obj1、obj2)就是我们将要做比较的对象。block返回的结果为NSComparisonResult类型来表示两个对象的顺序。

要对整个数组做排序，则需要使用NSArray的`sortArrayUsingComparator: `方法，如下代码所示：

```objc
NSArray *sortedArray = [self.persons sortedArrayUsingComparator:^NSComparisonResult(Person *p1, Person *p2){
 
    return [p1.surname compare:p2.surname];
 
}];
```

最终排序的结果如下所示：

```objc
Andersen Jane
Clark Anne
Johnson Rose
Smith John
Smith David
```

###使用NSDescriptor进行排序###

Sort descriptor不仅可以用来对数组进行排序，还能指定element在table view中的排序，以及Core Data中对fetch request返回的数据做排序处理。通过sort descriptor可以很方便的对数组进行多个key的排序。下面来看看如何对我们的数组做surname排序，然后在进行name排序：

```objc
NSSortDescriptor *firstDescriptor = [[NSSortDescriptor alloc] initWithKey:@"surname" ascending:YES];
NSSortDescriptor *secondDescriptor = [[NSSortDescriptor alloc] initWithKey:@"name" ascending:YES];
 
NSArray *sortDescriptors = [NSArray arrayWithObjects:firstDescriptor, secondDescriptor, nil];
 
NSArray *sortedArray = [self.persons sortedArrayUsingDescriptors:sortDescriptors];
```

上面代码的排序结果如下所示：

```objc
Andersen Jane
Clark Anne
Johnson Rose
Smith David
Smith John
```

###使用selector进行排序###

当面，我们也可以定义自己的方法进行两个对象做比较，并将该方法用于数组排序。comparator消息会被发送到数值中的每个对象中，并携带数组中另外的一个对象当做参数。自定义的的方法的返回结果是这样的：如果本身对象小于参数中的对象，就返回`NSOrederedAscending`，相反，则返回`NSOrderedDescending`，如果相等，那么返回`NSOrderedSame`。如下代码所示：

```objc
- (NSComparisonResult)compare:(Person *)otherPerson {
 
    return [self.dateOfBirth compare:otherPerson.dateOfBirth];
}
```

这个方法定义在Person类中，用来对person的生日进行排序。

上面所介绍的这些方法都是为了完成相同的事情：对数组做排序处理，你可能在想改选择使用哪个呢？当需要通过多个key进行排序，那么最简单的方法就是使用sort descriptor。如果比较方法很复杂的话，建议在使用外面自己的selector。Block是再iOS 4之后引入的一个强大功能，用block作比较，可以不必使用任何的变量就能完成一个简单的比较方法，当然，你也可以定义一个复杂的block，来替换selector。

最后，其实这里并没有标准答案，你可以跟着自己的感觉走:]
---
layout: post
title: "iOS学习笔记3"
date: 2015-01-06 20:35:21 +0800
comments: true
description: "iOS学习笔记3"
keywords: ios,security
categories: 
---

最近又重新看了下斯坦福公开课的课程，也看了下effective objective-c这本书，还杂七杂八看了一些东西，课程和书还都没有看完，但已觉得学到了不少新的知识点，或者是以前没有意识到的知识点。这里记录一下。

<!--more-->

###语法糖

Objective-c中引入了语法糖，让NSArray，NSDictionary等Foundation里集合的语法显得简捷，但需要主要的是，语法糖也有一些需要注意的地方。

+ 创建的语法糖只对mutable对象有，对immutable没有
+ 不能增加nil的对象，会抛出exception,而使用非语法糖时，则会截断至nil
+ subclass不能用，不过类簇也不推荐subclass，这个我在[此文](http://straybirdzls.github.io/blog/2014/11/16/xianche/)中也提到过

###Event响应机制

这一部分基本算是翻译自苹果的[官方文档](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009541-CH1-SW1)。

ios中的事件分为3种，1.触摸事件 2.传感器事件 3.远程控制事件。当事件发生时，UIKit会创建对应的event并将其放入app的event队列中。事件在被响应需要经过层层传递，首先UIApplication会从event队列中选出top event进行分发，通常会发给UIWindow，最终传递给一个初始对象进行处理。
对于触摸事件，初始对象被称为hit-test view，而找到其的过程称为hit-testing。hit-testing通过hitTest:withEvent:返回touch发生的view，其内部通过递归地调用pointInside:withEvent:来得到hit-test view.

而对于传感器事件和远程控制事件，则交给first-responder对象处理。

responder对象是是指可以响应event的对象。UIResponder是所有responder对象的基类， UIApplication，UIView，UIViewController等都是responder对象。first－responder是被指定为最先响应event的对象，可以通过`canBecomeFirstResponder`和`becomeFirstResponder message`将object设置为first－responder。当first－responder无法响应event时，其会传递给next responder直到UIWindow，UIApplication。

对于Touch event，hit－test view即为first－responder，而其响应链的响应顺序如下：    
1. viewj将event传递其view controller中view hierarchy的superview，直到topmost view     
2. topmost view将event传递给其view controller    
3. view controller将event传递给其topmost view的superview    
1-3循环，直至根view controller，最终至UIWindow直至UIApplication。    

###消息机制

在iOS中，我们知道消息是通过runtime在实现表中进行查找来实现响应的，除了通过正常的方式创建方法外，runtime机制还会通过几下步骤进行消息传递：

1. 动态方法解析，`ResolveInstanceMethod/ResolveClassMethod`,这是最后将方法加入IMP表中的机会，其中可以通过`class_addMethond`等加入对应的IMP，假如完之后会再进行一次消息的派送。因为这个的真正实现时将方法加入IMP表中，所以isResponseToSelector中是会查找到这其中的实现的。@dynamic是使用动态方法解析的一个很典型的例子，其告诉编译器property的setter和getter会在运行时提供。
2. 快速消息转发：可以通过`ForwardingTargetForSelector`来实现，只能将消息转发给一个对象
3. 标准转发，通过实现`-(NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector`和`-(void)forwardInvocation:(NSInvocation *)anInvocation`这两个方法进行转发。NSMethodSignature是方法签名，为什么要有这个呢？那是因为selector其实只是一个字符串，从它并不能知道参数的类型和返回值的类型，而方法签名实际上是用来描述参数的类型和返回值的类型的。也就是说，相同的返回值与参数的所有selector的签名其实是一致的。而Objc运行时要根据对象返回的这个签名来抓取参数，然后才会调用`- (void)forwardInvocation:(NSInvocation *)anInvocation`这个方法。

不得不说runtime真的好强大。

###KVO实现

KVO用到了isa swizzling,以下是从别人文章里引用的，可惜很抱歉，我忘记具体时哪篇文章。

> 当某个类的对象第一次被观察时，系统就会在运行期动态地创建该类的一个派生类，在这个派生类中重写基类中任何被观察属性的 setter 方法。    
> 派生类在被重写的 setter 方法实现真正的通知机制，就如手动实现键值观察那样。这么做是基于设置属性会调用 setter 方法，而通过重写就获得了 KVO 需要的通知机制。当然前提是要通过遵循 KVO 的>属性设置方式来变更属性值，如果仅是直接修改属性对应的成员变量，是无法实现 KVO 的。     
> 同时派生类还重写了 class 方法以“欺骗”外部调用者它就是起初的那个类。然后系统将这个对象的 isa 指针指向这个新诞生的派生类，因此这个对象就成为该派生类的对象了，因而在该对象上对 setter 的调用就会调用重写的 setter，从而激活键值通知机制。此外，派生类还重写了 dealloc 方法来释放资源。

###安全相关

+ class-dump的工作原理:因为 obj-c的动态特性导致 obj-c的二进制代码中会保留类名和方法名，所以可以用 otool 得到这些信息，而class-dump所做的就是把otool -ov得到的信息组织成结构更清晰的信息输出
+ 通过在xcode中设置`-Wl,-sectcreate,__RESTRICT,__restrict,/dev/null`可以给输出的二进制文件中增加`__RESTRICT/__restrict section`来防止dylib注入，更多的信息，可以参见[这篇文章](http://bbs.iosre.com/forum.php?mod=viewthread&tid=432)

###一些容易犯的错误

+ NSDictionary/NSArray这种里面只能存储对象，不能存储普通的变量，这个在写代码时很容易忘记
+ NSNotification Center 会通过_unsafe_unretain指向observer，所以不再需要的时候一定要记得remove，否则会带来意料不到的crash

这一篇比较杂七杂八，因为本身就是平时学习笔记的一些归纳总结，方便自己温习查看。

##参考文献

[苹果官网EventHandling](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009541-CH1-SW1)    
[防止tweak依附，App有高招；破解App保护，tweak留一手](http://bbs.iosre.com/forum.php?mod=viewthread&tid=432)
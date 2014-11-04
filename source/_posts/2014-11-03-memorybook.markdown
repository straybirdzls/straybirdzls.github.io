---
layout: post
title: "读书笔记(1):<iOS与OSX多线程和内存管理>"
date: 2014-11-04 21:25:41 +0800
comments: true
categories: 
description: "读书笔记(1)：iOS与OSX多线程和内存管理" 
keywords: objective-c,多线程,内存管理,ARC
---
最近这些天看了<iOS与OSX多线程和内存管理>这边书，觉得这本书还是写得很不错的，这一篇对内存管理中的一些点记一些笔记，主要集中在ARC上，关于block的以后再谈。我这里也不准备从最基本的概念入手，因为介绍ARC的文章已经太多了，所以我只记录一些我觉得比较特殊的点。

####查看自动释放池的状态

只需要使用先声明:

	extern void _objc_autoreleasePoolPrint();

就可以使用

	_objc_autoreleasePoolPrint();
	
打印释放池里的状态了。这个API是私有API,对于osx和iOS通用，还有一个API只对iOS可以使用，这里就不提了，个人觉得上面这个API就够用了。
<!--more--> 

####ARC下查看retain count

在启用了ARC的项目中，因为`RetainCount`已经不能使用了，但有时候我们也想观察一下当前对象到底有多少引用指向，这时候我们就可以使用`_objc_rootRetainCount(obj)`来查看对象的引用计数。

####使用__weak对象时是否会retain一个对象并放入自动释放池?

在书中，提到当使用`__weak`对象时，会`retain`一个`autorealeasing`对象，这么做的原因是可以保证在`@autoreleasepool`块结束之前对象可以保证正常访问。读到这里的时候，我非常困惑，因为实在很难想像有什么这样的使用场景。当没有对象强引用时，`__weak`对象将会自动指向nil，为什么还要将其放入自动释放池呢? 我百思不得其解。然后在实验书中代码时，我得到了不一样的结果。经过自己实验，我发现应该是苹果改了实现，也许是他们也发现将`__weak`对象`retain`一个`autorealeasing`对象是没有意义的。测试代码如下。

	id  __strong obj = [[NSObject alloc]init];
	_objc_autoreleasePoolPrint();
	id __weak o = obj;
	NSLog(@"count: %d",_objc_rootRetainCount(obj));
	NSLog(@"class=%@",[o class]);
	NSLog(@"count: %d",_objc_rootRetainCount(obj));
	_objc_autoreleasePoolPrint();

在xcode4.6中运行的结果如下，使用__weak，会有相应的对象放入自动释放池。

	objc[20987]: ##############
	objc[20987]: AUTORELEASE POOLS for thread 0x7fff761d3310
	objc[20987]: 1 releases pending.
	objc[20987]: [0x100803000]  ................  PAGE  (hot) (cold)
	objc[20987]: [0x100803038]  ################  POOL 0x100803038
	objc[20987]: ##############
	2014-11-04 15:26:15.094 memory-xcode46[20987:303] count: 1
	2014-11-04 15:26:15.094 memory-xcode46[20987:303] class=NSObject
	2014-11-04 15:26:15.095 memory-xcode46[20987:303] count: 2
	objc[20987]: ##############
	objc[20987]: AUTORELEASE POOLS for thread 0x7fff761d3310
	objc[20987]: 2 releases pending.
	objc[20987]: [0x100803000]  ................  PAGE  (hot) (cold)
	objc[20987]: [0x100803038]  ################  POOL 0x100803038
	objc[20987]: [0x100803040]       0x100500f10  NSObject
	objc[20987]: ##############
	
在xcode5/xcode6中的运行结果如下，使用__weak，并不会有相应的对象放入自动释放池。

	objc[20761]: ##############
	objc[20761]: AUTORELEASE POOLS for thread 0x7fff761d3310
	objc[20761]: 1 releases pending.
	objc[20761]: [0x100803000]  ................  PAGE  (hot) (cold)
	objc[20761]: [0x100803038]  ################  POOL 0x100803038
	objc[20761]: ##############
	2014-11-04 15:20:51.001 memeroy_test_xcode5[20761:303] count: 1
	2014-11-04 15:20:51.003 memeroy_test_xcode5[20761:303] class=NSObject
	2014-11-04 15:20:51.003 memeroy_test_xcode5[20761:303] count: 1
	objc[20761]: ##############
	objc[20761]: AUTORELEASE POOLS for thread 0x7fff761d3310
	objc[20761]: 1 releases pending.
	objc[20761]: [0x100803000]  ................  PAGE  (hot) (cold)
	objc[20761]: [0x100803038]  ################  POOL 0x100803038
	objc[20761]: ##############

由此可见，实现确实是不一样了，我尝试找apple的源码，在老版本的[实现](http://www.opensource.apple.com/source/objc4/objc4-493.11/runtime/objc-arr.mm)中(objc-arr.mm文件),可以看到: 使用`__weak`对象确实有相应的对象放入了自动释放池。

	id objc_loadWeak(id *location)
	{
    	return objc_autorelease(objc_loadWeakRetained(location));
	}
	
	objc_loadWeakRetained(id *location)
	{
		......
	}

遗憾的时候，在最新[源码](http://www.opensource.apple.com/source/objc4/objc4-646/runtime)中，已找不到objc-arc.mm文件，我尝试在其他可能的文件中找objc_loadWeak，可惜没找到，不知道是换到别处去了，还是苹果决定不再公开此文件。于是我尝试了一下在新的版本上查看汇编代码，可以看到直接调用了`_objc_loadWeakRetained`,这也侧面证明了现在使用`__weak`对象并没有相应的对象放入释放池。

	main.m:50:0
	movl	%ecx, -108(%rbp)        	callq	_objc_storeStrong
	leaq	-32(%rbp), %rdi
	.loc	1 52 0                 
	callq	_objc_loadWeakRetained
	movq	%rax, %rsi

这本书出来的时间也比较久了，所以可能有些内容跟最新的实现有出入，但通过书中的引子来思考并且找到最终答案让我受益颇多。

#### 默认一定是__strong?

ARC声明对象时，默认是`__strong`的，也即 `id test;`等同于`id __strong test;`。但是一定要注意`id *test;`是等同于`id __autoreleasing *test;`。即指向对象的指针默认是`__autoreleasing`的。 分配指向对象的时针时，必须得保证两边得所有权限制符是一致的，像如下的代码，编译器会报错。

	NSError *error = nil; 
	NSError **pError = &error;
	
改成如下就没问题啦。
	
	NSError *error = nil;	NSError * __strong *pError = &error;

最后总结一下，这本书关于所有权的阐述蛮有意思的，然后对苹果的实现机制也挖掘的比较深入，推荐想了解内存机制的人读一下。
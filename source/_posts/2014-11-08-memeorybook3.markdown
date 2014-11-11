---
layout: post
title: "读书笔记(3) iOS与OSX多线程和内存管理"
date: 2014-11-08 20:30:54 +0800
comments: true
description: "读书笔记(3)：iOS与OSX多线程和内存管理" 
keywords: objective-c,GCD,block,内存管理,多线程,ARC
categories: 
published: true
---
今天是<iOS与OSX多线程和内存管理>读书笔记的最后一篇了，主题是多线程。书中主要是讲GCD,并没有讲NSOperation的东西，我这一篇一方面简要地对GCD记录一些知识点，也谈一点点NSOperation方面的东西。对于NSOperation方面的知识，我看了很多人推荐的[教程](http://www.raywenderlich.com/19788/how-to-use-nsoperations-and-nsoperationqueues)，觉得写得非常好，推荐想学习的人看一看。

####慎用@sync

从字面意思`@async`是异步执行，那`@sync`就是同步执行了。使用`@sync`时一定要万分小心，比如主线程，则会直接造成阻塞，死锁。因为任务执行完之后才会返回，但任务又必须等待其之前的任务完成才能执行。

<!--more-->

####UI刷新

移动端非常注重用户体验，所以UI刷新一定要放在主线程。若使用GCD，可以用`dispatch_get_main_queue()`来获取主线程，若使用NSOperation,可以用`[NSOperationQueue mainQueue]`来获取，当然也可以通过delegate使用`performSelectorOnMainThread`来实现。

####一些好用的关键字

在GCD中，有很多很好用的功能，比如`dispatch_once`可以保证代码只被调用一次;`dispatch_group_async`可以定义一组任务，当一组任务全部完成时，会通过`dispatch_group_notify`来给通知;`dispatch_barrier_async`用于等待其他任务的完成，并完成任务之后，才能启动之后的任务，对于数据库访问以及文件的读写都非常有用;`dispatch_apply`是跟`@sync`和`group`都有关系的，它可以定义一组任务，并等待其完成;`dispatch_semaphore`提供类似于`dispatch_barrier_async`的功能，但是可以保证更小的粒度，它是通过信号量的方式来工作的。

####NSOperationQueue和GCD

自GCD出来后，NSOperationQueue就是用GCD来实现了。通过对上面推荐的教程的学习，NSOperation对于代码管理是非常方便的，而且可以cancel，对于依赖的管理也非常方便。根据苹果使用high level实现的惯例，一般情况下都是推荐使用NSOperation。当然GCD很轻量级，使用起来非常方便，所以根据具体的使用场景，也有很多需要的时候。

其实书中还讲了很多其他方面的内容，GCD确实很强大，有了它，我们不用自己管理线程，因为这方面我还处于学习阶段，所以暂时还没能有一些比较深刻的见解，有兴趣的可以看objc的这个[专题](http://objcio.cn/issue-2/)，讲得非常精彩。截止到现在，这本书<iOS与OSX多线程和内存管理>我也算是粗略地过了一遍，但实践出真知，我现在还处于学习阶段，接下来要多多练习，方能将这些全部变成自己的知识。
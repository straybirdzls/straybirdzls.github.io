---
layout: post
title: "读书笔记(3) iOS与OSX多线程和内存管理"
date: 2014-11-08 20:30:54 +0800
comments: true
description: "读书笔记(3)：iOS与OSX多线程和内存管理" 
keywords: objective-c,GCD,block,内存管理,多线程,ARC
categories: 
published: false
---
今天是<iOS与OSX多线程和内存管理>读书笔记的最后一篇了，主题是多线程。书中主要是讲GCD,并没有讲NSOperation的东西，我这一篇一方面简要地对GCD记录一些知识点，也谈一些NSOperation方面的东西。

####慎用@sync

从字面意思`@async`是异步执行，那`@sync`就是同步执行了。使用`@sync`时一定要万分小心，比如主线程，则会直接造成阻塞，死锁。因为任务执行完之后才会返回，但任务又必须等待其之前的任务完成才能执行。


####


---
layout: post
title: "Javascript学习2"
date: 2015-03-29 22:32:30 +0800
comments: true
categories: 
published: false
---

这篇接上篇，记录Javascript中的面向对象。ECMAScript中把对象定义为：无序属性的集合，其属性可以包含基本值，对象或者函数。其没有类的概念，每个对象都基于一种引用类型创建的，这种引用类型可以是原生的，也可以是自定义的。所以在javascript中其对象跟一般oo语言中的区别还是蛮大的。

<!-- more -->

ECMAScript中有2种属性，数据属性和访问器属性。数据属性包含一个数据值的位置，在这个位置可以读取和写入值。数据属性有4个描述其行为的特性:[[Configurable]],[[Enumerable]],[[Writable]],[[Value]]。要修改属性默认的特性，必须使用Object.defineProperty()方法。访问器属性不包含函数值：他们包含一对getter和setter函数（不是必需的）。使用Object.getOwnPropertyDescriptor()可以取得给定属性的描述符。

使用new Object()或者对象字面

工厂模式

构造函数模式

原型模式
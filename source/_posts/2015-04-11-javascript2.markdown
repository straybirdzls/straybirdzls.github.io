---
layout: post
title: "Javascript学习2"
date: 2015-04-12 14:54:51 +0800
comments: true
categories: 
---

这篇接着上篇，不过有些迟到了。随着Facebook React Native的开源，现在Javascript颇有点“王道”的意思了，一定要将学习坚持下去！这篇主要说说Javascript中的面向对象。

<!-- more -->

###对象
ECMAScript中把对象定义为：无序属性的集合，其属性可以包含基本值，对象或者函数。其没有类的概念，每个对象都基于一种引用类型创建的，这种引用类型可以是原生的，也可以是自定义的。所以在javascript中其对象跟一般oo语言中的区别还是蛮大的。

ECMAScript中给对象定义了2种（内部）属性，数据属性和访问器属性。数据属性包含一个数据值的位置，在这个位置可以读取和写入值。数据属性有4个描述其行为的特性:[[Configurable]],[[Enumerable]],[[Writable]],[[Value]]。要修改属性默认的特性，必须使用Object.defineProperty()方法。访问器属性不包含函数值：他们包含一对getter和setter函数（不是必需的）。使用Object.getOwnPropertyDescriptor()可以取得给定属性的描述符。

###创建对象

创建对象的最简单的方式就是使用new操作符或者字面量法，其如下：

	//new操作符
	var person = new Object();
	person.name = "hao";
	//字面量法
	var person = {name:"hao"};
	
这很明显的一个缺点就是会有大量的重复代码，为了解决这个问题，就有很多种设计模式，在javascript里用的最多的就是工厂模式，构造函数模式，原型模式以及组合使用构造函数与原型模式了，当然也还有其他很多的模式，不过它们基本都是对构造函数模式或者原型模式的变种。这里我只谈下最基本的。

####工厂模式
工厂模式抽象了创建具体对象的过程，考虑到在ECMAScript中无法创建类，开发人员就发明了一种函数来封装以特定接口创建对象的细节。

	function createPerson(name,age)
	{
		var o = new Object();
		o.name = name;
		o.age = age;
		o.sayName = function() {
			console.log(this.name);
		};
		return o;
	}
	
	var person = createPerson("hao",26);
		
工厂模式解决了创建相似对象时代码重复的问题，但没有解决对象识别的问题。

####构造函数模式
在ECMAScript中，任何函数都可以当作构造函数，只要使用new操作符。但按照管理，构造函数应该使用大写字母开头，而非构造函数则用小写字母开头，以做区分。

	function Person(name,age)
	{
		this.name = name;
		this.age = age;
		this.sayName = function() {
			console.log(this.name);
		}
	}
	
	var person = new Person("hao",26);
	
使用new操作符时，会经历4个步骤：1）创建1个新对象 2）将构造函数的作用域赋给新对象3）执行构造函数中的代码 4）返回新对象。使用构造函数模式时，生成的对象都会有一个constructor属性，上例中，该属性指向Person。但是检测对象类型，还是使用instanceof操作符会更好。

	console.log(person instanceof Object);
	console.log(person instanceof Person);
	
使用构造函数模式有一个问题就是函数对象不是共享的，每一个实力都会产生自己的函数对象，但实际上它们是可以共享的，为了解决这个问题，可以将函数定义转移到构造函数外部，如下

	this.sayName = sayName;	
	function sayName(){
		console.log(this.name);
	}
	
这样带来的问题就是会有很多的全局函数，既破坏了封装性，也让全局作用域名不副实-全局作用域中定义的函数实际上只能被某个对象调用。

####原型模式
在1中，我们提到过每个函数都有一个prototype属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和当法。

	function Person(){
	}
	Person.prototype.name = "hao";
	Person.prototype.age = 26;
	Person.prototype.sayName = function() {
		console.log(this.name);
	};
	
	var person = new Person();

原型模式创建出来的对象，属性和方法都是共享的，即原型对象的。默认情况下，原型对象都会自动获得一个constructor属性，这个属性包含一个指向prototpye属性所在函数的指针。当创建一个新对象时，该对象内部包含一个指针(内部属性)，指向构造函数的原型对象。虽然无法直接访问[[Prototype]]，但可以通过isPrototypeOf()方法来确定对象间是否存在这种关系。ECMAScript5中也新增了一个方法Object.getPrototypeOf().

	console.log(Person.prototype.isPrototypeOf(person)); //ture
	console.log(Object.getPrototypeOf(person) == Person.prototype); //true

当访问对象属性时，会首先搜索实例对象中的，没搜索到则搜索其原型对象中的。值得注意的是可以通过对象访问原型中的值，但不能重写。如果往实例中增加与原型中相同的属性，则会屏蔽原型中的。可以通过hasOwnProperty来检测属性是否来自于实例。而in操作符则不论属性属于实例还是原型中时都会返回true。

	var person1 = new Person();
	var person2 = new Person();
	person1.name = "test";
	console.log(person1.hasOwnProperty("name"));  //true
	console.log("name" in person1);               //true
	console.log(person2.hasOwnProperty("name"));  //false
	console.log("name" in person2);               //true
 
可以给原型对象直接赋值，但是会破坏constructor属性(不会影响instanceof操作符)。所以如果需要直接赋值时，最好自己将constructor属性补上(下面有示例)。

原型模式可以弥补构造函数模式不方便共享实例函数的不足，但是对于属性而言，则共享就不是很合理了。对于那些包含基本值的属性而言，可以用同名属性覆盖，但对于引用类型的属性而言，问题就比较突出了。

	function Person() {
	}
	Person.prototype = {
		constructor : Person,
		friends : ["hao"]
	};

	var person1 = new Person();
	var person2 = new Person();
	person1.friends.push("test");
	console.log(person1.friends);     //[ 'hao', 'test' ]
	console.log(person2.friends);     //[ 'hao', 'test' ]
	
####组合使用构造函数模式与原型模式
通过上面的构造函数模式和原型模式的分析，我们知道，对于属性而言，我们一般不需要共享，最好使用构造函数模式，而对于函数而言，一般需要共享，最好使用原型模式，所以就有了组合模式，现在也是被用的最多的一种模式，这里也就不多做说明了。

本想这次也记录一下继承的，不过发现内容有点多，这个就放到下回了。

###参考文献
JavaScript高级程序设计（第3版）
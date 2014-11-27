---
layout: post
title: "ios学习笔记2"
date: 2014-11-26 22:12:09 +0800
comments: true
description: "ios学习笔记2"
keywords: ios,Core Foubdation,views,id,instancetype,file's owner,Core Graphics
categories: 
---

这一篇接上篇，依然是iOS Programming: The Big Nerd Ranch Guides这本书的学习笔记，这本书对于各种点讲的还比较清楚，这一篇主要集中在view controller相关的点上。

<!--more-->

####View Controller之间的关系

刚开始的时候，容易被View Controller之间的关系搞迷糊，其实归根到底关系就2种，family关系和presented/presenting关系。  
family关系，显而易见，就是包含关系了，一个view controller包含几个子view controller，可以通过UIViewController的viewControllers来获取。子view controller可以通过parentViewController/navigationController/tabBarController/splitViewController来获取父controller，当然后3个都是会顺着view controller的关系往上查找相应的view controller类型，若没有，则返回nil。  
presented/presenting关系，描述的是一种模态controller，presented的controller将会之于当前view的top之上，可以通过presentedViewController/presentingViewController来访问。值得注意的是1).family中有一个view controll而有presentedViewController，则其他的也跟着拥有; 2).默认的presentingViewController会去controller family里寻找最父级的controller，如果不想拥有这种行为，需要将view controller的definesPresentationContext属性设为YES,并将modalPresentationStyle设置为UIModalPresentationCurrentContext。

####UITableViewController

UITableViewController是实际中非常常用的view conroller，其自带了一个tableView，然后还拥有delegate和datasource，datasource主要是用来管理数据源，其只用来数据源的获取和刷新，真正跟tableview交互的是delegate。UITableViewController自带的API非常丰富，从我的学习中，有需求的时候就多看看API的说明，总能找到一款满足需求的API。

本来还想再多写一点，结果发现很多东西不是三言两语可以说得清楚，只需要自己动手跟着书中的例子走一遍，然后再实践几次，慢慢地熟悉了，就都了然于心了。

###参考文献
书籍: iOS Programming: The Big Nerd Ranch Guides
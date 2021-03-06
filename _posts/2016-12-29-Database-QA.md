---
layout: post
title: "数据库系统原理期末复习 Q&A"
excerpt: "这篇博文主要记录：在复习过程中，自己或别人碰到的一些小问题。"
tags:
  - Database
categories: 做笔记
---

这篇博文主要记录：在复习过程中，自己或别人碰到的一些小问题。

------

Q1：是不是所有的关系都有候选码？

A：不是。关系有三种类型：基本关系（也称基本表／基表）、查询表、视图表。其中，查询表就是查询结果对应的表。我们知道，查询是在原有表上的筛选，如果不加DISTINCT条件，就可能会出现多个元组完全一样的情况，而不能使用属性（组）来唯一标识一个元组，不存在候选码。即：不是所有的关系都有候选码。

ps.基本关系一定有候选码。

 

Q2：候选码、主码、主属性、码之间的对应关系？

A：候选码和主码都是属性或属性组的**集合**。当一个关系有多个候选码的时候，选择其中一个为主码。

而主属性即是各个候选码的所有**属性**，不包含在**任何候选码**中的属性称为非主属性。

码=主码或候选码。

例如，有一张学生信息表，{学号}和{身份证号}是候选码。因此，‘学号’、‘身份证号’为这个关系的主属性，同时是这个关系的码。可以选择其中一个作为这个表的主码，而其他所有属性均为非主属性。

 

Q3：选课表的主码是{学号,课程号}，但是这两个属性又是选课表的外码。由定义知，外码是指非主属性引用其他关系表或自身关系的码。那么，‘学号’和‘课程号’是非主属性还是主属性？

A：主属性。

首先我们看**外码**的定义：设F是基本关系R的一个或一组属性，但不是关系R的码，Ks是基本关系S的主码。如果F与Ks相对应，则称F是R的外码。

回归到这个问题：{学号}和{课程号}虽然都是外码，但它们参照的并不是同一张表。‘学号’参照学生表，‘课程号’参照课程表。根据定义，{学号}是选课表的外码，{课程号}也是选课表的外码，因此它们各自都不是选课表的主码。

但是，因为{学号,课程号}是选课表的主码，是包含在候选码内的属性。根据定义，只有不包含在任何候选码中的属性才是非主属性。因此，**选课表的主属性是‘学号’、‘课程号’，非主属性是‘成绩’**。

这个问题看似有些矛盾，但是本质上是可以讲通的。

 

Q4：如果被参照关系中的元组被删除, 应该 采用什么样的策略?

A：采用**级联删除**策略或**不予处理。**

级联删除：将被参照关系的数据一并删除（可能导致数据不一致，慎用！）

 

Q5：为什么oracle 12c/MS SQL等数据库管理系统，在使用DROP TABLE后还保留了视图？

![img](https://github.com/HusterHope/blogimage/raw/master/DB.png)

A：<http://docs.oracle.com/database/121/SQLRF/statements_9003.htm#SQLRF01806>

查看oracle官方文档，我们注意到：Drop Table命令只是把表放到了回收站，视图的定义还保留在数据字典里。这时，除非重建以前的表，或者重建基于其他表的视图，否则视图实际上已经不可用了。

其他DBMS此处不作说明，原理在各自的官方文档处应该比较完善。

 

Q6：如何通过一个关系的函数依赖确定它的候选码，并判断它属于第几范式？

例如：有关系模式R(A,B,C,D,E)和函数依赖A→B,BC→D,****DE→A，列出R的候选码，并确定关系R属于第几范式？

A：我们知道，要判断一个或一组属性（记作X）是否为一个关系的候选码，只需要求出属性集X关于函数依赖集F的闭包，再判断该闭包是否为全集即可。

但是，如果通过已知函数依赖集来确定候选码，就会稍有些麻烦，且课本上也没有给出理论性的判断方法。查阅了一些资料，下面这个链接的博文给出了一种比较简单的方法，可以参考：

<http://stinaq.me/blog/2012/11/21/an-easy-way-to-finding-the-candidate-key-of-a-database-relation/>

按照这篇博文的意思，我们需要先将存在的所有函数依赖集，用盒子(boxes)和箭头(arrows)画出来，则题中依赖集绘制如下：

![img](https://github.com/HusterHope/blogimage/raw/master/QA2.jpg)

接下来开始判断候选码。

从A出发：A+={A,B}!=U，因此再加入C（A+已经含有B，所以B不用加入候选码的集合中），{ABC}+=ABCD!=U，这时再加入E即为U，因此{ACE}是一个候选码。

从B出发：B+=B!=U，因此再加入C，{BC}+=BCD!=U，再加入E，我们发现{BCE}+=ABCDE,已经是全集，于是{BCE}也是一个候选码。

从C出发：C+=C!=U，因此再加入D，{CD}+=CD!=U，因此再加入E，我们发现{CDE}+已经是全集，于是{CDE}是候选码。

从D出发：D+=D!=U，{DE}+=ABCD!=U，于是得到候选码{CDE}，与从C出发结果相同。同理，从E出发我们会得到候选码{ACE}，与从A出发相同。

综上，关系R的候选码是{ACE}、{BCE}和{CDE}。

接下来判断R属于第几范式，这一步就相对容易了。

通过候选码的判断我们知道，A、B、C、D、E均是主属性，因此R至少为3范式。

那么R是否为BCNF呢？我们只需要观察所有的函数依赖左侧是否为码即可。这里显然不是。

因此我们得出结论：R为BC范式。

 

------

更新记录：12.26  V1.0 新增 Q1、Q2、Q3

12.26  V1.1  补充和修改对‘外码’、‘码’的定义

12.26 V1.2 新增Q4、Q5

12.29 V1.3 新增Q6
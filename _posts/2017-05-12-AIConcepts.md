---
title: "ML, Data Science, AI, Deep Learning, and Statistics"
layout: post
excerpt: "整理贴：机器学习(Machine Learning)、数据科学(Data Science)、人工智能(Artificial Intelligence)、深度学习(Deep Learning)和统计学(Statistics)之间的区别及联系。"
last_modified_at: 2012-02-05T10:27:01-05:00
tags:
  - AI
  - ML
  - DL
categories: 做笔记
---

整理贴：机器学习(Machine Learning)、数据科学(Data Science)、人工智能(Artificial Intelligence)、深度学习(Deep Learning)和统计学(Statistics)之间的区别及联系。

------

今天偶然看到一篇文章：

[一文读懂机器学习、数据科学、人工智能、深度学习和统计学之间的区别](http://www.bibdr.org/nd.jsp?id=86&_np=2_435)

原文如下，作者Vincent Granville。

[Difference between Machine Learning, Data Science, AI, Deep Learning, and Statistics](http://www.datasciencecentral.com/profiles/blogs/difference-between-machine-learning-data-science-ai-deep-learning)

毕竟自己也是信息大类的专业，这些概念不算特别陌生，但要说区别还是挺麻烦的。

看了作者讲的概念，以及杂七杂八的引用文章，有一些茅塞顿开之感，虽然目前也不算是定论，就当是把这些概念整理一下加深印象好了。虽然说中文翻译后的标题告诉我们「一文读懂」，翻译做的也不错，但还是比较标题党。这么广的范畴，一个例子都不举，一文读懂只是针对行内人士的吧。

闲话不讲，下面从大范围往小范围说，排序：数据科学>人工智能>机器学习(≈统计学)>深度学习，需要注意的是，它们之间并不是完全的包含与被包含关系，更多的是部分交叉，大小的比较只是在概念的细化层次上。

有关这些概念，在 [Polly Mitchell-Guthrie](http://blogs.sas.com/content/author/pollymitchellguthrie/) 的博客里给出了一张图可以参考：

![img](http://ohn6qfqhe.bkt.clouddn.com/relation.png)

即：

![img](http://ohn6qfqhe.bkt.clouddn.com/relation_ch.png)

------

这些概念中，我认为涵盖面最广的是数据科学(Data Science)。

先看数据科学的含义：

> Data science, also known as data-driven science, is an interdisciplinary field about scientific methods, processes and systems to extract knowledge or insights from data in various forms, either structured or unstructured, similar to Knowledge Discovery in Databases (KDD). (From [WikiPedia](https://en.wikipedia.org/wiki/Data_science))

译：数据科学(亦称数据驱动科学)是一个关于**科学方法、过程和系统**的跨学科领域，目的是从结构化或者非结构化的数据中提取知识或见解，类似从数据库里发现知识(KDD)。

> 数据科学采用了数学，统计学，信息科学和计算机科学等领域的许多领域的技术和理论，特别是从机器学习，分类，集群分析，数据挖掘，数据库和可视化等子领域。

机器学习是什么？1998年Tom Mitchell给出的定义如下：

> 一个计算机程序，给它一个任务T(Task)和一个性能测量方法P(Performance)。如果在经验E(Experience)的影响下，P对T的结果得到了改进，那么就说该程序从E中学习。

更多信息可以参考笔记[【Stanford-ML】机器学习的动机与应用](https://husterhope.github.io/2017/02/14/CS229-1.html)

维基百科提到，**数据科学运用了机器学习领域的技术和理论**。并且，前者比后者要广泛，Vincent Granville提到：“**数据科学中的数据可能并非来自机器或机器处理，例如调查数据可能就是手动收集。**”

> Data, in data science, may or may not come from a *machine* or mechanical process (survey data could be manually collected).

-> **数据科学领域大于且包含机器学习。**

------

下面说人工智能。

人工智能，听起来就很有逼格对吧？但是Vincent Granville讲了一个有微妙笑点的故事：

> 在我事业的早期阶段（大约1990年），我开发过图像远程感知技术，其中包括识别卫星图像的模式（形状和特征，比如湖泊）和执行图像分割：那段时间我的研究工作被称为是**计算统计学(computational statistic)**，但在我的母校，**隔壁的计算机科学系也在做着几乎完全一样的事情**，但他们把自己的工作叫做是**人工智能**。

记得我曾经在某个地方看过或者听过一段很形象的话来描述机器学习和人工智能的关系：「人工智能」是一个在计算机技术还没有像现在这样高度发达的时候就产生的术语，基本上而言是指**通过人工的方法来代替人类天生的自然智能**。后来，「机器学习」这个词出现了，相关领域学者认为AI(人工智能)这个词太老，干脆自成一派，都说自己是搞「机器学习」的，但本质上就是**人工智能的一个分支**。

如果说「人工智能」是包括了「机器学习」的，我想应该没什么人会反对吧。

**-> 人工智能大于且包含机器学习。**

人工智能和数据科学的关系就像是平面上两个相交但是不相互包含的圆。它们有交叉，但是不相重合的地方也很多。相对来说，数据科学的概念范围更广（？）

------

统计学，看上去跟计算机最不沾边的词，在文章开头的图里和机器学习毫无交集，却并不是和机器学习没有关联。

> 机器学习是计算机科学与人工智能的一个子领域。
>
> 而统计学是数学的一个子领域。

但是！它们有着共同的目标：**从数据中学习**。

> Both machine learning and statistics share the same goal:  **Learning**** from data**.

并且它们的很多概念是相关的，比如：回归(Regression)／监督学习(Supervised Learning)、数据点(Data point)／实例(Instance)等。更多比较可参考[Machine Learning Vs. Statistics](http://www.edvancer.in/machine-learning-vs-statistics/)

**-> 统计学和机器学习领域不同，但相似点很多。**

------

还剩最后一个概念，深度学习，这个概念就和机器学习联系很紧密了。

> 深度学习是如今非常流行的一种机器学习方法。它涉及到一种特殊类型的数学模型，可认为它是**特定类型的简单模块的结合**（函数结合），这些模块可被调整从而更好的预测最终输出。

深度学习通常和神经网络相关联。

**-> 机器学习概念完全包含深度学习。**

------

想用原作者文章下面来自[Suresh Babu](http://www.datasciencecentral.com/profile/SureshBabu515)的评论片段来作为结尾：

> Tools and what we do with the tools, in many different forms, have been the forces guiding human learning since the dawn.

在很多方面，工具以及我们使用工具所做的事情，自人类诞生以来就在引导着人类的学习。

------



来源见文章内引用和链接

于openGL实验课（逃

End.
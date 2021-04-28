---
title: "IEEE VR 2021-B(l)ending Realities Notes"
layout: post
categories: 做笔记
tags:
  - MR
  - VR
  - HCI
  - RDW
  - IEEEVR2021
---

演讲地址： [IEEE VR 2021 - 31 March - Stream A - YouTube](https://www.youtube.com/watch?v=Ds-h1J4MFMI) ，对应视频时间段 3:45:00 - 4:45:00

<!-- more -->

Frank Steinicke教授从VR发展过程和若干交叉学科出发，较为完整地介绍了混合现实环境下的视觉扭曲工作，也就是标题里巧妙的【B(l)ending Realities】。

这篇文章从这个演讲出发，对感兴趣的内容稍加整理。笔记总体遵循演讲流程，为了归纳方便，部分内容顺序有所调整。

## 发展历程和目标

首先介绍了一些历史背景，如1968年的第一个混合现实头盔，1973年第一部以虚拟现实为技术主题的电影《World on a Wire》，1993年第一届IEEE VR开幕，第一次提出Presence的概念(The feeling of being there)，1999年著名科幻电影《黑客帝国》，2005年为Frank自己第一次参加VR，2021年Microsoft Hololens的大规模商业化等。

提出一个opening question：

> 当技术没有足够完备时，我们能够用VR/AR技术做些什么？
>
> 当然可以做工程来提升技术，但这需要很长时间和大量资源投入，且技术永远没有“完美”的时候。虽然目前的虚拟现实、增强现实环境不够完善，但并不影响实验探索。

接下来引用了Ivan E.Sutherland在1965年发表的名为 [The Ultimate Display](https://my.eng.utah.edu/~cs6360/Readings/UltimateDisplay.pdf) 的文章，指出了部分关于混合现实的未来：

* “A chair displayed in such a room would be good enough to sit in”：一把能够坐下的椅子——现实-虚拟的无缝性（Seamlessness）
* “...handcuffs displayed in such a room would be confining...”：房间里的手铐——现实-虚拟的交互性（Interactivity）
* “...and a bullet displayed in such a room would be fatal”：房间里的子弹——现实-虚拟的可转换性（Transformability）
* “With appropriate programming such a display could literally be the Wonderland into which Alice walked.”：最终目标-像爱丽丝漫游仙境一样的完全沉浸的混合现实环境

划定领域：AI+IoT+混合现实的交叉领域。

![](https://github.com/HusterHope/blogimage/raw/master/20210428-1.png)

Blended Reality Space 定义了一个现实-虚拟环境(RV environment)，拥有一个或多个虚拟物体和角色，支持RV转换、交互和无缝衔接。

接下来提到现实-虚拟的轴线，这一概念也相对比较老了，类似的介绍可以参考博客中总结的[另一篇文章]()，此处不再展开。对于现实-虚拟世界的轴线，每个点都是一大块研究领域，我们有多种方式连接两个端点：

![](https://github.com/HusterHope/blogimage/raw/master/20210428-2.png)

## 感知过程和特性

在感知回路中，人类将经历如下过程：

![](https://github.com/HusterHope/blogimage/raw/master/20210428-3.png)

具体来说，

* 从感知（Perception）到认知（Cognition）：从离散的感官刺激中，感受器将信号发给大脑（bottom-up） ，大脑也会根据先验知识产生信号传给感受器（Top-Down）；
* 从认知（Cognition）到行为（Action）：大脑发出指导行为的信号（Top-Down），同时行为也将反向传递给大脑（Bottom-Up）；
* 随着行为的发生，感知信号也在不断更新，从而形成完整的感知回路。

在混合现实环境下，感知将获得两种反馈。一种是预测的反馈（Predicted Feedback），即相应行为发生后，混合现实环境**应该发生的**改变；另一种是观察到的反馈（Observed Feedback），即混合现实环境**实际产生的**改变。这两者之间会形成感知差异（例如追踪不稳定、时延等），当差异在一定范围内时，用户会本能地忽视这种差异。

不同模态的信息获取速度差异极大，下图给出了视、听、触、嗅、味5维信息速度差异。注意右下角一个极小的小白块，它占所有信息的0.7%，表示所有获取的信息中，人们真正在意到的部分。

![](https://github.com/HusterHope/blogimage/raw/master/20210428-4.png)

举例而言，对于人眼视觉系统，水平方向的视场角大约有180度，呈现全部视觉内容大约需要5.76亿像素。截至2021年，8K超高清视频直播能够达到约3000万像素，看上去全沉浸媒体似乎短时间内会是一种伪命题。然而基于上述感知过滤机制，人眼实际注视区域仅为1-2度，约700万像素，其他区域信息量大约为100万像素 。

![](https://github.com/HusterHope/blogimage/raw/master/20210428-5.png)

另一个比较有趣的例子来自一个视频短片：[Test Your Awareness : Whodunnit? - YouTube](https://www.youtube.com/watch?v=ubNF9QNEQLA) 

![](https://github.com/HusterHope/blogimage/raw/master/20210428-6.png)

跟随镜头语言的叙事，我们在一分多钟的时间中看到，即使开始和结束时场景中有多达20处变化，人眼也会选择性忽略。

在整个感知回路中，不同阶段均存在部分信息损失，这为相关技术的实现提供了可行性。

* 感知过程中-Perceptual Filtering：感知过程中的过滤（为编码压缩提供了可行性）
* 认知过程中-Selective Attention：选择性注意力（例如在嘈杂环境下捕捉某个特定声音）
* 行动过程中-Motor Adaptation：运动适应性（小范围内的反馈差别能够被神经系统忽略）

## 如何混合现实

这部分基本是讲一些近年来的研究工作，主要集中在重定向行走（Redirected Walking, RDW）和虚拟助手（Virtual Agent）两个方面。考虑到研究方向相关性，这里重点介绍前者。

在RDW方面，问题背景来源于虚拟环境和真实环境的空间大小不匹配，容易破坏用户体验，并产生一些危险情况。

![](https://github.com/HusterHope/blogimage/raw/master/20210428-7.png)

而前文提到过，人眼视觉系统其实并不精确，而VR环境中的6自由度包含了旋转和位移，因此可以从不同层面出发，分别进行一定程度的偏移处理和探索。

（下图出自 [这篇文章](https://zhuanlan.zhihu.com/p/33467983) ）

![](https://github.com/HusterHope/blogimage/raw/master/20210428-8.png)

* Translation gains（位置增益）：用户在真实环境中移动1个单位后，虚拟环境实际改变超过1个单位；
* Rotation gains（旋转增益）：用户在真实环境中原地旋转一定角度后，虚拟环境旋转不等于该角度；
* Curvature（曲率偏移）：虚拟环境给用户直线行走的错觉，但实际按照一定角度在真实环境中绕圈。

而找到这几种“欺骗”的上限一直是很关键的工作，随着设备显示质量和追踪精度的不断提升，相关阈值也在不断刷新。目前仍然需要大约5*5m的环境进行Redirected Walking。

![](https://github.com/HusterHope/blogimage/raw/master/20210428-9.png)

![](https://github.com/HusterHope/blogimage/raw/master/20210428-10.png)

此外，还有利用眨眼间隙进行场景微调的工作，可以通过神经网络预测**光照、场景内容变化**时，视点位置和眨眼行为的影响。

> 当然这部分算是Frank教授的私货，这篇工作是他带的phd研究生在做的点，相关成果目前尚未发表。

![](https://github.com/HusterHope/blogimage/raw/master/20210428-11.png)



而在虚拟助理方面，人们希望人工智能接近人类智能，并通过经典的图灵测试。下图展现了一个有趣的现象：

![](https://github.com/HusterHope/blogimage/raw/master/20210428-12.png)

通过上图可以看出，人类日常行为包含智能和非智能两部分。“非智能”可以是一些发呆、思考等行为形成的自然停顿，而这方面却是计算机系统鲜有考虑的，因为后者更倾向于输入-输出的连续和快速响应。

于是有相关研究工作在模拟真人行为方面做出探索，例如Schmidt等人在“Imperfect Virtual Agents”中提出：

> The greatest perfection is imperfection.

在图灵测试概念的基础上，McGuigan等人于2006年提出“Graphics Turing Test”，即从图形角度进行图灵测试。在过去几十年间，图形图像处理技术得到了高速发展，由此也产生了更多问题。当虚拟人物和真实人物差异性越来越小后，相关伦理、生理、心理等问题也将浮出水面。

混合现实的未来需要更多交叉领域的学者共同探索。

![](https://github.com/HusterHope/blogimage/raw/master/20210428-13.png)
---
title: "Reading | 移动端AR实时目标检测系统"
layout: post
categories: 读论文
tags:
  - AR
  - CV
  - DL
  - CloudComputing
  - ParallelProcessing
---

> Luyang Liu, Hongyu Li, and Marco Gruteser. 2019. Edge Assisted Real-time Object Detection for Mobile Augmented Reality. In Pro- ceedings of The 25th Annual International Conference on Mobile Computing and Networking, Los Cabos, Mexico, October 21–25, 2019 (MobiCom ’19), 16 pages. 

原文地址：[Edge Assisted Real-time Object Detection for Mobile Augmented Reality](http://www.winlab.rutgers.edu/~luyang/papers/mobicom19_augmented_reality.pdf)

<!-- more -->

整理一篇与增强现实(Augmented Reality，下文简称AR)相关的系统设计+实现论文，此文核心贡献是借助云计算实现了移动端AR实时目标检测。

同时，本文综合运用了深度学习模型和视频编码技术，结合系统设计和实现，得到了不错的结果，是对现有AR系统的一种补充。

## 问题背景

现有的移动端AR解决方案（如ARKit[1]和ARCore[2]）支持平面检测和空间锚（plane detection and object pining），但不能检测复杂物体和环境。而新型应用中对这类功能的需求日益增长，如驾驶行为和互动游戏等场景（见下图）。

![图1](https://github.com/HusterHope/blogimage/raw/master/20190428-1.jpg)



「实时目标检测」这个问题，最大的挑战在于**时延**和**检测精度**。在现存性能最强的AR终端上（文中指Magic Leap One[3]），运行最轻量级的目标检测模型，处理一帧高清视频大约需要50ms的时间，也就是一秒20帧。这无法达到每秒60帧的流畅体验，并且能耗和散热都存在问题。倘若借助无线网络和服务器进行云计算，则又会引入传输延迟，降低检测精度（原因下文展开），并占用大量网络带宽。

文章对于这两个问题分别进行了分析。

首先**对时延进行建模**。加入云计算辅助后，整个系统变成AR移动端和云计算端的架构。端到端的时延可以分为传输时延$t_{offload}$和渲染时延$t_{render}$两个部分。对每一帧视频数据流，传输时延包括了编码、传输、解码、目标检测、结果回传五个部分，即：

$$t_{offload}=t_{encode}+t_{trans}+t_{decode}+t_{infer}+t_{transback}$$ 

![图2](https://github.com/HusterHope/blogimage/raw/master/20190428-2.jpg)

然后是**检测精度**。由于视频内容不断更新，在用户视角变化后，上一帧的检测结果在新的内容中可能已经有了一定偏离值。经过文中实际测试，目标检测的错误率会在数帧延迟后飙升至五成以上（下图a）。

现存目标检测算法通常需要10ms以上来处理一帧，即$t_{infer}>10ms$（下图b）；此外，随着码率（Encoding bitrate）提升，传输时延$t_{trans}$也随之提高（下图c）；但码率降低也会影响到目标检测精度（下图d）。

![图3](https://github.com/HusterHope/blogimage/raw/master/20190428-3.jpg)

经过这些初步分析，我们发现时延、检测精度和视频质量之间存在明显的矛盾关系——借助现有架构或是仅凭AR移动端的计算能力，完全无法解决实时目标检测这一任务。即便加入了更强算力的云计算端，也会因为各类时延影响精度，导致最终性能无法达到需求。

下面介绍论文中提出的新系统架构和技术方法，来尝试解决这一问题。

## 系统架构和关键技术

整体来看，系统仍然是端到端架构，借助无线网传输数据。移动设备用于跟踪和渲染（tracking and rendering），云端运行目标检测的整个流程。架构如下图所示：

![图4](https://github.com/HusterHope/blogimage/raw/master/20190428-4.jpg)

下面对其中涉及到的几个关键技术作详细介绍，分别是：

* 动态RoI编码(DRE)
* 并行流推理(PSI)
* 基于运动矢量的目标跟踪(MvOT)
* 自适应云计算(Adaptive Offloading)

### Dynamic RoI Encoding(DRE)

*RoI(Region of Interest)*，即视频中用户感兴趣的区域。DRE则是根据RoI进行动态编码，其核心思想是，对内容中感兴趣的部分使用接近无损的压缩，对其他部分适当提高压缩效率，降低内容质量，从而降低总数据量。DRE常用于处理监控视频和360全景视频。

DRE的重点在于**如何确定RoI**。在AR场景下，一种最简单的策略就是将用户正在观察的区域作为RoI，但AR应用中常出现引起用户改变观察区域的事件，比如场景边缘突然出现一个虚拟物体，这时仅将观察区作为RoI就不合适了。值得注意的是，RoI这个概念既出现在视频压缩的概念中，也出现在作目标检测的CNN(Convolutional Neural Network)中。

借助这两者的联系，本文将目标检测的CNN得到的RoI作为下一帧视频压缩需要使用的RoI。不过这种方式也有一个明显的缺点，如果在非RoI区域出现新目标，则可能由于所在区域的压缩而无法被检测到（图3-d的现象）。解决方案是：适当扩大RoI区域，以及降低检测置信度的阈值。

> 有关「检测置信度阈值（prediction confidence threshold）」的个人理解：适当**提高查全率（recall），降低查准率（precision）**。比如在图像中原本一个区域可能是一棵树，也可能不是，那么在我们降低检测阈值后，检测算法仍然会将这个区域标记为一棵树。

这一步的主要过程如下图所示

![图5](https://github.com/HusterHope/blogimage/raw/master/20190428-5.jpg)

### Parallel Streaming and Inference(PSI)

并行流和推理。视频数据的每一帧比较容易按照空间维度划分(slice)，然后并行编码、传输、解码和推理（执行目标检测）。并行处理后能够显著降低时延。比如，将图像沿纵向划分为4等分后，时延变化如下图所示。

![图6](https://github.com/HusterHope/blogimage/raw/master/20190428-6.jpg)

对图像进行划分后再编码、传输、解码的过程比较自然，但进行目标检测时就遇到问题了。多个划分块之间的内容通常并不是彼此独立的，因此存在一些横跨多个划分块的物体。卷积运算在处理边界值时通常会作padding（填充一些0值作为补充），若在划分块上使用这种处理方式，将直接导致目标检测的结果与原来图像不同。

于是，作者这里引入了*有效区域(valid regiion)*的概念，基本思想是在每个划分块上控制卷积运算的范围，接收到后一个划分块时再计算连接处的值。有效区域的定义如下：

$$H_i^{out}=(H_i^{in}-1)/S+1$$ 

$$W_i^{out}=\left\{ \begin{aligned} &\frac{W_i^{in}-(F-1)/2-1}{S}+1,\quad & i=1,2,...,n-1 \\ &\frac{W_i^{in}-1}{S}+1,&\quad i=n \end{aligned} \right.$$ 

其中，$H_i^{out}$和$W_i^{out}$分别是有效区域的高(Height)和宽(Width)。由于是纵向划分，所以$H_i^{out}$保持不变；随着划分块的依次接收，$W_i^{out}$也逐渐增大。$S$为步长(Stride)，即卷积窗口每次滑动时的变化量；$F$为卷积空间范围(extent)，例如$5\times 5$的卷积核$F$则为5。根据定义推算，有效区域的范围比较直观。随着网络层数的加深，需要计算的有效区域也会逐渐缩小。

![](https://github.com/HusterHope/blogimage/raw/master/20190428-7.jpg)

### Motion vector based Object Tracking(MvOT)

运动矢量(Motion Vector)是视频编码中一个常见的方法（已用于H264/H265标准）。这个系统中引入运动矢量来提升目标检测的精度，可以说是一个比较巧妙的做法。基本思想比较简单，大致可以概括为：

将**上一次缓存帧(cache)的检测结果**与**缓存帧和当前帧对应物体的运动矢量**结合，得到当前帧的检测结果。

![](https://github.com/HusterHope/blogimage/raw/master/20190428-8.jpg)

在实际使用时，一旦延迟提升就会导致检测准确度大幅下降。不过借助硬件加速和前面提到的控制时延的方法，MvOT可以在保证目标检测准确的同时满足低时延。唯一的缺陷是，对于画面中新出现的物体，第一次检测可能稍有时延（因为无法使用运动矢量）。

### Adaptive Offloading

「Offloading」本意为卸载，这里用来指代**使一帧图像离开本地，交付云端计算**的过程。而这一节则描述了决定哪些帧需要进行云计算的选择策略。核心策略包含两条：

* 上一个交付云计算的帧，已被云端接收；
* 当前帧与上一个云计算的帧有显著差别。

第一条决策用来避免网络拥塞，实现时需根据上一次的传输延迟，决定是否发送。

第二条策略用来减少对计算资源的开销，实现的关键是估计两帧之间的差异。差异主要由两方面体现：（1）是否有大的**运动变化**（包含用户移动和场景中的目标运动），可以根据上一节得到的**运动矢量**进行估算；（2）是否有大量改变的像素，可以通过**帧内预测块和帧间预测块**进行估算（同样是视频编码中的技术，不多展开）

## 性能验证

主要检测以下性能指标：目标检测精度、检测时延、端到端跟踪和渲染时延、Offloading时延、带宽和计算资源消耗。具体步骤和相关数据在原论文中描述得比较细，自处不再展开。

总的来说，系统将检测精度提高了20.2%～34.8%，将错误率降低了27%～38.2%，offloading延迟降低了32.4%~50.1%，在AR设备上进行MvOT计算仅需2.24ms。

## 讨论

作者给出的本系统优缺点主要包括：

优点

* 泛化能力。系统主要提供了一个软件解决方案，可扩展到不同的硬件和操作系统上。
* 是对现有AR开发工具上的补充和拓展。

缺点

* 对网络条件要求严格，有一些问题未充分展开调研。

## 参考

[1] Apple ARKit. https://developer.apple.com/arkit/.

[2] Google ARCore. https://developers.google.com/ar/.

[3] Magic Leap One. https://www.magicleap.com/.
---
title: "Reading | 单张图像重建三维网格"
layout: post
categories: 读论文
tags:
  - CG
  - CV
  - DL
  - Re3D
  - SingleView
---

原文地址：[A Skeleton-bridged Deep Learning Approach for Generating Meshes of Complex Topologies from Single RGB Images](https://arxiv.org/pdf/1903.04704.pdf)

中文解读(来自paper weekly)：[CVPR 2019 | 基于骨架表达的单张图片三维物体重建方法](https://mp.weixin.qq.com/s/UthnL7fqEIEGvzcaXmFHKQ)

<!-- more -->

有段时间没更新paperlist了，正值CVPR2019中的工作陆续公开，跟进。

之前对于三维重建的理解和学习更多停留在多视图几何方面，也就是传统的Structure from Motion那一套过程。这几年随着神经网络的应用向着各个领域蔓延，三维重建和点云方面的工作也出现了很多。

虽然已经在paper weekly上有了一份比较完整的解读，但基本只是翻译，于是自己再做些整理，用于梳理工作的框架和主要思想（不少算法层面的细节还未能真正理顺）。

本文分为四个部分：问题定义、相关工作、方法解读和讨论。



## 问题定义

首先归纳一下三维物体的表达方式，主要包含以下三种：

* 体素（Volume）：可以看成是扩展到三维空间的像素
* 点云（Point Cloud）：从物体表面提取的点
* 网格（Mesh）：小面片的组合

然后是这个工作的输入与输出：

* 输入：单张RGB图像
* 输出：三维模型（网格表达）

## 相关工作

先简要回顾几类建模方法：

* 手工建模：使用3ds max, maya, blender之类的建模软件
* 摄影几何（Projection）：传统的Structure from Motion(SfM)那一套流程，在10年前左右有一波热潮[1]。
* 基于深度神经网络的方法（DeepNets based）：近几年随着深度学习大热之后产生的大量工作，在这篇paper中重点回顾。

上面提到三维物体的表达主要有三种形式，即体素，点云和网格。下面针对基于深度网络的方法分别列出几个主要前人工作。

* 基于体素：构建二维图像到三维占用区域（occupancy grid）的映射[2]。体素表达的优点在于比较规则，且直观；缺点是三维卷积运算通常具有高计算复杂度，导致模型分辨率较低。
* 基于点云：学习点集之间的相似性[3]。点云表达的优点是简单，因此易于学习和操作；缺点是精度较低，多数情况下比较稀疏。
* 基于网格：学习形变[4]。网格表达的优点是轻量，能够还原细节；缺点是不便于处理不规则拓扑结构的物体。

这样，我们归纳出「单张图像重建三维模型」的主要需求：高分辨率、高精度、适应复杂拓扑结构，以及容易由深度网络学习。这篇文章在这几方面需求上作出了一些创新，下面详细介绍。

## 方法解读

在介绍方法前，先看看实验结果。

![](https://github.com/HusterHope/blogimage/raw/master/20190417-1.jpeg)

可以看到，这篇文章最后的模型表达也是网格，但相比[4]而言，对于复杂拓扑结构的物体，模型质量有了明显提升。

整个方法主要分为三个步骤：

![](https://github.com/HusterHope/blogimage/raw/master/20190417-2.jpeg)

数据结构的变化为：图像($I$) -> 稠密点云（骨架$K$）-> 粗糙体素模型($V_k$) -> 体素模型($V$) -> 基本网格($M_b$) -> 输出网格($M$)。

### 从图像到骨架

方法的第一步。输入为图像$I$，输出为点云表达的骨架$K$。

* 数据集：ShapeNet-Skeleton

学习任务离不开数据集的支撑。本文的数据集由ShapeNet[5]构造而来，可以把ShapeNet看成三维的ImageNet。

> ShapeNet is an ongoing effort to establish a richly-annotated, large-scale dataset of 3D shapes.

构造过程包括：对模型表面采样得到点云数据 -> 提取骨骼点云 -> 对骨骼点进行分类。

提取骨骼的方法参考了[6]，分类过程是对点的邻域做主成分分析（PCA），分成了线状点(curve)和面状点(surface)，用于下一步处理。按作者说法数据集之后会开源，此处不深入探究了。

* CurSkeNet和SurSkeNet

上一步我们将骨骼中的点划分成了两类，这一步的两个网络则分别用于接收这两类点作为输入。同时，借助ResNet[7] 将原始图像编码为向量，与上述两类点云共同放入网络。

CurSkeNet和SurSkeNet的网络结构是多层感知机(MLP)，训练完成后将两类点云合并。

* 模型训练

损失函数是Chamfer Distance(CD)，正则化项为拉普拉斯平滑。表达式详见原始论文。

CD由两项组成，是一个对称的测度。前项是对每个预测值，找到最小的实际值，后项是对每个实际值，找到最小的预测值。

拉普拉斯平滑项用于确保局部一致性。

### 从骨架到网格

方法的第二步。输入为骨架$K$，经过网络学习得到体素模型$V$，后产生基本网格模型$M_b$作为输出。

![](https://github.com/HusterHope/blogimage/raw/master/20190417-3.jpeg)

第二步稍显复杂，我们进行适当拆解。

注意，网络学习后输出的体素模型$V$的分辨率为$128^3$，即$V_{128}$空间，可以看成若干个小立方体堆叠而成。这个学习步骤内包含两个并行结构：全局体素推理(Global Volume Inference)和子体素合成(Sub Volume Synthesis)。

首先将骨架转换成两种细分粒度下的体素模型$V_k^l=64^3$和$V_k^h=128^3$，前者用于Global，后者用于Sub，并对$V_k^h$行均匀裁剪(uniformly crop)，形成若干个$V_{64}$；

先看Global部分。与第一步相同，用ResNet编码图像，并用多个3D逆卷积层（deconvolution layers）将图像$I$变为$32^3$大小的体素。

> 卷积可以看成是数据降维运算，对应的逆卷积则是升维。

接下来在图像数据的引导下，对global $V_k^l$作体素修正(volume correction)，也就是以$32^3$大小的体素空间修正$64^3$空间中可能存在的误差。

在看Sub部分，与上面类似，对$V_k^h$裁剪后的每个$V_{64}$，用全局体素$V_k^l$进行引导。

然后进行子体素合成，并用U-Net结构[8]进行体素提炼(Volume refinement)，得到体素模型$V$。

> U-Net是一种来自医学领域的处理方法，具体可参考对应文献，这里不探究其细节。

最后，从体素模型提取网格，使用图形学的经典算法Marching Cube[9]，并对提取到的网格用QEM算法[10]进行简化。得到这一步的基本网格模型$M_b$作为输出。

### 对基础网格形变

方法的第三步。输入为基础网格$M_b$，输出为最终网格$M$。主要用于恢复模型的细节。

![](https://github.com/HusterHope/blogimage/raw/master/20190417-4.jpeg)

这一步学习模型形变的思想和[4]类似，将输入图像作为参考。

* VGG-16[11]在这里的作用可以看成是提取图像特征
* GCNN是图神经网络，用于处理非欧氏结构

> 二维图像都是均匀分布的像素点，三维网格则是具有复杂拓扑结构的图，GCNN可以看成是CNN的抽象

这一步的过程还需再学习一下，先不乱讲

## 讨论

这篇文章能够被顶会接收，单从思想上来看，有这么几个值得学习的点：

* 问题定义明确
  * 重点克服复杂的拓扑结构，尤其是像桌椅这类具有细长结构和孔洞的物体
* 充分利用不同形式的三维模型，并采用分步方法
  * 大问题：从图像到网格 -> 用点云结构的骨架作桥梁
  * 小问题：从骨架到网格 -> 用体素结构作桥梁
* 跨学科知识
  * 计算机视觉：Deep learning相关的应用
  * 计算机图形学：处理三维模型的经典算法
  * 医学：一些数据处理的特殊方法
* ...

与实践相结合，共同构成了一份有价值的工作。



ps.不少细节尚未梳理，如有疑惑和纰漏，请不吝赐教。



## 参考文献

[1] Agarwal, Sameer, Noah Snavely, Ian Simon, Steven M. Seitz and Richard Szeliski. “Building Rome in a day.” *ICCV* (2009).

[2] Choy, Christopher Bongsoo, Danfei Xu, JunYoung Gwak, Kevin Chen and Silvio Savarese. “3D-R2N2: A Unified Approach for Single and Multi-view 3D Object Reconstruction.” *ECCV* (2016).

[3] Fan, Haoqiang, Hao Su and Leonidas J. Guibas. “A Point Set Generation Network for 3D Object Reconstruction from a Single Image.” *2017 IEEE Conference on Computer Vision and Pattern Recognition (CVPR)* (2017): 2463-2471.

[4] Wang, Nanyang, Yinda Zhang, Zhuwen Li, Yanwei Fu, Wei Liu and Yu-Gang Jiang. “Pixel2Mesh: Generating 3D Mesh Models from Single RGB Images.” *ECCV* (2018).

[5] Chang, Angel Xuan et al. “ShapeNet: An Information-Rich 3D Model Repository.” *CoRR* abs/1512.03012 (2015): n. pag.

[6] Wu, Shihao, Hui Huang, Minglun Gong, Matthias Zwicker and Daniel Cohen-Or. “Deep points consolidation.” *ACM Trans. Graph.* 34 (2015): 176:1-176:13.

[7] He, Kaiming et al. “Deep Residual Learning for Image Recognition.” 2016 IEEE Conference on Computer Vision and Pattern Recognition (CVPR) (2016): 770-778.

[8] Ronneberger, Olaf et al. “U-Net: Convolutional Networks for Biomedical Image Segmentation.” *MICCAI* (2015).

[9] Lorensen, William E. and Harvey E. Cline. “Marching cubes: A high resolution 3D surface construction algorithm.” *SIGGRAPH* (1987).

[10] Kobbelt, Leif, Swen Campagna and Hans-Peter Seidel. “A General Framework for Mesh Decimation.” *Graphics Interface* (1998).

[11] Simonyan, Karen and Andrew Zisserman. “Very Deep Convolutional Networks for Large-Scale Image Recognition.” *CoRR* abs/1409.1556 (2015): n. pag.
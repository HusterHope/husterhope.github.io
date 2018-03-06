---
title: "对比几个三维重建系统"
layout: post
categories: 解问题
tags:
  - 3D
  - SfM
  - MVS
---



本文简要介绍三维重建的基本流程，列举若干常见系统，给出项目和文档地址，比较它们的工作管线，为深入钻研系统结构作为铺垫。

### 三维重建概述

我们知道，照相机/摄像机的原理是将一个三维场景或物体投影到二维平面上，过去是胶片，现在是经过感光元件再记录到存储器。降维的过程通常不可避免地会存在信息的损失，而所谓的重建(Reconstruction)，顾名思义就是要从获取到的二维图像中复原原始三维场景或物体。

相对照相机的诞生，三维重建的概念本身说年轻也算年轻；但相比从2012年开始比较火的深度学习、神经网络等概念，三维重建也算是古老的研究领域了，因为第一个基于图像的三维重建系统诞生于1992年（CMU的Tomasi等人的工作）。

三维重建的流程大致如下：首先，通过多角度拍摄或者从视频中提取得到一组图像序列，将这些图像序列作为整个系统的输入；随后，在多视角的图像中，根据纹理特征提取出稀疏特征点（称为点云），通过这些特征点估计相机位置和参数；在得到相机参数并完成特征点匹配后，我们就可以获得更稠密的点云（这些点可以附带颜色，从远处看就像还原了物体本身一样，但从近处能明显看出它们只是一些点）；最后根据这些点重建物体表面，并进行纹理映射，就还原出三维场景和物体了。

概括起来就是：图像获取->特征匹配->深度估计->稀疏点云->相机参数估计->稠密点云->表面重建->纹理映射

目前已有不少免费系统（部分系统已开源）能够完成三维重建的整个流程，在深入研读某一个系统的代码之前，我决定先根据前人资料横向比较一下这些系统的异同。除了亲自运行了几个系统外，主要参考了[这个视频](https://www.youtube.com/watch?v=ELHOjC_V-FE)的讲解（需要科学上网），以下内容的部分截图也出自这里。感谢视频原作者和以下每个系统的创造者。

---

### 现有系统简要对比

![](http://ohn6qfqhe.bkt.clouddn.com/SC-1.jpg)

图中从左至右的过程依次是从原始图像到稀疏点云、重建稠密点云、重建表面和纹理映射。对应列的绿色代表该系统具备此功能，红色反之。

#### [VisualSFM](http://ccwu.me/vsfm/): A Visual Structure from Motion System

是否开源：否

[文档](http://ccwu.me/vsfm/doc.html)：小规模 (5k词左右)

兼容性：兼容OS X/Windows/Linux

主要特性：完成从图像到稠密点云的重建过程，值得注意的是，VSFM系统的输出可以放入CMP-MVS（Multi-View Reconstruction Preserving Weakly-Supported Surfaces, CVPR 2011），[MVE](https://www.gcc.tu-darmstadt.de/home/proj/mve/index.en.jsp)（by Michael Goesele's research group），[SURE](http://www.ifp.uni-stuttgart.de/publications/software/sure/index.en.html)（by Mathias Rothermel and Konrad Wenzel）, [MeshRecon](http://zhuoliang.me/meshrecon.html)（by Zhuoliang Kang）等系统中进行后处理。

视频中展示的稠密点云精度比较高，但构建速度较慢（讲解者在重构一个人形塑像时用了1小时40分钟）。

![](http://ohn6qfqhe.bkt.clouddn.com/SC-2.png)

#### [Meshlab](http://www.meshlab.net/): the open source system for processing and editing 3D triangular meshes. 

是否开源：[是](https://github.com/cnr-isti-vclab/meshlab)

文档：无详细文档，但有视频教程

兼容性：兼容OS X/Windows/Linux

主要特性：从稠密点云进行表面重建，并完成纹理映射。

附加：可选取感兴趣的区域(Region of Interest, ROI)进行编辑，比如降低面片数量、删除区域外的稀疏点等。

此外，诸多其他系统的点云结果可在本系统内查看。（下图红色区域为用户选择的非感兴趣区域，可进行删除）

![](http://ohn6qfqhe.bkt.clouddn.com/SC-3.png)

#### [Colmap](https://colmap.github.io/): a general-purpose Structure-from-Motion (SfM) and Multi-View Stereo (MVS) pipeline with a graphical and command-line interface.

是否开源：[是](https://github.com/colmap/colmap)

[文档](https://colmap.github.io/)：有详细说明，共13章节

兼容性：兼容OS X/Windows/Linux

主要特性：输入多视角图像，输出稀疏点云并估计相机参数，后续步骤需基于[CUDA](https://en.wikipedia.org/wiki/CUDA)实现。

实际体验效果：

![](http://ohn6qfqhe.bkt.clouddn.com/SC-4.png)

可见重建精度还是不错的，并且整体操作上比较快。

#### [Bundler](http://www.cs.cornell.edu/~snavely/bundler/): Structure from Motion (SfM) for Unordered Image Collections

是否开源：[是](https://github.com/snavely/bundler_sfm)

文档：小规模(5k词左右)，无详细接口说明

兼容性：虽然已发布windows版本，但开发和测试主要集中在Linux平台

主要特性：输入多视角图像，输出稀疏点云并估计相机参数，与Colmap类似，视频显示在实际处理中Bundler速度更快(30-40min)。虽然精度和点云密度不如colmap和visualSFM高，但本系统可以计算相机参数，因此可放入下面的CMVS系统中进行后处理。

下图为bundler处理后的结果（在meshlab中查看），能够明显看到点云密度不高。

![](http://ohn6qfqhe.bkt.clouddn.com/SC-5.png)

#### [CMVS](https://www.di.ens.fr/cmvs/): Clustering Views for Multi-view Stereo

是否开源：[是](https://github.com/pmoulon/CMVS-PMVS)

文档：小规模，无接口说明

兼容性：有windows版本，linux为命令行版本

主要特性：将SfM系统的输出（稀疏点云）作为本系统的输入，本系统可将输入图像分解成一组可管理大小的图像集群。之后可以使用MVS软件独立并且并行地处理每个集群。本系统应该与SfM软件（例如Bundler）和MVS软件（例如PMVS2）结合使用。

不过视频作者的例子中，bundler的输出放在本系统中作为输入可能会存在问题，使用时还需注意。（下图为meshlab中查看的处理结果，细节丢失较严重）

![](http://ohn6qfqhe.bkt.clouddn.com/SC-6.png)

#### [MVE](https://www.gcc.tu-darmstadt.de/home/proj/mve/): a complete end-to-end pipeline for image-based geometry reconstruction.

是否开源：[是](https://github.com/simonfuhrmann/mve)

[文档](https://github.com/simonfuhrmann/mve/wiki/MVE-Users-Guide)：中等规模，有示例，有管线说明。

兼容性：windows/OS X均可运行，文档说更推荐用OS X系统。

主要特性：包含了SfM, 点云重建，表面重建。基本上能够完成三维重建的整个流程。

实际体验：已编译运行，未进行完整操作流程。下图为MVE的用户界面

![](http://ohn6qfqhe.bkt.clouddn.com/SC-7.png)

#### [MVS-Texturing](https://www.gcc.tu-darmstadt.de/home/proj/texrecon/): 3D Reconstruction Texturing

是否开源：[是](https://github.com/nmoehrle/mvs-texturing)

文档：小规模，有用法说明。

兼容性：命令行程序

主要特性：能够完成大规模图像输入的纹理映射。特别是能够在缩放、模糊、遮挡物、曝光变化的情况下完成高质量的纹理映射。最后的效果比较干净。

![](http://ohn6qfqhe.bkt.clouddn.com/SC-8.jpg)

#### [OpenMVG](http://imagine.enpc.fr/~moulonp/openMVG/): open Multiple View Geometry

是否开源：[是]()

[文档](http://openmvg.readthedocs.io/en/latest/)：大规模，分8个章节。

兼容性：三大平台

主要功能：精确解决多视图几何中的常见问题；保证代码可读性和易用性。

#### [OpenMVS](http://cdcseacave.github.io/openMVS/): open Multi-View Stereo reconstruction library

是否开源：[是](https://github.com/cdcseacave/openMVS)

[文档](https://github.com/cdcseacave/openMVS/wiki)：中等规模，分5节，逻辑比较顺畅。

兼容性：三大平台均可编译

主要功能：稠密点云重建、表面重建、表面细化、纹理映射。

值得注意的是表面细化(mesh refinement)这一步，是其他系统中比较少见的，效果如下：

![](http://ohn6qfqhe.bkt.clouddn.com/SC-9.jpg)

![](http://ohn6qfqhe.bkt.clouddn.com/SC-10.jpg)

虽然OpenMVS和上面的OpenMVG并不是同一个团队所作，但这两个系统恰好一起完成了三维重建的整个流程，可以协同使用。

---

小结：对比现有若干系统后，我决定从文档较规范的系统入手，仔细挖掘其中的关键函数和对应的作用，试着逐渐理解。比如可视化程度比较好的meshlab，更适合OS X平台的MVE，以及能够配套使用的完整库OpenMVG和OpenMVS。顺便挖一些有价值的论文一起啃。未来几个月的任务仍然艰巨，不可懈怠。

此外，这份文档处于入门级别，仅仅是粗略记录这些系统的功能，方便找到项目地址。仅供参考，欢迎交流。
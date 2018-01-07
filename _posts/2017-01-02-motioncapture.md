---
title: "动作捕捉数据的存储"
layout: post
excerpt: "运动捕捉的数据有许多格式，包括C3D、ASF/AMC、BVH、TRC、FBX等。"
categories: 做笔记
tags:
  - MotionCapture
---



运动捕捉的数据有许多格式，包括C3D、ASF/AMC、BVH、TRC、FBX等。

每一种文件格式都是分块、分层的。由于文件格式不同，数据信息项各不一样，所以存取方法也各不相同。下面对这些数据格式做一些简单的区分。

### 1.C3D格式

C3D格式功能十分强大。它通常由多个512字节的信息块组成，这些信息块包含C3D文件中的各个段和记录。所有C3D文件包含三个或更多个段，每个段由至少一个（512字节）信息块组成。在C3D文件的部分中，信息存储在记录中。所有C3D文件包含：

（1）标题记录（即标题部分）

（2）存储在参数部分内的参数记录

（3）存储在数据部分内的数据记录。

更多有关C3D文件格式的说明请参考C3D说明手册（全英文，从CSDN论坛扒来的，居然还要两点积分，这里直接放度盘方便取阅）

> 链接: <https://pan.baidu.com/s/1qYhffAG> 密码: xe8r

### 2.ASF/AMC格式

ASF/AMC文件格式是Acclaim Games公司设计开发的，全称是Acclaim Skeleton File/Acclaim Motion Capture data，其中ASF是骨架文件信息，AMC是运动数据信息。分开存储的好处就是一个ASF骨架可以运用到不同的AMC中，而不必要对每个AMC运动数据都写入一个ASF骨架信息，减少了数据冗余。

其实说了上面一大段的概述，也未必能对整个文件格式有本质了解。幸运的是，卡内基梅隆大学（Carnegie Mellon University）的课程数据库提供了很多现有的ASF/AMC数据文件：

<http://mocap.cs.cmu.edu/>

示例数据（在数据库中检索run得到）：

AMC（动作数据）：<http://mocap.cs.cmu.edu/subjects/02/02_03.amc>

ASF（骨骼数据，链接为下载链接）：<http://mocap.cs.cmu.edu/subjects/02/02.asf>

对这种动作数据的读取可以使用Neil Lawrence的MOCAP工具包：<https://github.com/lawrennd/mocap>

CSDN上的一篇文章对数据字段有较详细的描述，引作参考：

<http://blog.csdn.net/zb1165048017/article/details/49339493>

### 3.BVH格式

BVH：Biovision hierarchical data，它是在BVA格式的基础上的改进，在动作捕获后解析出来的文件。
BVH文件分为2个主要部分：骨架信息和数据块。

（1）骨架信息：按照层级关系，定义了root、hip、leg等位置和旋转分量，从而形成一个完整的骨架。

（2）数据块：对应上面的骨架各部位，标出每帧的数据信息。

原始说明参考自威斯康星大学麦迪逊分校（University of Wisconsin-Madison）的课程数据库：

<http://research.cs.wisc.edu/graphics/Courses/cs-838-1999/Jeff/BVH.html>

对BVH的详细说明可以参考如下博客：

<http://blog.sina.com.cn/s/blog_4ae49c20010007ih.html>

### 4.TRC格式

这个就是我们课程所使用的设备（东方新锐DVMC-8820）的数据输出格式。

这个格式相比前面的每一种来说都要简单不少。因为有数据源，所以可以查看数据本身来了解各个字段的含义，实验过程中所输出的示例文件可从以下度盘链接下载。

> 链接: <https://pan.baidu.com/s/1qY2dKde> 密码: pmg8

我们不难看出，这种文件格式包含两个部分的信息：说明部分和数据部分。说明部分主要是数据频率、相机频率、帧数等参数的说明，数据部分则是直接存放每一帧中每一个标示球所在的位置。由于这种格式的文件没有骨架信息，所以需要在相应可视化软件里面手动把数据点和骨架相连，例如MotionBuilder。

在简便易读的同时，它的缺点也很明显。因为缺少骨架和正/反向动力学的约束，导致某些异常的数据（噪点）也被保存了下来，这会直接影响到最后的角色动作，形成动作突变、骨骼错位等问题，给后期处理带来极大的不便。

### 5.FBX格式

这里仅说一些浅显的理解。

FBX是filmbox这套软件所使用的格式，现在改称Motionbuilder。

FBX可以存放模型的贴图、骨骼、动画等信息，因此在各种建模软件和游戏引擎中都有应用（maya、3ds max、Unity等），较好地解决了多个程序的文件格式兼容性问题（虽然有时会出现一些贴图丢失的bug）3ds max和Maya均支持导入、导出FBX；FBX的模型文件在Unity中可以得到较好的显示。

------

End.


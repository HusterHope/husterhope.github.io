---
title: "Debug｜多视点中间图像生成"
layout: post
categories: 解问题
tags:
  - CUDA
  - OpenCV
  - TextureRendering
  - Environment
---

近期接手了一个中间视点生成的项目。参考论文[1]寻找解决方案，对应的Github复现工作为[2]。本文重点对复现的代码进行解读。

<!-- more -->

## 基本原理

简单描述一下论文和对应代码的原理。不同于常规利用深度估计进行视点合成的算法[3]，论文的主要思路是采用图像变形(Morphing)。

![](https://github.com/HusterHope/blogimage/raw/master/20200804-1.png)

**这个方法的基本过程包括：图像特征提取->特征滤波和匹配->三角化->计算仿射变换矩阵->计算变换中间值->实时渲染。以及论文里提出的部分优化方法。**

在阅读复现的代码时，会额外发现一些繁琐却不可或缺的部分，如二进制文件的I/O操作、数据结构组织、高斯模糊、CUDA并行计算、OpenGL渲染框架等。

此外，复现的代码并没有按照原论文完整实现优化的细节，而只是基本写通了整个框架，输出结果并不十分理想（见项目概览）。论文作者本人估计也因为项目年代久远，没有公开在文中提到的数据集和应用程序，更没有回复询问的邮件。

输出两幅图像中间任意位置的内容，核心是寻找仿射变换矩阵。对于一个三角形而言，基本原理如下图所示[4]：

![](https://github.com/HusterHope/blogimage/raw/master/20200804-2.png)

整幅图像显然不能直接进行这样的变换，因此引入特征提取和匹配过程，将图像划分为若干个小三角形，再逐一寻找映射关系，并利用差值输出中间位置的。

## 源码编译

这个过程存在一些奇怪的坑，本打算仔细记录每一步的问题，但前后编译碰到的问题多到让我严重怀疑这个工程是不是真的能跑通，以至于到了终于通起来的时候，已经忘了很多踩过的坑了。于是这里只写几个卡了很久的地方。

环境：CUDA 11.0 + Win10 + VS2015 + OpenCV 3.2

* CUDA配置

  由于换设备后没有完整安装CUDA，需要首先解决这个环境的问题。

  我在折腾的时候发现需要卸载所有残留再进行ToolKit完整安装，才能够正常运行

* VS+CUDA

  CUDA在系统中正常测试没问题后，放入VS测试代码发现提示一堆“unresolved external symbol“。这个问题尝试了StackOverFlow和CSDN上的几种常见解决方案，最终通过在项目属性中引入`cudart.lib`等相关库解决（Property-Configuration properties-Linker-Input-Additional Dependencies）

* AKAZE库若干问题

  与AKAZE相关的报错，使用Cmake(gui)对`akaze-cpuidentity`进行重编译，注意指定版本。

* 配置文件问题

  编译时还遇到过OpenCV读取视频出错，但读取图像就没有任何问题的情况。解决过程包括更换项目目录名（不要有中文路径）；将路径中的单斜杠`\`换成双斜杠`\\`等（后面这个问题非常罕见，可能因设备不同而随机出现）。

其他各种小问题基本能够通过Google解决。

## 项目概览

正如代码作者在`Readme`中所述，本项目在大的框架上分为两个模块：`mvv_demo`和`cuda_demo`。前一模块进行特征提取和滤波，并将匹配后的特征点进行三角化，同时存储对应三角形的仿射变换关系；后一模块利用CUDA做并行计算，对仿射变换的插值结果进行实时渲染。

![](https://github.com/HusterHope/blogimage/raw/master/20200804-3.png)

将项目完整Clone到本地后得到上述文件夹，两个模块的VS项目文件分别位于`MVV1/mvv_demo/mvv_demo.sln`和`MVV1/cuda_demo/cuda_demo.sln`，程序入口分别在`kernel.cu`和`cuda_interop.cu`处。`mvv_demo`包含三个子项目：`build_geometry`、`gpu_wrapper`和`interpolate_images`，`gpu_wrapper`将另外两个库作为引用；`cuda_demo`仅包含`cuda_interop`单个工程，规模相对小。

从初步跑通的结果看，还存在大量三角面片空间映射位置的错误，以及图像轮廓附近的空洞。下方左图为原始视频的某一帧，右图为本项目合成后的某一帧中间视点。

![](https://github.com/HusterHope/blogimage/raw/master/20200804-4.png)

为了尝试理解这个效果的产生原因，决定对整个项目代码全部挖一遍。

（8月5日：已在文末更新Bug产生原因和输出结果）

## 模块解构

### MVV_DEMO

先从`mvv_demo`的入口`Kernel.cu`开始，除掉开头对视频的读取操作和条件判断，进入核心函数`video_loop`。这里面包含采用OpenCV对两个视频进行同步、逐帧存储两个视频文件、**每隔20帧使用AKAZE进行特征提取和匹配，并存储对应的仿射变换参数。**

`video_loop`的核心又来到了`save_frame_master`，它的功能涵盖了上一段加粗的内容。关键的**特征提取**部分包含3步：

* 使用AKAZE从两个视频中逐帧提取特征点
* 特征滤波：Lowe’s ratio test[5]
* 特征滤波：RANSAC

紧跟着是非常关键的**三角划分(triangulation)**步骤，包含在`create_matched_geometry()`函数中，并在`build_geometry.cpp`内实现，三角划分的核心算法直接使用OpenCV框架内的subdiv2D\[6\]（如有需要，后续在单独深挖该部分代码）

三角划分结束后，进入**栅格化(raster)**操作，即记录每个像素点所属的三角形，便于后续进行并行化处理。此处对于每一个三角形，我们已知它的三个顶点坐标，需要计算出内部所有像素点的坐标，并进行标记。方法是：

* 找出三角形的最小包围盒(bounding box)，得到一个矩形；
* 遍历该矩形内的所有像素点，判断其是否在三角形内。

> 判断方式有很多，作者这里自己实现了一个`orient2d`函数，判断行列式：
> $$
> \
> \left|\begin{array}{cccc} 
> a_x &    a_y    & 1 \\ 
> b_x &    b_y    & 1\\ 
> c_x &    c_y    & 1 
> \end{array}\right| 
> \
> $$
> 的正负性，从而判断点和三角形的相对位置[7]。对每个三角形，记录所有在其内部的像素点坐标。

对栅格化的结果，先将所有像素的索引均置为0，再遍历每个三角形内部的点，对第`i`个三角形，内部像素点设定索引为`i+1`。将索引结果存入前缀为`raster`的二进制文件中

接下来**构造仿射变换矩阵**，OpenCV的实现是一个2*3大小的矩阵，记录了2维空间内两个三角形对应的缩放、旋转和平移信息\[8\]，并将正变换和逆变换共12个参数存入前缀为`affine`的二进制文件中。

`mvv_demo`的最后，通过函数`save_img_binary`将图像拉伸为一维向量，存入前缀为`img`的二进制文件中。

### CUDA_DEMO

相比`mvv_demo`的过程，`cuda_demo`的代码量要少很多，本质上理解了**并行处理的方式以及仿射变换的计算**即可，剩下的大多是输入/输出操作和空间开辟/释放操作。

大致流程是：CUDA环境设置->OpenGL环境设置->并行计算

> 可以通过编译CUDA示例程序来查看本机CUDA的详细情况（Windows环境），来确认并行计算时能够划分的块大小和线程数量。
>
> 默认路径（版本11.0）：/NVIDIA Corporation/CUDA Samples/v11.0/1_Utilities/deviceQuery

CUDA的环境设置主要是内存空间的开辟和Grid/Block/Thread的划分，有关Grid/Block/Thread的区别可以参考\[9\]\[10\]。本项目中，作者将每一个像素点作为一个线程，block大小为32*32。

> OpenGL的部分在以前的博客中有专门的Tag进行讨论，此处不作深入描述。

**仿射变换**的部分，代码中引入了1/4像素的机制，以及调整差值位置的参数$\tau​$。设仿射变换矩阵为$A​$，对位于$(c,r)​$的像素点而言，代码中实现映射位置的计算方式为：

$for\ i\ in\ range(4):$ 

$$new_c=(1-\tau+\tau\times A_{11})\times(col-0.5+\frac{i}{4})+\tau\times A_{12}\times(row-0.5+\frac{i}{4})+\tau \times A_{13}$$ 
	
$$new_r=\tau\times A_{21}\times(col-0.5+\frac{i}{4})+(1-\tau+\tau\times A_{12})\times(row-0.5+\frac{i}{4})+\tau \times A_{13}$$

> 暂时没有理解$(1-\tau)$在式中的作用。

---

为了让整个过程更完整且清晰，创建了一个思维导图，代码看到哪就记录到哪，看完的时候整张图大致如下：

![](https://github.com/HusterHope/blogimage/raw/master/20200804-5.png)

## 后续

预计：

* 将划分的三角形在图像中可视化，分析映射后空洞的形成原因。
* 实现一些边缘填充的方法。

8月5日更新

利用OpenCV将三角化的结果输出，发现映射关系并没有问题：

![](https://github.com/HusterHope/blogimage/raw/master/20200804-6.png)

这就又回到仔细挖掘I/O代码的状态了，最终在`cuda_demo`项目的`video_kernel.cu`文件中，发现了如下语句：

```
short offset = (affine_index - 1) * 12;
```

这一行的作用是定位仿射变换参数的索引，从可视化的结果来看，三角形的个数大约在几千个。于是怀疑这个读取存在数据溢出，遂将`short`改为`int`。

然后居然就，效果正常了。。

---

很少有逐行分析完整工程代码的经验，这次阅读代码的过程大概持续了5天（编译大概还花了3-4天），在实现细节上还是学到了很多论文里看不到的东西。

后续也要更多培养动手习惯和读更大规模代码的能力。

另外，也完成了上一篇文章开头的flag。

> 如果有经常关注这个博客的读者，这里说一声「抱歉」。
>
> 近半年没有形成多少像样的产出，也删改了一些曾经发过的恋爱记录和心理上的波动。6月的一场失恋后，朋友评论说我「不好好写博客了」。随着重返北京后环境的变化，心态和工作效率已有明显改善。
>
> 顺便定一个小目标：这个博客坚持写到毕业。

## 参考

\[1\] [Spatial and Temporal Interpolation of Multi-view Image Sequences](<https://www.researchgate.net/publication/301950085_Spatial_and_Temporal_Interpolation_of_Multi-view_Image_Sequences> )

\[2\] [szat/MVV1 - Github](<https://github.com/szat/MVV1>)

\[3\] [真实场景的虚拟视点合成详解 - cnblogs](<https://www.cnblogs.com/riddick/category/1085181.html>)

\[4\] [Geometric Operations](<https://www.cs.auckland.ac.nz/courses/compsci773s1c/lectures/ImageProcessing-html/topic2.htm>)

\[5\] [Distinctive Image Features from Scale-Invariant Keypoints - David G. Lowe](<https://www.cs.ubc.ca/~lowe/papers/ijcv04.pdf>)

\[6\] [opencv/modules/imgproc/src/subdivision2d.cpp - Github](<https://github.com/opencv/opencv/blob/808ba552c532408bddd5fe51784cf4209296448a/modules/imgproc/src/subdivision2d.cpp>)

\[7\] [二维平面上判断点是否在三角形内(算法4) - cnblogs](<https://www.cnblogs.com/TenosDoIt/p/4024413.html>)

\[8\] [Affine Transformations - OpenCV](<https://docs.opencv.org/3.4/d4/d61/tutorial_warp_affine.html>)

\[9\] [Difference between threadIdx, blockIdx statements - Nvidia Developer](<https://forums.developer.nvidia.com/t/difference-between-threadidx-blockidx-statements/12161>)

\[10\] [【CUDA】grid、block、thread的关系及thread索引的计算 - CSDN](<https://blog.csdn.net/zhangzhe_0305/article/details/78063238>)
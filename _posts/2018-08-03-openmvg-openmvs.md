---
title: "Ubuntu16.04使用OpenMVG+OpenMVS"
layout: post
categories: 解问题
tags:
  - Re3D
  - SfM
  - MVS
  - PointCloud
  - Linux
  - Ubuntu
  - Environment
  - TextureRendering
---

在之前的三维重建系统相关文章中，我先后尝试使用了Bundler/PMVS-CMVS/Theia/Meshlab等重建工具，并采用Bundler+PMVS-CMVS重建管线作为实现端到端系统的核心方法。

存在的几个主要缺陷有：

* 未进行表面重建+纹理映射步骤，没有实现完整的重建过程。
* 使用的重建方法较老，源已很久未更新和维护。
* 支持的输入有很大局限性。
* 其它未列出的小问题..

考虑到以上种种问题，为了后续能让三维重建系统的研究工作更加接近state of art，只好放弃原来使用的系统，改用更加完善的系统作为优化和研究的重心，也就是本文要介绍的OpenMVG+OpenMVS。

<!-- more -->

从下载源码，安装依赖，编译，运行样例，构造完整流程脚本和测试，整个过程差不多花了一周时间。本文会梳理一些坑，并在最后给出这两个库的重建效果。

## 项目地址

[OpenMVG](https://github.com/openMVG/openMVG) | [OpenMVS](https://github.com/cdcseacave/openMVS) | [Meshlab](http://www.meshlab.net/)

之前整理过一篇对比各个重建系统功能的文章（[链接](https://leohope.com/%E8%A7%A3%E9%97%AE%E9%A2%98/2018/03/06/compare-re3d-system/)），里面简单介绍了OpenMVG和OpenMVS在整个三维重建管线中能够完成的不同功能。

> 常用的多视图三维重建管线：Structure from Motion(SfM) -> Multi-View Stereo(MVS) -> Surface Generation(SG) -> Texture Mapping(TM)
>
> 实际使用后发现，链接文章里的参考图片列出的OpenMVG和OpenMVS功能中有个细节错误：OpenMVG完成的步骤仅限于SfM，后续MVS/SG和TM都由OpenMVS实现。

为了更容易照顾到旧环境，之前使用的操作系统一直是Ubuntu14.04。但使用OpenMVG和OpenMVS需要的依赖库更适合用16.04来安装（比如GLFW3，Ubuntu14.04默认的apt-get安装会使用GLFW2），且OpenMVS的官方编译说明文档也用16.04作为环境。于是再次忍痛备份+卸载虚拟机，并使用16.04版本的镜像来安装新系统。

Meshlab则是一款非常方便的点云/网格浏览器，在本项目中用于查看模型的生成效果。

用了几天，似乎16.04和14.04在操作习惯上没什么差别

## Linux编译

> 编译如同炼丹，你永远不知道什么时候会卡在什么位置。

### OpenMVG

OpenMVG的编译基本上参考[官方文档](https://github.com/openMVG/openMVG/blob/master/BUILD.md#linux)，比较顺利的通过了。

如需使用OpenMVG+PMVS-CMVS管线，可参考[这篇文章](http://tombraiderjf.com/2018/01/02/openMVG%E5%92%8CPMVS%E4%B8%89%E7%BB%B4%E7%82%B9%E4%BA%91%E9%87%8D%E5%BB%BA/)。

### OpenMVS

照例先上[官方文档](https://github.com/cdcseacave/openMVS/wiki/Building)。

这个库的坑就比OpenMVG多了不少，列出几个坑。

* main_path

在官方文档的第三步，会有一句指令：

```
main_path=`pwd`
```

一开始也没在意就照做了，后来发现这个值在编译后面某个依赖库(VCGLib)的时候是要用到的！

所以这个`main_path`应该填写为vcglib文件夹所在的目录，便于后面编译。

* USE_CUDA

因为我的主机（虚拟机的宿主机）不带NVIDIA的GPU，所以用不了CUDA，这个地方需要在openMVS目录下找到`CmakeLists.txt`文件，找到CUDA开关，设置为OFF：

```
SET(OpenMVS_USE_CUDA OFF CACHE BOOL "Enable CUDA library")
```

* boost

眼看着编译基本ok了，到了百分之七八十的时候就崩。检查不出原因。

最后在issue列表里反复找解决方案，在试过若干种解决策略后，一条不起眼的评论引起了我的注意：（参见[Github issue](https://github.com/cdcseacave/openMVS/issues/346)）

![](https://github.com/HusterHope/blogimage/raw/master/mvgmvs-1.jpg)

说是要给boost库升级到1.63。

然后照着[这份指南](https://askubuntu.com/questions/859333/how-to-install-libboost-version1-59-or-newer-on-ubuntu16-04)更新了boost，居然就，编译通过了，还正好是在我准备收拾东西离开实验室的时候。冲着这评论反手就是一个赞。

所以编译这个东西..在现在的我看来就是玄学，换台电脑/换个系统/换个环境，甚至偶尔网络断那么一分钟，都可能崩。

以上几个坑全当给后来人铺路吧。

## 运行效果

OpenMVG的初步使用可以参考上文编译部分给出的指南，对SfM过程的简单梳理可以参考[这篇文章](http://tombraiderjf.com/2018/01/03/SfM%E5%AE%9E%E7%8E%B0%E8%BF%87%E7%A8%8B%E5%88%86%E6%9E%90/)。

在获得OpenMVG的输出后，OpenMVS的用法可以直接套用官方的[Usage](https://github.com/cdcseacave/openMVS/wiki/Usage)，第一遍还是可以一步步实现一下，便于理解整个管线，后面就可以直接手写脚本来实现整个过程了。

目前在官方提供的数据集上运行后，纹理信息没有正常显示，但用另一个室外建筑的数据集试运行后，似乎没有太大问题。以下两图分别是官方数据集和[Pozzoveggiani](http://www.diegm.uniud.it/~fusiello/demo/samantha/)数据集上的运行效果，SfM方法均为增量式（Incremental）。

![](https://github.com/HusterHope/blogimage/raw/master/mvgmvs-2.jpg)

![](https://github.com/HusterHope/blogimage/raw/master/mvgmvs-3.jpg)

## 运行时间

配置：VMware虚拟机，Ubuntu16.04系统，分配内存8G，4核i7处理器，无GPU代码。

样例数据共11张图，分辨率为2832*2128，相机参数已知。由于数据质量较高，可用全局式SfM（Global SfM）重建，约12分钟完成全部流程，参数全为默认值。

时间占比：稠密点云构建最为为耗时，大约7分钟完成，表面重建和纹理映射各用两分钟左右。

Pozzoveggiani数据集共54张图，分辨率1024*768，相机参数未知。采用AKAZE_FLOAT作为特征提取方法，精度设置为HIGH，最近邻搜索方法设置为ANNL2，网格提取（简化）步骤的resolution-level设为4，其余参数保持不变。

使用增量式SfM方法得到重建结果，共耗时约30分钟。

## 全流程脚本

下面的脚本用于运行OpenMVG和OpenMVS进行全部重建，调用增量式SfM方法。（如需使用请修改头部全局变量）

```sh
#!/bin/bash
# Mvgmvs_incremental.sh
#   copyright (c) 2018 Lhp

IMAGE_PATH='/path-to-images'
CAMERA_SENSOR_WIDTH_DIRECTORY="/path-to-openMVG/src/openMVG/exif/sensor_width_database"
OPENMVG_SFM_PATH='/path-to-openMVG_build/software/SfM'
OPENMVG_BIN_PATH='/path-to-openMVG_build/Linux-x86_64-RELEASE'
OPENMVG_OUT_PATH=$IMAGE_PATH/out
MATCHES_DIR=$OPENMVG_OUT_PATH/matches
REC_DIR=$OPENMVG_OUT_PATH/reconstruction_incremental
OPENMVS_BIN_PATH='/path-to-openMVS_build/bin'
# FOCAL_LENGTH=1228.8

echo "--- Reconstruction Begin ---"
cd $IMAGE_PATH && mkdir out
cd $OPENMVG_OUT_PATH && mkdir matches reconstruction_incremental
cd
echo "--- OpenMVG Intrinsics analysis 1/12 ---"
# $OPENMVG_BIN_PATH/openMVG_main_SfMInit_ImageListing -i $IMAGE_PATH -f $FOCAL_LENGTH -o $MATCHES_DIR
$OPENMVG_BIN_PATH/openMVG_main_SfMInit_ImageListing -i $IMAGE_PATH -o $MATCHES_DIR -d $CAMERA_SENSOR_WIDTH_DIRECTORY/sensor_width_camera_database.txt

echo "--- OpenMVG Compute features 2/12 ---"
# $OPENMVG_BIN_PATH/openMVG_main_ComputeFeatures -i $MATCHES_DIR/sfm_data.json -o $MATCHES_DIR -m AKAZE_FLOAT -p HIGH
$OPENMVG_BIN_PATH/openMVG_main_ComputeFeatures -i $MATCHES_DIR/sfm_data.json -o $MATCHES_DIR

echo "--- OpenMVG Compute Matches 3/12 ---"
$OPENMVG_BIN_PATH/openMVG_main_ComputeMatches -i $MATCHES_DIR/sfm_data.json -o $MATCHES_DIR
# $OPENMVG_BIN_PATH/openMVG_main_ComputeMatches -i $MATCHES_DIR/sfm_data.json -o $MATCHES_DIR -n ANNL2

echo "--- OpenMVG Do Incremental reconstruction 4/12 ---"
$OPENMVG_BIN_PATH/openMVG_main_IncrementalSfM -i $MATCHES_DIR/sfm_data.json -m $MATCHES_DIR -o $REC_DIR

echo "--- OpenMVG Colorize Structure 5/12 ---"
$OPENMVG_BIN_PATH/openMVG_main_ComputeSfM_DataColor -i $REC_DIR/sfm_data.bin -o $REC_DIR colorized.ply

echo "--- OpenMVG Structure from Known Poses (robust triangulation) 6/12 ---"
$OPENMVG_BIN_PATH/openMVG_main_ComputeStructureFromKnownPoses -i $REC_DIR/sfm_data.bin -m $MATCHES_DIR -f $MATCHES_DIR matches.f.bin -o $REC_DIR robust.bin

echo "--- OpenMVG Compute SfM Data Color 7/12 ---"
$OPENMVG_BIN_PATH/openMVG_main_ComputeSfM_DataColor -i $REC_DIR/robust.bin -o $REC_DIR robust_colorized.ply

echo "--- OpenMVG to OpenMVS 8/12 ---"
cd $OPENMVG_OUT_PATH/reconstruction_incremental
$OPENMVG_BIN_PATH/openMVG_main_openMVG2openMVS -i sfm_data.bin -o scene.mvs

echo "--- OpenMVS: Dense Point-Cloud Reconstruction 9/12 ---"
$OPENMVS_BIN_PATH/DensifyPointCloud scene.mvs

echo "--- OpenMVS: Rough Mesh Reconstruction 10/12 ---"
$OPENMVS_BIN_PATH/ReconstructMesh scene_dense.mvs
# $OPENMVS_BIN_PATH/ReconstructMesh -d 4 scene_dense.mvs

echo "--- OpenMVS: Mesh Refinement 11/12 ---"
$OPENMVS_BIN_PATH/RefineMesh --resolution-level=4 scene_mesh.mvs
# $OPENMVS_BIN_PATH/RefineMesh scene_mesh.mvs

echo "--- OpenMVS: Mesh Texturing 12/12 ---"
$OPENMVS_BIN_PATH/TextureMesh scene_dense_mesh.mvs
cd

echo "--- Done ---"
```

## 相关参数

对脚本中的部分参数作些说明：

* -f $FOCAL_LENGTH

在第一步，如果图像的EXIF信息缺失或相机型号不在数据库列表中，则需要手动设置焦距，粗略的计算方法为：图像长边值乘以1.2，比如1024*768的图像数据集，FOCAL_LENGTH=1228.8

* -m AKAZE_FLOAT -p HIGH

在第二步，-m用于指定特征提取方法，默认为SIFT（不同特征提取方法之间的差异待调研），-p用于指定提取的精细度（分为NORMAL/HIGH/ULTRA，默认为NORMAL），程度越高耗费时间越长，对应重建的精细度也越高。由于图像本身内容细节有差别，这里的设置也不是越高越好。通常对特征较少的图像需要用更高的提取精度。例如针对Computer Vision Group - Faculty of Informatics - Technical University of Munich提供的[数据集](https://vision.in.tum.de/data/datasets/3dreconstruction)，我尝试重建其中的bird（21张，分辨率1024*768），设置到ULTRA精度明显获得了更好的效果，下图分别是HIGH和ULTRA的效果对比，重建时间分别为2分钟和8分钟。

![](https://github.com/HusterHope/blogimage/raw/master/mvgmvs-4.jpg)

![](https://github.com/HusterHope/blogimage/raw/master/mvgmvs-5.jpg)

* -n ANNL2

在第三步，用于指定最近邻搜索的方法（不同搜索方法的差异待调研）。

* --resolution-level=4

在第11步，这个参数的输入是为了降低计算成本，默认会更为耗时。

* 全局SfM

在第3步结尾添加`-g e`，将第4步调用的程序改为openMVG_main_GlobalSfM，并在第6步将`matches.f.bin`修改为`matches.e.bin`。对于小型非专业数据集，通常采用增量式SfM效果会更好。

## 小结

当玩具鸟被重建出来后，每一根羽毛都看得清清楚楚，就知道这几天的努力都没有白费，换系统/换管线/调参数试错的代价绝对是值得的。

在使用OpenMVG的过程中翻到了Jiang Fan同学博客上半年前的笔记，并获得一些交流，在此一并感谢。

三维重建是个深坑，希望这些整理的过程能够带给自己一些新的思考，能够多多少少排除一些「坑」。文中不足之处，欢迎同行和朋友们讨论交流。
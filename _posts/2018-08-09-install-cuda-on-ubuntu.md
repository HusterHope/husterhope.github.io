---
title: "Ubuntu16.04安装CUDA全流程笔记"
layout: post
categories: 解问题
tags:
  - CUDA
  - Linux
  - Ubuntu
  - Environment
---

需要高并行化计算的场景就需要用上GPU了，而CUDA环境的配置就成了绕不开的问题。

<!-- more -->

## 系统准备

由于用惯了虚拟机，在台式机上装双系统的分盘和启动操作又重新摸索了一遍，费了些时间。

工具：[UltraISO](https://cn.ultraiso.net/) + U盘

系统：Ubuntu16.04 ISO [Download](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/xenial/ubuntu-16.04.5-desktop-amd64.iso)

首先在Windows下的磁盘管理工具中，压缩一个可用空间较大的磁盘，给Linux系统分出一定空间，我设置为150G。划为4个区：（仅供参考）

* /：主分区 50G Ext4 /
* swap：逻辑分区 8G swap
* /boot：逻辑分区 1G /boot
* /home：逻辑分区 剩余所有空间 /home

（第一次分区装盘时给根目录空间留少了，不够装CUDA..又被某个CSDN博客给坑了一次

PS.分好区后选择的启动盘不要选分出来的小区域，一定要选完整的磁盘，这样系统启动信息才会写进磁盘头，从而被BIOS发现。

## CUDA安装流程

为了装CUDA折腾了三遍系统，最后按照官方文档的指点装好了。

（第一遍系统根目录空间不够 / 第二遍方法不当机子卡死 / 第三遍按照文档成功配置

[官方文档](https://docs.nvidia.com/cuda/pdf/CUDA_Installation_Guide_Linux.pdf)

主要步骤归纳如下：（这些步骤算是整理给自己看的，强烈建议具备一定英文理解能力的读者去看原文档，对新手十分友好）

### 准备阶段

* 查看nvidia显卡信息

```
sudo update-pciids
lspci | grep -i nvidia
```

大致输出如下：

```
01:00.0 VGA compatible controller: NVIDIA Corporation GP106 [GeForce GTX 1060 6GB] (rev a1)
01:00.1 Audio device: NVIDIA Corporation GP106 High Definition Audio Controller (rev a1)
```

可在[这个网址](https://developer.nvidia.com/cuda-gpus)找到支持CUDA的GPU型号（GeForce GTX 1060是支持的）

* 查看设备信息

```
uname -m && cat /etc/*release
```

* 查看GCC版本

```
gcc --version
gcc (Ubuntu 5.4.0-6ubuntu1~16.04.10) 5.4.0 20160609
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

### Linux内核版本号和头文件

> 最好在安装CUDA驱动程序前或更改内核版本时，手动确认安装正确版本的内核头文件和开发包。

系统核版本号：

```
uname -r
```

安装内核头文件

```
sudo apt-get install linux-headers-$(uname -r)
```

### 安装过程

* 下载：CUDA安装包[下载地址](http://developer.nvidia.com/cuda-downloads)。

安装包分为RPM/Deb和runfile包两种。Runfile支持更多种分发系统，但不会更新本地包管理系统，RPM/Deb只支持特定版本系统，但会与本地包管理系统留好接口，因此文档中更推荐使用RPM/Deb安装包。最后选择如下图所示：

![](https://github.com/HusterHope/blogimage/raw/master/CUDAinstall-1.png)

* MD5码校验

因为安装包比较大，下载完成后我们需要使用MD5码验证包的完整性：

```
md5sum cuda-repo-ubuntu1604-9-2-local_9.2.148-1_amd64.deb
```

将输出结果与[官方链接内容](https://developer.download.nvidia.com/compute/cuda/9.2/Prod2/docs/sidebar/md5sum-c.txt)对比，如完全吻合则代表安装包完整。以上述包为例，输出应该为：（第一个字段为MD5码）

```
957dcb32cb41336b18c5480d36c825fa cuda-repo-ubuntu1604-9-2-local_9.2.148-1_amd64.deb
```

> 冲突矩阵：如果之前系统中安装过CUDA，请参考文档中给出的冲突矩阵以确认是否发生版本冲突，并视情况删除一些包。本文略去。

* 安装

进入下载目录，依次执行：

```
sudo dpkg -i cuda-repo-ubuntu1604-9-2-local_9.2.148-1_amd64.deb
sudo apt-key add /var/cuda-repo-<version>/7fa2af80.pub
sudo apt-get update
sudo apt-get install cuda
```

> CUDA工具包包含创建，构造和运行CUDA应用程序所需的CUDA驱动程序和工具，以及库，头文件，CUDA示例源代码和其他资源。

文档4-6章为其它安装方法和跨平台设置，本文略去。

### 安装后的配置

* 环境变量

```
export PATH=/usr/local/cuda-9.2/bin${PATH:+:${PATH}}
```

* 启动nvidia-persistenced

查看状态：`systemctl status nvidia-persistenced`

启动：`sudo systemctl enable nvidia-persistenced`

### 验证安装是否成功

* 找驱动信息

```
cat /proc/driver/nvidia/version
```

* 编译样例程序

```
cd '/usr/local/cuda-9.2/samples'
sudo make
```

编译后的结果存放`/usr/local/cuda-9.2/samples/bin/x86_64/linux/release`目录下。

先后执行`deviceQuery`和`bandwidthTest`两个程序以验证安装是否成功。正确安装后的输出应有如下关键字：

deviceQuery:

![](https://github.com/HusterHope/blogimage/raw/master/CUDAinstall-2.png)

bandwidthTest:

![](https://github.com/HusterHope/blogimage/raw/master/CUDAinstall-3.png)

大功告成！

---

安装心得：对于自己从未接触过的库，不要随便尝试论坛/问答区开发者给出的快捷方法，一定要稍稍理解清楚每一步是在做什么，会对系统环境产生什么影响。官方文档比较靠谱的情况下，认真看下来效率也很高，并且基本能够一步到位。

按照官方文档这样安装CUDA后，OpenMVG和OpenMVS的流程（见上一篇文章）就已经能够检测到CUDA了，速度待测。


---
title: "三维重建系统踩坑记录（2）——Linux"
layout: post
categories: 解问题
tags:
  - Re3D
  - SfM
  - MVS
  - PointCloud
  - Qmake
  - Cmake
  - Linux
---

接着前一阵子记下的[上一篇](https://leohope.com/%E8%A7%A3%E9%97%AE%E9%A2%98/2018/03/22/multi-re3d-bugs/)文章写。

下列项目的具体地址可以在[调研记录](https://leohope.com/%E8%A7%A3%E9%97%AE%E9%A2%98/2018/03/06/compare-re3d-system/)里找到。

### Bundler

首先非常感人的是，Bundler终于在Ubuntu14.04下，通过编译源码跑通了。如上一篇文章所述，坑确实出在Sift文件上。

现在只需要在一个有若干图像的文件夹下执行`RunBundler.sh`即可。

至于查看效果，可以非常方便地使用`Meshlab`。

<!-- more -->

### Meshlab

这是一个可以直接从Ubuntu应用商店获取到的程序，使用起来还算稳定。当然界面跟大型三维建模软件（3dmax/Maya等）不能比。

因为多视图三维重建的四个关键步骤（稀疏点云->稠密点云->网格重建->纹理映射）里，Bundler完成了第一步，Meshlab只能完成第三、四步，第二步未完成，所以目前只能用Meshlab作为显示和渲染的工具。

### Colmap

因为Bundler的重建结果直观显示相机位置，无法观察视点密度，在减少视点后会明显出现模型破损。

正常渲染（顶点数量：29000左右）：

![](https://github.com/HusterHope/blogimage/raw/master/3drecon-bug-1.jpg)

减少部分视点后渲染（顶点数量：11000左右）：

![](https://github.com/HusterHope/blogimage/raw/master/3drecon-bug-2.jpg)

因此考虑使用Colmap作为重建工具，更清晰地显示相机位置，并重建稠密点云。

为了便于后期优化，放弃了使用Mac下现有的可执行程序，转而进入Ubuntu使用源码编译。

源码编译就是一切坑的开始。核心的坑在不熟悉的`Qmake`和`Cmake`。

按照项目原作者给出的编译方法，理论上会很顺畅。首先是Ubuntu下的环境：

```
sudo apt-get install \
    cmake \
    build-essential \
    libboost-all-dev \
    libeigen3-dev \
    libsuitesparse-dev \
    libfreeimage-dev \
    libgoogle-glog-dev \
    libgflags-dev \
    libglew-dev \
    qtbase5-dev \
    libqt5opengl5-dev
```

然后是[Ceres Solver](http://ceres-solver.org/)：

```
sudo apt-get install libatlas-base-dev libsuitesparse-dev
git clone https://ceres-solver.googlesource.com/ceres-solver
cd ceres-solver
mkdir build
cd build
cmake .. -DBUILD_TESTING=OFF -DBUILD_EXAMPLES=OFF
make
sudo make install
```

这个地方Google的源码链接不是很稳定（已科学上网依然服务器超时）`https://ceres-solver.googlesource.com/ceres-solver`，转至下载压缩包再编译，过程类似，不算坑。

坑在这部分：

```
cd path/to/colmap
mkdir build
cd build
cmake ..
make
sudo make install
```

进行到第四句，首先是提示Qmake版本低了，需要5.4版本以上的，而本地包版本是5.2.1。

然后就下了个5.7版本，并附带了Qt Creator。

接下来试了StackOverFlow上几乎所有的修改默认Qmake版本的方法，包括（不限于）：

* 修改`/usr/lib/`内的Qmake快捷方式，指向5.7版本；
* 修改系统环境变量`PATH`，默认优先搜索5.7版本`qmake`路径；
* 修改`makefile`里寻找`qmake`路径的段落；

均无效..

后来把`5.2.1`版本的Qmake删了，回溯找调用，原来是Cmake中的QtCore一直在找5.2.1版本的路径。就是下面这个文件和它的关联文件内容：

![](https://github.com/HusterHope/blogimage/raw/master/3drecon-bug-3.jpg)

但局部修改不一定有效，备份后试了一下，把整个Cmake环境搞崩了，放弃并恢复。

然后..想了一种更直接的方法，为什么不直接修改Colmap makefile里的Qt版本要求呢？

![](https://github.com/HusterHope/blogimage/raw/master/3drecon-bug-4.jpg)

然后我就把最低要求为5.4改成了上图中的5.2。。`cmake`终于不报错了。

本来想着这样的操作可以顺风顺水一路高歌，但`make`进度到86%左右时，还是出现了问题。

![](https://github.com/HusterHope/blogimage/raw/master/3drecon-bug-5.jpg)

胜利的曙光再次熄灭。

> marked override, but does not override

按照这样的关键字继续查，说法不一，尝试了改编译选项等解决方案，依然没有进展。所以..低于5.4版本的Qt还是不太能用吧。

倒是看到了一个项目里，用户和项目开发者就这个问题在issue里互怼：

![](https://github.com/HusterHope/blogimage/raw/master/3drecon-bug-6.jpg)

> 用户：这个Bug怎么办啊！
>
> 开发者：这种问题别提问了，看完文档再来问！
>
> 用户：我文档都看过了依然没解决啊！
>
> 开发者：你就不会去看看之前类似的问题么？
>
> 用户：...

以及..也有不少吐槽Cmake使用体验很差的（答案和评论区都有）：

![](https://github.com/HusterHope/blogimage/raw/master/3drecon-bug-7.jpg)

> hate this piece of garbage.

算是找了点共鸣..

至于源码编译Colmap的问题..继续放着吧，暂时就用Bundler+Colmap处理。

---

踩坑记录，依然，未完待续。
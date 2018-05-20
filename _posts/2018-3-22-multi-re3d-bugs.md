---
title: "三维重建系统踩坑记录（1）——MacOS"
layout: post
categories: 解问题
tags:
  - Re3D
  - SfM
  - MVS
  - PointCloud
  - Git
  - Github
  - MacOS
---

对多视图三维重建的理论方面有了初步认识之后，就可以开始尝试运行完整的三维重建系统管线了。

然而这个过程充满坎坷，到写下这个记录为止，依然没有找到合适的解决方案。按照能够想起来的过程整理一下（估计也没人会碰到这些问题了，碰到了解决起来应该也比我快吧orz

<!-- more -->

---

首先是用哪个系统的问题。

根据之前的[调研记录](http://northfish.space/%E8%A7%A3%E9%97%AE%E9%A2%98/2018/03/06/compare-re3d-system/)，能够顺利完成第一步（Structure From Motion）的系统有：

* Bundler
* Colmap
* MVE
* Theia
* VisualSfM
* OpenMVG

### VisualSfM

```
踩坑时间：半天

编译情况：未尝试
```

不开源，直接舍弃。

### Colmap

```
踩坑时间：半天

编译情况：未尝试
```

去年跟我做差不多课题的一个师兄就选的这个框架，然而这个系统的核心部分是基于GPU的..文档很详细，如果可以的话，先搁置一下再试。

### Bundler

```
踩坑时间：1天

编译情况：通过

运行情况：关键模块未能正常运行
```

文档不怎么长，按作者的要求，先后安装：

* 源码：https://github.com/snavely/bundler_sfm
* 特征提取器SIFT：http://www.cs.ubc.ca/~lowe/keypoints/
* 图像头读取器jhead：http://www.sentex.net/~mwandel/jhead/
* ImageMagick库：mac直接用`brew install imagemagick`（需先安装[Homebrew](https://brew.sh/)）

> 为什么要用Homebrew？一方面它用来安装新库时比较方便，不需要手动下载然后编译测试，另一方面它能够在安装后将环境变量一起配置好，比如库文件的存放目录、链接关系等，总之强烈推荐。

然后就可以在工程主目录下直接`make`了。

中间似乎还有报错，应该是缺库之类，按照需求补齐。还有`error: invalid value 'f95' in '-x f95'`这种类型的报错，可以试试下面的`make`语句（根据系统内gcc的版本适当修改，我用的gcc-5编译通过了）：

```
make FC=gfortran CC=gcc-6 CXX=g++-6
```

然后设置`libANN`的路径：

```
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/bundler/bin
```

这些都没有问题了之后，终于可以运行Bundler脚本了！

> The easiest way to start using Bundler is to either use the provided RunBundler.sh bash script.

找到插件里给出的样例目录，进这个目录执行脚本。

Bundler首先提取了每个图像的参数并记录下来。

![](http://ohn6qfqhe.bkt.clouddn.com/sfmlbug-1.jpeg)

出现了问题：

![](http://ohn6qfqhe.bkt.clouddn.com/sfmlbug-2.jpeg)

特征提取器SIFT的可执行文件不能正常运行..原来是只支持Linux和Win..

然后找Github该项目的讨论区：https://github.com/snavely/bundler_sfm/issues/52

也有人询问类似的问题：能否换个特征提取器？

作者说当然可以，但不少输入和输出的数据结构要改（懂的..

然后作者在回复里还给出了下面这个系统：TheiaSfM，可以不使用SIFT。

![](http://ohn6qfqhe.bkt.clouddn.com/sfmlbug-3.jpeg)

刚好之前也调研过，那就试一试！

### Theia

```
踩坑时间：2天

编译情况：通过

运行情况：正常模块比例 93/101，整个程序管线崩了。
```

有了前面的经验，这次先把环境和需要的库全都装好。

* Cmake（这个应该很常见）
* 装完Cmake装eigen3：`brew install eigen`
* OpenImageIO：`brew install openimageio`
* ceres solver：`brew install ceres-solver`
* Google-glog：`brew install glog`，会打包安装`gflag`

> Make sure all of these libraries are installed properly before proceeding.

没错我确认这些都装好了！继续！

首先还是把库从Github上clone到本地，然后创建`theia-build`文件夹（存放编译产生的文件），进入这个文件夹，开始cmake。

然后..报错，OpenImageIO的问题（以下简称为*OiIO*）。

然后在Theia的讨论区找到了问题，也就是Mac OS X下，用Homebrew自动安装的OiiO是最新版，不太能用。

![](http://ohn6qfqhe.bkt.clouddn.com/sfmlbug-4.jpeg)

那退个旧版吧..查查怎么用homebrew把这个库退到旧版：

https://stackoverflow.com/questions/3987683/homebrew-install-specific-version-of-formula?rq=1

这个问题里的回答非常清晰易懂，大概给出了这些方案：

* 检查本机是否安装了旧版，如果有，用`brew switch`语句来切换版本。

对我来说并不存在OiIO的旧版，因为这个库以前没用过。故舍弃此方案。

* 查找目前homebrew上有哪些版本可用（`brew search`命令）。如果有多个版本，可以直接指定版本安装。

一般来说存在多版本的库都是常用的大型库，查了这个OiIO，没有其他版本。舍弃。

* 用`brew versions`指令查看git仓库里的旧版。

尝试后：versions指令已经在2015年被弃用..

其他与versions指令有关的解决方案，如：`brew tap homebrew/boneyard`等指令全部尝试..boneyard也已经迁移，无效..舍弃versions

在发现Google的解决方案全部过时或者不满足使用条件后，只好自己动手。

先找到程序包，从源码开始编译：https://github.com/OpenImageIO/oiio/releases/tag/Release-1.7.17

过程和跑其他程序没多大差别，但是..Theia找不到编译出来的程序，即使放到系统目录也不行。

应该是环境变量设置的问题了..但这个过程很麻烦啊，还是试着用homebrew吧。

百般折腾之后，我将OiIO编译出的结果用Homebrew的存储方式记录，目录格式为`usr/local/Cellar/openimageio/1.7.17`。

用上面的`brew switch`指令，成功将这个OiIO库切到了旧版本...终于可以重新编译Theia了。

又发现缺少google test库和openMP库，继续装，重新编译。openMP似乎一直有问题，

> OpenMP是支持跨平台共享内存方式的多线程并发的编程API。

既然编译没有报error，那就先不管这个库的依赖了，继续。

来到了`make -j4`这一步，等待数分钟，未报错，但报了一大堆warning。

`make test`，这是作者事先写好的单元测试脚本，用来验证模块的功能是否正常，共101个单元测试，我执行之后报了8个错。

![](http://ohn6qfqhe.bkt.clouddn.com/sfmlbug-5.jpeg)

说多不多，说少不少，也不知道这些具体有啥影响，作者在文档里很良心地留了一句话：

> You probably do not need to worry about the failing tests. Email the mailing list if you are unsure about failing tests.
>
> 如果不懂报的是什么错，可以去这个工程对应的Google网上论坛问问。

于是来到论坛找作者，用test作为搜索关键字。发现了另一个跟我报错差不多的人（甚至他还多了几个错）：

![](http://ohn6qfqhe.bkt.clouddn.com/sfmlbug-6.jpeg)

这是去年11月底的帖子，当时作者给的引导是执行一个脚本看看具体报什么错：

```
./bin/twoview_info_test --logtostderr --v=2
```

然后那个人也没回音了，我就按作者的指引试了下，将记录反馈到一个新帖子里回复作者。

等了一晚上后得到作者回复：

![](http://ohn6qfqhe.bkt.clouddn.com/sfmlbug-7.jpeg)

> 你确定同步了Master分支？我也不知道为什么这些简单的单元测试会报出这么奇怪的段错误。如果继续安装下去，整个流程肯定是跑不通的。

让人哭笑不得的是..作者居然怀疑我有没有用他的源码？我..只好表达感谢换个坑挖吧..

（感谢作者能够告诉我这些错会导致最后的程序崩掉..不然又要试后续步骤了



（未完待续）

---

### 小结

本文深度尝试的系统：Bundler、Theia

简单试探的系统：Colmap、VisualSfM

在Mac下差不多可以舍弃的系统：Bundler、Theia、Colmap、VisualSfM

下一步继续尝试的系统：OpenMVG，MVE



欢迎成功运行过这些系统的同学（或者同样踩坑中的小伙伴）留言交流。
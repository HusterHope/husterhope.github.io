---
title: "FFmpeg交叉编译了解一下"
layout: post
categories: 解问题
tags:
  - FFmpeg
  - Ubuntu
  - Cmake
  - Linux
---

学习一下用linux交叉编译Windows项目，ffmpeg具体后续如何使用暂时不管，首先需要在linux下正常编译通过。

> 项目地址：https://github.com/rdp/ffmpeg-windows-build-helpers

<!-- more -->

然后按照文档的指示：

```
$ git clone https://github.com/rdp/ffmpeg-windows-build-helpers.git
$ cd ffmpeg-windows-build-helpers
$ ./cross_compile_ffmpeg.sh
```

（项目说明里提示至少留出4G的磁盘空间，事实证明用了不止5G）

理论上只需要这三步，不过师兄说这个项目bug很多，因此执行第三步的时候可能会踩上一天的坑。

那就干脆踩一个坑记一笔，这样比较容易复现。

第一遍跑的时候会让你安装外部依赖库，按提示用apt-get命令即可。

然后选择是否需要安装非商用库，这里选【否】。

32位/64位：都进行。

然后跑大概30分钟（根据网速可能会不同，仅供参考），会首先看到一个提示：

```
MinGW-w64 has been built without errors.
```

这说明第一个模块还比较顺利，后面开始出坑，总的来说还没有出现特别难以解决的情况，只需不断google -> test -> google -> test。总体上错误分为三类：

### 从外网下载源码安装包时响应时间过长

这一点即使科学上网也不能完全解决，因此有些访问超时的库只好找镜像下载。这些库包括：

* libwebp
* libsamplerate
* libvpx
* aomedia

改动方式是从github或者google搜索的结果中找到镜像包，把`cross_compile_ffmpeg.sh`中的对应链接改成github链接或者其他镜像包链接即可。例如`libvpx`的修改过程为：

```
将： https://chromium.googlesource.com/webm/libvpx.git
改为： https://github.com/webmproject/libvpx.git
```

其余包的处理方法类似。不过像`aomedia`包转到github下载后会出现包名称与脚本名称不符的情况，需在脚本后重命名：

```
mv aom_git aom  # rename aom_git to aom for further operations
```

### Ubuntu14.04上有些依赖库版本过低

脚本运行中出现如下错误：

```
'/home/lhp/Documents/ffmpeg-windows-build-helpers/sandbox/win32/libsndfile_git/configure.ac' 327

configure.ac:54: error: macro PKG_INSTALLDIR is not defined; is a m4 file missing?
```

尝试解决方案：手动升级`pkg-config`版本（Ubuntu14.04最高自动支持0.26版，解决方案说升级到0.27版以上），前往https://pkg-config.freedesktop.org/releases/下载新版本。

然后根据`ffmpeg-windows-build-helpers`的issue页面（[#issue244](https://github.com/rdp/ffmpeg-windows-build-helpers/issues/244)），有人给出如下环境配置需求：

```
updated :

automake to 1.15.1

autoconf to 2.69

libtool to 2.4.6
```

手动找到这些库下载，解压。按照对应的`INSTALL`文本说明操作，主要步骤应该如下：

```
cd automake
sudo ./configure
sudo make
sudo make install
```

### 奇怪的其他问题

```
AX_SPLIT_VERSION command not found 
```

同样问题的用户表示可以不使用`libtesseract `这个库，即删去脚本`cross_compile_ffmpeg.sh`中1703行的`--enable-libtesseract`，并注释掉1900行的`build_libtesseract`。

解决这部分问题，就能看到32位编译成功的提示（不过似乎因为没有安装7z所以压缩失败了）：

```
Done! You will find 32-bit static binaries in /home/lhp/Documents/ffmpeg-windows-build-helpers/sandbox/win32/ffmpeg_git.
./cross_compile_ffmpeg.sh: line 1797: 7z: command not found
You will find redistributable archives in /home/lhp/Documents/ffmpeg-windows-build-helpers/sandbox/redist.
```

然后开始编译64位，不出意外的话也能正常完成。

```
Done! You will find 64-bit static binaries in /home/lhp/Documents/ffmpeg-windows-build-helpers/sandbox/win64/ffmpeg_git.
./cross_compile_ffmpeg.sh: line 1797: 7z: command not found
You will find redistributable archives in /home/lhp/Documents/ffmpeg-windows-build-helpers/sandbox/redist.
```

---

顺带一提..编译这种大型库的时候最好找点事情做..不要玩手机，会停不下来（个人教训

最好刷点跟代码没啥关系的书。比如我就在编译的时间内把《孙子兵法》集注版刷了一半，边看古书边看代码，诶挺有意思的。

> 治众如治寡，分数是也。

完。
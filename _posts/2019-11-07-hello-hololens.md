---
title: "【Hololens】调试+安装环境配置"
layout: post
categories: 解问题
tags:
  - MR
  - Hololens
  - Unity
  - Environment
---

<!-- more -->

由于项目需求开始接手Hololens开发。

今年上半年微软官方就开始了Hololens2的预售，然而等了大半年，看上去国内是拿不到货了。所以很遗憾，9012年了仍然只能摸到Hololens1代开发版设备。

目前微软官方提供的Hololens教程不断突出Hololens 2的相关内容，使得原本1代的很多文档混在其中难以辨别。加上Hololens从上市至今也过去了3年，配套的开发环境从Unity到Visual Studio 都发生了很大改变，特别是Unity每年都在更新，而前后版本的兼容性并不好，通常解决bug的最好方式就是安装特定版本的Unity再进行编译。

## 环境需求

### PC端

系统：Windows 10

Unity版本：2018.4.12f1(64-bit) 。该版本Unity有长期支持（LTS）标志，相对其他版本较为稳定

注意给添加几个功能模块：UWP Build Support（IL2CPP和.NET），Windows Build Support，在Unity Hub中应该能够看到这样的显示：

![](https://github.com/HusterHope/blogimage/raw/master/20191107-1.png)

Visual Studio版本：Community 2017/2019均可

安装时需要附加“使用C++的桌面开发”和“通用Windows平台开发”两个模块。

### Hololens端

* 从Windows Store中安装“Holographic remoting”（用以和PC端建立连接，在Unity调试时用到）
* 在系统设置里开启开发者模式，并与PC建立配对（生成PIN，在Visual Studio生成应用程序并部署到Hololens上时需要用到）

## 测试和安装过程

### 调试

类似于直接使用HTC Vive这样的VR头显设备，MR开发过程中的调试可以直接使用Unity的Holographic Emulation功能进行无线连接（Unity2018.4版本中，对应功能位于Window/XR目录下），在Hololens中打开应用Holographic remoting，记录IP地址，输入Unity，实现连接。这样在Unity中运行场景即可同步到Hololens中。

Notes：

* Unity - Project Settings
  * 【Player Settings】需要切换到UWP的页卡，并执行以下两步：在XR Settings中勾选Virtual Reality Supported，并导入Windows Mixed Reality SDK（通常会自动添加）；在Other Settings里使用.NET配置（Scripting Runtime Version为.NET 4.x Equivalent，Scripting Backend为.NET，Api Compatibility Level为.NET 4.x）。
  * 【Quality】需要将Windows platform下面的Level选为Very low，以确保Hololens中渲染的实时性。
* 其他关键细节请参考原始教程。

### 安装

如果需要将应用打包成一个可以在Hololens中独立运行的App，则需要配合Visual Studio进行编译。首先在Unity中调整Build Settings。每个选项的具体参数如下图所示：

![](https://github.com/HusterHope/blogimage/raw/master/20191107-2.png)

接下来切换到Visual Studio，注意版本号应该与上一步的Visual Studio Version选项相同。

在VS的顶部工具栏分别选择Release-x86-Remote Machine，初次配置Remote Machine时会弹出窗口，输入对应的IP地址和PIN与Hololens配对（注意在配环境时Hololens端打开的【开发者模式】，这里需要用到）。

连接成功后即可依次点击【Debug】-【Start without debugging(Ctrl+F5)】，等待VS将应用部署到Hololens上。部署完成后就能在Hololens中找到对应的应用程序了。

---

整个过程找对教程跑通后其实并不复杂，主要踩坑原因是上手方式不太对。第一次直接参考开源工程[GalaxyExplorer]( https://github.com/microsoft/GalaxyExplorer )的说明文档进行编译会出现很多问题，包括环境的关联也比较乱。

后来找到微软官方教程[MR Basics 100]( https://docs.microsoft.com/en-us/windows/mixed-reality/holograms-100)逐步排坑，终于搞定了直接用Unity调试Hololens的操作，以及用Visual Studio将应用安装到Hololens的应用列表的方法，留档备忘。打通Unity的接口之后，MR和VR的开发就能够互通了。

PS. Galaxy Explorer真的是个好工程。

后续学习路线：

微软官方Hololens教程：[HoloLens (1st gen) and immersive headset tutorials - MR Basics 101](https://docs.microsoft.com/en-us/windows/mixed-reality/holograms-101)

Github上的MR开发工具包：[MRTK](https://github.com/microsoft/MixedRealityToolkit-Unity)


---
title: "【Kinect初体验】LiveScan3D-Hololens工程使用"
layout: post
categories: 解问题
tags:
  - Kinect
  - Unity
  - Re3D
  - SfM
  - VBR
  - Network
  - MR
  - SingleView
---

功能：将多个[Microsoft Kinect 2](https://developer.microsoft.com/en-us/windows/kinect)设备分别连接到多个客户端（个人电脑），服务器（某一台电脑端）将客户端捕捉到的这些多视角RGB-D数据重建成三维点云，并在Unity中实时显示，还能够通过[Hololens](https://www.microsoft.com/en-us/hololens)设备实时观察。

[项目地址](https://github.com/MarekKowalski/LiveScan3D-Hololens)

使用设备：Windows 10系统的台式机\*1（将其同时作为客户端和服务器），Kinect\*1。

背景：第一次接触Kinect..

结果：全程按照项目文档顺利实现。

<!-- more -->

---

按照LiveScan3D-Hololens的文档，为了实现该项目的效果，首先需要一个更早的工程作为支撑——[LiveScan3D](https://github.com/MarekKowalski/LiveScan3D)。

这个工程的文档可以说是很详尽了（[文档地址](https://github.com/MarekKowalski/LiveScan3D/blob/master/docs/manual.pdf)）。基本步骤如下：

安装Kinect2.0 SDK->连接Kinect->启动服务器->打开客户端->连接服务器->将标定图片打印下来，放入摄像机视野内->相机标定

客户端界面如下，白板上固定的纸就是标定点，帧速率约为24，为实时传输。

![](http://ohn6qfqhe.bkt.clouddn.com/kinect-unity-1.png)

客户端显示的深度图：

![](http://ohn6qfqhe.bkt.clouddn.com/kinect-unity-2.png)

服务器实时接收的点云数据如下：（可以看到帧速率有所下降）

![](http://ohn6qfqhe.bkt.clouddn.com/kinect-unity-3.png)

可以通过鼠标和键盘的简单交互调整视角：

![](http://ohn6qfqhe.bkt.clouddn.com/kinect-unity-4.png)

以上过程对于我这种小白用户来说非常友好。

然后返回LiveScan3D-Hololens，按照该工程的描述，只需在服务器上运行Unity工程即可看到效果。

初步效果稍有问题，将相机沿Z轴负方向稍作拉伸即可看到效果，单相机单客户端单服务器的情况下，帧速率约为20。下图为Unity中的效果：

![](http://ohn6qfqhe.bkt.clouddn.com/kinect-unity-5.png)

对于放到Unity里能够正常显示的场景，借助Unity引擎强大的硬件支持，就可以转化成VR/AR/MR应用了，虽然点云还比较稀疏，也存在不少噪声点，但不影响继续探索和开发的动力。

现在对这种能够按照文档跑通而碰不上bug的项目感到非常不习惯（

只能说，项目原作者真心666
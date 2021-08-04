---
title: "Unity移动端VR场景简易优化笔记"
layout: post
categories: 解问题
tags:
  - Unity
  - Performance optimization
---

<!-- more -->

平台：Pico Neo 3，Unity2020.3LTS

待优化资源： [Mars Environment | 3D 环境 | Unity Asset Store](https://assetstore.unity.com/packages/3d/environments/mars-environment-42564) 

目标：在Pico端快速部署并流畅运行火星场景（fps>60）

第一遍直接部署场景时移动端出现黑屏，发现犯了一个很低级的错误，没有设置移动端Graphics API，默认将Vulkan放在了列表里。

修改后运行，FPS约等于1，直接卡成PPT。

参考一些官方和常规的操作来进行配置，比如：

 [6 性能优化指南 — PicoVR Unity SDK 0.1 文档](http://sdk.picovr.com/docs/sdk/cn/chapter_six.html) 

在Pico官方提供的性能优化指南里，对照6.5部分完成所有推荐配置，并将移动端的Quality设置为Very Low。FPS在5左右，仍然不能看。

如果走到场景的最边缘，会发现FPS能稳定到72，证明该资源的天空盒与环境光并不太吃资源。剩下的核心优化点就在地形上了。

地形方面看到有文章提到可以用MTE这个插件进行优化，即先将Unity的Terrain转换成三维模型，再对模型做减面操作。相比原始的Terrain会有明显性能提升（宣称）。

 [Mesh Terrain Editor Free | 地形 | Unity Asset Store](https://assetstore.unity.com/packages/tools/terrain/mesh-terrain-editor-free-67758) 

不过由于免费版插件不太支持这个规模的地形，也就搁置了。

后面参考其他通用优化方案也进行了一定调整，在调整一些纹理图大小后，FPS上到了15左右。

 [Unity 性能优化总结(适合初学者)_胖胖的橘猫君-CSDN博客_unity性能优化](https://blog.csdn.net/qq_35361471/article/details/82825395) 

根据之前和本次优化的一点经验，PicoVR这样的移动端，最吃资源的部分在纹理图大小这块，基本超过2K*2K的贴图就会明显卡顿。

针对地形，我们参考Unity文档提供的参数说明。

 [地形设置 (Terrain Settings) - Unity 手册](https://docs.unity.cn/cn/2018.4/Manual/terrain-OtherSettings.html) 

调整一些参数后发现，里面最核心的两个值是Pixel Error和Base Map Dist.

* Pixel Error：地形贴图（如高度贴图和纹理）与生成的地形之间的映射精度。值越高表示精度越低，但渲染开销也越低。 
* Base Map Dist. Unity 以全分辨率显示地形纹理的最大距离。超过此距离后，系统将使用较低分辨率的合成图像来提高效率。 

由于整个地形很大（1500m*1500m），将第二个参数调低后，FPS出现了质的飞跃，基本也就完成了基本要求。
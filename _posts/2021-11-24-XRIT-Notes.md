---
title: "XR Interaction Toolkit Notes"
layout: post
categories: 做笔记
tags:
  - VR
  - UnityXR
---

Unity XR席卷各个设备开发，有必要对官方文档学习一个。

 [XR Interaction Toolkit -XR Interaction Toolkit - 0.9.4-preview (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.xr.interaction.toolkit@0.9/manual/index.html) 

<!-- more -->

更新记录：

2021-11-24 V0.1

2021-12-2 更新Locomotion部分 全文更新完毕

## 简介

核心：Interactor、Interactable两类组件，以及连接这两类组件的管理器Interaction Manager。

> Interactor：可以直观理解为手柄、射线等用于选择或移动场景中其他物体的组件。
>
> Interactable：可以直观理解为场景中能够被操纵的对象，比如被抓取、点击、投掷等。

主要特点：

* 跨平台响应XR控制器输入；
* 基本的物体悬停(hover)、选择和抓取；
* 通过XR控制器的触觉反馈；
* UI画布
* VR camera rig，支持Stationary/Room-Scale(在Locomotion章节展开)

## Interactor和Interactable

### Interactor

支持多种形态，例如Unity Physics Ray Casting

为了响应来自用户的输入，Interactor需要绑定一个控制器（例如手柄、头显等）

* XR Ray Interactor可以世界与Unity UI元件交互（勾选Enable Interaction with UI GameObjects）

### Interactable

* 支持默认球类碰撞体检测，但其他多面体类型的碰撞检测效果更好

### 通用特性

* 默认状态下，所有Interactable能够被所有Interactor所影响。
* 交互状态
  * 悬停（Hover）：一种潜在的交互意向，但没有真正触发。类似于鼠标滑过按钮的状态；（进入：OnHoverEnter/离开：OnHoverExit.）
  * 选择（Select）：按钮或触发器被选中（进入：OnSelectEnter/离开：OnSelectExit）；
  * 激活（Activate）：对选中的物体进行进一步交互操作，例如枪支开火（触发：OnActivate）；
* 交互层级（InteractionLayerMask）：可以设定不同层级的Interactor和Interactable，让层间不相互干扰。
  * “This can be useful in multiplayer scenarios to confine each player's or faction's freedom of movement to their own areas.”
* 两类组件均含有事件回调功能，可在触发不同交互状态时调用其他GameObject的方法。

## UI画布

* 初始化模板可以选用GameObject->XR->UI Canvas；
* EventSystem：每一个EventSystem组件需要通过一个输入模块（InputModule）来处理输入；
* 用户只能和WorldSpace模式下的UI元素交互；
* 新组件Tracked Device Graphic Raycaster，暂未详细了解；

## XR交互刷新循环

Interaction Manager作为管理刷新的核心，大致经历以下步骤：

* 遍历所有Interactor和Interactable，检索存在的hover或selection状态
* 为所有Interactor生成目前悬停和选择的有效物体列表
* 对每个Interactor和Interactable，检查当前交互状态是否持续
* 交互事件结束后，对失效物体从列表中清除
* 重新检索交互状态，开启新一轮循环

## 穿插：Pico Neo 3 Pro Eye 眼动追踪测试

记录XRIT的主要目的是使用眼动设备做下一步实验，于是先解决这部分的demo测试问题。

学习轨迹：

在Pico Neo3上跑通基于UnityXR SDK的Eye-Tracking Demo【1】，Clone，打开工程，按Pico官方文档【2】配置。

（Pico这一年换了三个版本SDK，终于还是跟Oculus一样向UnityXR靠拢了，默默许愿之后别大改。。）

发现工程里每个场景的XR Rig都有一个脚本找不到，应该是对应了PXR_Manager，手动补上。

如果比较在意效果，也可以按上述文档【5.1.2 手柄使用说明】，顺带补上手柄模型。

测试时排除了无线串流【3】的方式，因为目前pico还不支持用SteamVR测试交互功能【4】，无限串流目前只能作为观察PC端运行效果的工具，无法引入手柄端的交互。

改代码时发现用了单例模式，复习了一波面向对象。。然后撞进了Shader入门书作者LeleFeng的CSDN【5】（还是要学习大佬们的归纳整理思路

以及边看代码边补了一点Unity中的资源管理和序列化【6】 ，这部分就暂时这样了。

## Locomotion

组件列表：

* XR Rig
* Locomotion
* Teleportation
* Snap Turn

### 架构

Locomotion请求流程：

1. Locomotion Provider检查Locomotion System是否繁忙；（若为是，则直接返回）
2. 若为否，则前者申请对后者的独占访问权限；
3. 若申请成功，前者移动XR Rig；
4. 移动结束后，Locomotion Provider释放访问权限。

可以把Locomotion System看作Locomotion Provider访问XR Rig的仲裁者。

Locomotion Provider包含Teleportation和Snap Turn两类Provider，前者是远距离瞬间传送，后者是原地视角旋转。

Teleportation：

* Anchor：用户预定义的传送点和角度
* Area：一个能够被任意TP的区域平面

-More Info-

入坑过程里，主要参考Manual【7】学习概念，按照Unity Learn的教程【8】完成环境配置细节。

基础模块配置参考【8】，这与【7】的前三节基本一致，但【7】描述的细节不太到位，还有些截图明显不符合当前版本，初看容易晕。

### 线段类型

【7】中还整理了一些Raycast线段类型、视觉效果的介绍，可以当成手柄射线与外界交互之间的形态。

* 直线：最基本的射线形态，就一个变量`Max Raycast Distance`，如同字面意思。
* 抛物线（Projectile Curve）：
  * 将Raycast作为抛物轨迹的模拟，给定初速度、重力加速度，即可计算完整的轨迹
  * `Additional Flight Time`决定了曲线总长度
  * `Sample Frequency`采样频率，越高则曲线越平滑
  * Teleportation的场景下，推荐使用抛物线
* 贝塞尔曲线：参数类似于抛物线，没太多可说的

同样，线段视觉效果里的参数属性文档也介绍的比较易懂，不再展开。



## Reference

【1】 [picoxr/Eye-Tracking-UnityXR (github.com)](https://github.com/picoxr/Eye-Tracking-UnityXR) 

【2】 [2 SDK配置说明 — Unity XR Platform SDK 0.1 文档 (picovr.com)](https://sdk.picovr.com/docs/XRPlatformSDK/Unity/cn/chapter_two.html) 

【3】 [Pico Neo 2 a great headset for develop VR apps in Unity3d (wixsite.com)](https://robertocolonello.wixsite.com/vrandbeyond-1/post/pico-neo-2-a-great-headset-for-develop-vr-apps-in-unity3d) 

【4】[Pico Neo3在使用Unity XR SDK进行无线串流并导入SteamVR SDK的问题 - Pico Developer Answers (pico-interactive.com)](https://devanswers.pico-interactive.com/index.php?qa=1808&qa_1=pico-neo3在使用unity-xr-sdk进行无线串流并导入steamvr-sdk的问题&show=1852#a1852) 

【5】[面向对象_candycat-CSDN博客](https://blog.csdn.net/candycat1992/category_1250639.html) 

【6】[玩转Unity资源，对象和序列化(上) | qingqing.zhao's blog (zhaoqingqing.github.io)](https://zhaoqingqing.github.io/2016/08/30/unity_assets_object_serialization_1.html)

【7】 [Locomotion | XR Interaction Toolkit | 0.9.4-preview (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.xr.interaction.toolkit@0.9/manual/locomotion.html) 

【8】 [Locomotion and Teleportation in the XR Interaction Toolkit - Unity Learn](https://learn.unity.com/tutorial/locomotion-and-teleportation-in-the-xr-interaction-toolkit#5f91d116edbc2a001ffb3ba2) 
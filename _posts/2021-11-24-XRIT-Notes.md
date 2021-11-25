---
title: "XR Interaction Toolkit Notes"
layout: post
categories: 做笔记
---

Unity XR席卷各个设备开发，有必要对官方文档学习一个。

 [XR Interaction Toolkit -XR Interaction Toolkit - 0.9.4-preview (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.xr.interaction.toolkit@0.9/manual/index.html) 

<!-- more -->

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

## Locomotion

组件列表：

* XR Rig
* Locomotion
* Teleportation
* Snap Turn



-TODO-

teleportation测试

Eyetracking测试


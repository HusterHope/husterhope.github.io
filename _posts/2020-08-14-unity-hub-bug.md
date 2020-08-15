---
title: "Debug｜Unity安卓工程闪退"
layout: post
categories: 解问题
tags:
  - Unity
  - Android
---

<!-- more -->

自从Unity强行推出Unity Hub作为版本管理和项目控制的入口后，不仅许可证需要频繁更新，偶尔神奇的小bug也出其不意。

最近在整的一个Unity安卓平台工程，突然间通过Hub的入口就打不开了，具体表现为，**每次点击项目后，Unity Hub先消失，并弹出一个Unity启动界面，随后Unity Hub重新出现，而Unity并没有被正常打开。**

既然这样，直接想法是不通过Unity Hub，而是手动打开所需的Unity Scene。

成功打开，但得到了这样的错误提示：

> Your license does not cover Android publishing.

需要强行切换到Windows平台。行吧，先切过去看看。

再后面，针对上面的错误提示寻找解决方案，包括：

* 重新登陆账号（打开Unity->右上角Account->Sign out->关闭Unity->再次打开->右上角Account登陆）
* 重新激活许可证（Unity Hub中的许可证管理；还可通过删除`C:\ProgramData\Unity`文件夹后重新申请许可证）
* 重装Unity Hub
* 重装Unity对应版本
* 彻底卸载并重装全套Unity
* 以及时不时各种重启

上述方法全部没有解决问题，不过在重装后，问题出现了变化。

> Unity已存在，您无法加载相同的版本。

然后在[CSDN的一个博文](https://blog.csdn.net/qq_43413788/article/details/107792882)里搜到了奇怪的解决方案，为了突出这个问题究竟有多么无聊，我截图贴一下描述：

![](https://github.com/HusterHope/blogimage/raw/master/20200814-1.png)

于是正是因为更换了网络连接方式，导致Unity-安卓工程无法正常打开。

稍记一笔，吸取教训。
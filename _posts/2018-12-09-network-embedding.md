---
title: "【计算机网络】Network Embeddings相关概念"
layout: post
tags:
  - ParallelProcessing
  - Network
categories: 做笔记
---

<!-- more -->

* 网络嵌入（Network Embeddings）

作用：方便将现有算法移植到新网络环境（拓扑）中，我们称新网络“模拟”了旧网络。

参数：Dilation（扩张）/Congestion（拥塞）/Load factor（负载系数）/Expansion（网络扩张）

* Dilation

>  Longest path onto which any given edge is mapped.

扩张，任意一个给定边，嵌入新网络后的最大长度。

通常用来衡量间接数据通信造成的减速。

* Congestion

> Maximum number of edges mapped onto one edge.

拥塞，嵌入新网络后映射到同一条边的最大边数。

当旧结构中的特定边需要同时传输多条信息时，衡量潜在的降速问题。

* Load Factor

> Maximum number of nodes mapped onto one node.

负载系数，映射到同一结点的最大节点数。

当旧结构中的特定结点需要处理多项任务时，衡量潜在的降速问题。

* Expansion

> Ratio of the number of nodes in the two graphs.

网络扩张：新旧网络节点数对比。

该参数与另外三个参数紧密相关。

---

例：将一棵7个节点的二叉树嵌入平面网格。

![](https://github.com/HusterHope/blogimage/raw/master/20181209-1.jpg)

对应的Dilation / Congestion / Load factor / Expansion 分别为：

![](https://github.com/HusterHope/blogimage/raw/master/20181209-2.jpg)

---

参考资料：Behrooz Parhami. Introduction to Parallel Processing.
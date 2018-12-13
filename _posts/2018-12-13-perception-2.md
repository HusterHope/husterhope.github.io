---
title: "【Perception-2】投影变换"
layout: post
categories: 做笔记
tags:
  - Math
  - Algorithm
  - SingleView
---

Coursera Penn-Perception Week2 笔记整理

https://www.coursera.org/learn/robotics-perception/home/week/2

<!-- more -->

上一周课程介绍了透视投影和齐次坐标系，并重点描述了相机模型中各个参数的具体含义。本周的课程主题为「投影变换」，可以视为对已有知识的加深和应用。

## Where we are？

这一节主要讲解了如何通过单幅图像计算相机的位置和朝向（外参），也就是第一周讲的Rotation and Translation。这个过程是循序渐进的，大致思路如下：

### 使用一个灭点估计旋转

相机坐标系通常以水平向右为X轴，竖直向下为Y轴，垂直于图像平面为Z轴。

在图像中找到一个**灭点**，设其齐次坐标为$[0,0,1,0]^T$。由第一周的知识，若不考虑径向畸变（内参$L$），则得到如下等式：

![](https://github.com/HusterHope/blogimage/raw/master/PennP2-1.jpg)

> 将这个点代入等式后，我们发现在改变相机位置（Transformation）不会引起灭点位置的变化。这一事实可以类比夜空的星，当我们保持观察角度不变，任意行走也不会改变星星在视野中的位置。

通过该式，我们可以计算出$r_3$，以及相机坐标系相对世界坐标系的两个旋转角$\alpha,\beta$。

![](https://github.com/HusterHope/blogimage/raw/master/PennP2-2.jpg)

旋转变换具有3个自由度（yaw,pitch,roll）,一个灭点只能估计出其中两个，因此需要更多条件来估计旋转。

### 使用两个灭点估计旋转

找到物理世界中互相垂直的两个方向（比如人造建筑物的一角，规则的十字路口等），使用与一个灭点类似的计算方式，我们能够得到$r_1,r_2,r_3$中的两个参数（不妨设为$r_1,r_2$），由三轴的空间关系，得$r_3=r_1\times r_2$，从而得到完整的旋转矩阵。

![](https://github.com/HusterHope/blogimage/raw/master/PennP2-3.jpg)

### 使用四个已知点坐标估计位置

找到物理世界中的一个平面，与这一平面上的四个点，设这些点在物理世界中的$Z$坐标为0，则可得到一个齐次变换$H$。

![](https://github.com/HusterHope/blogimage/raw/master/PennP2-4.jpg)

本节内容只是作为引入，我们会在后面几节内容中深入讲解并多次用到这类变换。

刚才的讨论是以第一人称视角进行的，在获得相机的旋转和平移参数后，我们就可以在世界坐标系中表示相机的位置和朝向了，即从第一人称转换到第三人称。

## 计算投影变换

### 投影变换的性质

* 共线的三个点在变换后依然共线；
* 三条或三条以上的平行线变换后相交于一点；
* $3\times3$的可逆阵，具有8个相互独立的参数。

### 棋盘格变换

首先介绍一种特殊的投影变换——棋盘格变换，它直观且易于理解。

![](https://github.com/HusterHope/blogimage/raw/master/PennP2-5.jpg)

在一个棋盘格图案上找3个特殊点，分别为水平方向的无穷远点$(1,0,0)$，垂直方向的无穷远点$(0,1,0)$，原点$(0,0,1)$，变换到图像平面后对应点分别为$A,B,C$（假设它们对应的坐标已知，分别为$a,b,c$）

因为$A,B,C$均不共线，可以令它们为基，构造投影变换$(\alpha a,\beta b,\gamma c)$，得如下关系式：

![](https://github.com/HusterHope/blogimage/raw/master/PennP2-6.jpg)

此时还无法解出这个投影矩阵的具体值，因此在棋盘格上引入第四个点$(1,1,1)$，设其变换到图像平面后对应点$D$，代入上式，即可解出$\alpha,\beta,\gamma$的具体值。

### 一般的投影变换

![](https://github.com/HusterHope/blogimage/raw/master/PennP2-7.jpg)

这时我们没有标准的棋盘格图案了，但可以引入类似的**虚拟点**。

首先找到这些虚拟点映射到$a,b,c,d$的投影矩阵，记为$T$；再找到它们映射到$a',b',c',d'$的投影矩阵，记为$T'$。这样，$a,b,c,d$到$a',b',c',d'$的整个投影变换满足：

$$a'=T'T^{-1}a$$.

## 投影变换和灭点

> 投影变换也称作单应（Homography）或共线（Collineation）

如果我们按照上一节的方法构造投影变换，则容易看出该变换第一列与灭点$A$线性相关，第二列与灭点$B$线性相关。将两个灭点相连，能够得到水平线（Horozon）

由齐次坐标的点-线对偶性关系，水平线$h=a\times b$.

* 水平线的位置能够反映相机的两个旋转角。例如，当相机朝下时，水平线会出现在图像中央偏上的位置。

![](https://github.com/HusterHope/blogimage/raw/master/PennP2-9.jpg)

## 利用交叉比进行单视图度量

这一节围绕交叉比（Cross ratio）这个概念展开。最核心的性质是：投影变换前后，线段交叉比大小不变。如下图所示。

![](https://github.com/HusterHope/blogimage/raw/master/PennP2-1.jpg)

利用这一性质，我们可以在单视图内进行一些简单的长度计算（Single View Metrology）。

## 足球赛中的双视图度量

1966年世界杯，英格兰VS德国的比赛，一个具有争议的进球判罚。

参考文献：Reid, Ian D. and Andrew Zisserman. “Goal-directed Video Metrology.” *ECCV* (1996).
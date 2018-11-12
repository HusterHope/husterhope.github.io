---
title: "【Perception-1】图像构成的几何原理"
layout: post
categories: 做笔记
tags:
  - Math
  - Algorithm
  - SingleView
---

Coursera Penn-Perception Week1 笔记整理

https://www.coursera.org/learn/robotics-perception/home/week/1

<!-- more -->

## 透镜模型

薄凸透镜模型和焦距公式

![](https://github.com/HusterHope/blogimage/raw/master/PennP1-1.jpg)

## 单视图几何

- 物理世界中平行的线可能在图像中相交；
- 在物理世界中相交而在图像中平行的线可以用来确定焦距（如下图所示）；

![](https://github.com/HusterHope/blogimage/raw/master/PennP1-2.jpg)

* 水平线（Horizon）的确定：通过平行线在图像中的交点确定灭点 -> 找到多个灭点构成的线 -> 形成水平线

> Horizon is the set of all directions into the infinity.

- 物理世界中永远不存在灭点。

## 有关透视投影的更多知识

提及了透视画法的演变历史及其几何原理。

![](https://github.com/HusterHope/blogimage/raw/master/PennP1-3.jpg)

假设场景中有一个平放在水平面上的圆，讨论改变相机高度和焦距对透视的影响

* 改变高度：相机越低，圆被「挤压」得约严重。
* 改变焦距：不会改变形状。

![](https://github.com/HusterHope/blogimage/raw/master/PennP1-4.jpg)

![](https://github.com/HusterHope/blogimage/raw/master/PennP1-5.jpg)

## 灭点

若图像平面与地面垂直，则水平线即为光心高度。

应用：高度检测

![](https://github.com/HusterHope/blogimage/raw/master/PennP1-6.jpg)

## 透视投影

齐次坐标系的引入：想象一个位于图像后方的虚拟观察者。

![](https://github.com/HusterHope/blogimage/raw/master/PennP1-7.jpg)

* 点不再是点，而是由观察者视点引出的一条射线。
* 线可视作平面法向量。

-> “**点为射线，线为平面。**”

## 点-线对偶性

![](https://github.com/HusterHope/blogimage/raw/master/PennP1-8.jpg)

特殊情况：

* 无穷远点：$(a,b,0)$
* 无穷远线：$(0,0,1)$

## 旋转和变换

$x,y,z$坐标轴分别用红、绿、蓝色表示。

* 世界坐标系和相机坐标系的关系及其转换方式。

角标$c$和$w$分别代表相机坐标系和世界坐标系，$^cR_w$表示由相机坐标系变换到世界坐标系所需要的旋转矩阵，反之，$^wR_c$表示上述变换的逆变换。类似地，$^cT_w$表示由相机坐标系变换到世界坐标系所需要的平移矩阵。

保持**旋转矩阵行列式为1**。

使用4*4的变换矩阵，同时包含平移和旋转两个变换。

旋转的逆变换就是旋转矩阵的转置，平移的逆变换是改变正负号。

## 针孔摄像机模型

Pinhole Camera Model，这个pinhole不知道是不是这么翻译，就暂且叫做「针孔」吧，可以理解成小孔成像。针孔的位置叫做「光心」（Optical Center）。

![](https://github.com/HusterHope/blogimage/raw/master/PennP1-9.jpg)
$$
L(K [R\quad t]\left[       \begin{array}{lcr}         X \\ 1      \end{array}     \right])
$$

* $L$：透镜参数（内参）
* $K$：传感器和孔的空间关系（内参）
* $R, t$：相机机身参数（朝向和世界坐标中的位置）（外参）

需解决的问题：中心位置和焦距

调焦的原理：

$$x = f \frac{X}{Z}$$ 

* 焦距测量：Internet/Checkboard/手工粗略测量（参见「单视图几何」）

## 滑动变焦特效

滑动变焦（Dolly Zoom），最早是导演希区柯克用于电影《Vertigo》中的一种表现手法，所以有时也称为“Vertigo效果”。通过一边改变摄像机位置，一边改变焦距，使得图像中部分对象看上去大小不变，同时前景/背景靠近或远离该对象，形成一种视觉假象。[这篇文章](https://www.vmovier.com/31234/)里有一些案例展示。

这个内容同时也是本章的Programming Exercise，几何原理大致如下所示。

![](https://github.com/HusterHope/blogimage/raw/master/PennP1-10.jpg)

## 相机内参

第一人称三维场景->光学图像平面->数字图像平面

上述过程产生的变换矩阵合并为$K$。

由三维场景到光学图像平面这一步很简单，就是两个相似三角形等式写成矩阵形式。以坐标$y$为例：

$$\frac{f}{y'}=\frac{Z}{Y}$$ 

其中$f$是焦距，$y'$是图像平面上的纵坐标，$Z$是光心到物体的距离，$Y$是物体真实大小。

将$x$与$z$按照上述规则类似写出，得到如下等式和对应的矩阵：

![](https://github.com/HusterHope/blogimage/raw/master/PennP1-11.jpg)

接下来是光学图像平面到数字图像平面，说白了是一个单位换算和坐标变换的过程。因为光学图像中心是由光心出发，与图像产生的垂足（该点称为法点，记作$(p_x,p_y)$），而图像矩阵的原点通常在左上角。此外，物理单位长度和像素单位长度通常也不一致（存在缩放系数$\alpha_x,\alpha_y$）。使用如下形式的变换矩阵加以处理：

![](https://github.com/HusterHope/blogimage/raw/master/PennP1-12.jpg)

其中$s$用于调整图像的倾斜。

两步合并，即得校准矩阵$K$：

![](https://github.com/HusterHope/blogimage/raw/master/PennP1-13.jpg)

## 世界坐标系到相机坐标系转换

经过上一节的讨论，我们顺利解决了第一人称三维世界到二维像素平面的变换过程。然而第一人称建立在「以观察者自身为宇宙中心」的假设下，当存在多个观察者——或者说来自多角度的图像——我们就需要事先从第三人称转换到第一人称了，这部分就是外参$R$和$t$。

这部分变换的具体细节已在前文「旋转和变换」小节有所描述，因此直接代入上一节的矩阵等式中，用$[R\quad t]$替换$[I\quad 0]$，这就离我们的针孔摄像机模型又近了一步（只差$L$了，会在「相机标定」中介绍）。

![](https://github.com/HusterHope/blogimage/raw/master/PennP1-14.jpg)

注意两个特例：

* 棋盘格平面

稍用一些计算机视觉的知识，我们可以快速提取到棋盘格平面上的每个「角点」。由于这些点全部处在同一平面上，不妨设它们的坐标为$(X,Y,0)$，因而我们可以直接删去$[X,Y,Z,1]$中的$Z$坐标与$[R\quad t]$中的第三列($r_{13},r_{23},r_{33}$)，将新的$[R\quad t]$矩阵记作矩阵$H$，称为同质变换。

* 以光心为旋转中心构造全景图片

由于相机空间坐标不变，$[R\quad t]$可简化为$[R]$。

此外，旋转相机时任意两物体间视角不变，因此获取深度信息仍然需要相机的位置变化。

## 从灭点计算内参

这个部分原视频讲得很清楚，主要是用些几何知识，过程很漂亮，不作过多整理。

## 相机标定

这部分虽然是整个章节的最后一部分内容，却是在相机模型中最先处理的部分。不同于前文的各种线性变换，本节主要对「径向畸变」这种非线性变换进行建模。

* 径向畸变：由中心开始，像素点随中心半径按比例畸变

模拟：从$u,v$两个方向分别拟合。

![](https://github.com/HusterHope/blogimage/raw/master/PennP1-15.jpg)

剩下的就是用Matlab工具包进行实际操作的过程演示，略。

---

重点小结：

* 理解透视投影和齐次坐标系；
* 齐次坐标系中的点、线和对应关系；
* 灭点及其应用（高度估计、内参计算等）；
* 相机模型中各个参数的具体含义，变换构造原理；
* 通过「滑动变焦特效」的编程作业理解各种变量的关联。


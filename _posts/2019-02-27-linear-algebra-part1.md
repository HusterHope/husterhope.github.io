---
title: "《Linear Algebra Done Right》学习笔记（1）"
layout: post
categories: 做笔记
tags:
  - Math
---

<!-- more -->

本科的数学课程非常偏向工程应用，「线性代数」一遍学下来，大概只记得矩阵乘法怎么算了。然而很多时候，必须从一个概念的本质出发，才能真正有所感悟。

主要参考：《Linear Algebra Done Right》第二版，[书籍主页](http://linear.axler.net/index.html)（包含作者提供的视频讲解） | [习题答案](https://mrashidi568blog.files.wordpress.com/2017/01/ladrsm2e.pdf)

其他资源：[3Blue1Brown - 线性代数的本质](https://www.bilibili.com/video/av6731067) 用浅显的动画还原了基本原理，值得一看。

笔记以每个章节的要点为框架，整理关键的定理、引理、性质等，对非常重要的部分进行**加粗**。详细的定义和证明过程需参考原文。

整本书的笔记分为3个部分，1-4章，5-7章，和8-10章。原本打算全部刷完一起整理，然而刷习题的过程太磨人了，于是一边推进度一边整理笔记回顾。

引一位UCB学长的告诫作为填坑动机。

> 最后还是强调一点，读书是一定要刷习题的，不然学不懂，学习数学要扎实，不扎实的一味求快最后只能推倒重来。95分和85分还是有本质区别的。

希望这次不要「一味求快」。

## 向量空间及其基本性质

### Complex Numbers

* 复数域和基本性质

* 域的概念

### Definition of Vector Space

* 二元组、三元组到 $n-list$

* 引出Vector的概念，定义数乘和加法两类运算。

* 多项式向量空间 $P(F):$所有系数在$F$中的多项式集合。

### Properties of Vector Spaces

* 基本性质
* 加法单位元和加法逆元唯一

### Subspaces

* 子空间是向量空间的一个子集，其中$\{0\}$是最小子集，$V$是最大子集
* **子空间的验证方法**（三点）：加法单位元、加法封闭性、乘法封闭性

### Sums and Direct Sums

类比：并集&互不相交集合的并集

* **Direct Sum**：若$\forall v\in V$可被唯一表示成$u_1+···+u_m$的形式（$u_j\in U_j$），则称$V$是子空间$U_1,...,U_m$的Direct Sum，记作$V=U_1\oplus ···\oplus U_m$

* 判断$n$个子集可以Direct Sum $\Leftrightarrow$ $0$的唯一表达
* 判断两个子集($U,W$)可以Direct Sum $\Leftrightarrow$ $U \cap W=\{0\}$

## 有限维向量空间

### Span and Linear Independence

* 向量张成的空间 $span(v_1,···,v_m)=\{a_1v_1+···+a_mv_m:a_1,···,a_m\in F\}$ 
* 线性无关：只有 $ a_1=···=a_m=0$ 能够使得 $a_1v_1+···+a_mv_m=0$。
* **线性相关引理**：从线性相关组中移除一个能由其他向量线性表示的向量，则余下向量张成的空间不变。
* 线性无关组长度 $\leq$ 张成列表（spanning list）长度
* 有限维向量空间的子空间也是有限维

### Basis

线性独立且张成空间$V$，则构成$V$的一组基(Basis)

* **$V$中任意向量$v$可由基唯一线性表示。**
* 任意张成列表可以缩减为基；任意线性独立列表可以扩展为基。
* 对偶子空间：$\forall U\subset V,\exists W\subset V, s.t.V=U\oplus W $.

### Dimension

$V$中所有基均等于$V$的维度，记为$dimV$。

* $U$是$V$的子空间$\Rightarrow$ $dimU\leq dimV$
* **基的证明方法**（三证二）：线性独立、张成$V$、长度等于$dimV$
* 类似集合论的计数原理。

> 重要习题：
>
> 习题12：反证法
>
> 习题17：数学归纳法

## 线性变换

### Definitions and Examples

线性变换：满足可加（Additivity）和同质（Homogeneity）的**函数**

> 由向量空间$V$到$W$的线性变换记作$T\in L(V,W)$。$V$和$W$将在本章后续内容中多次出现。

在各个例子中，$From\ F^n\ to\ F^m$尤为重要。

* 向量空间中的线性变换**实质为基变换**。
* 两个线性变换的乘积具备一些类似乘法的性质，但不可交换。

### Nullspaces and Ranges

零空间（Nullspaces）：$V$中在$T$变换后等于$0$的元素集合

值域（Ranges）：$W$中能够被$T$映射到的元素集合。

* **两者关联**：$dimV = dim\ nullT+dim\ rangeT$
* 内射（injective）定义：若$T(u)=T(v)$，则$u=v$；性质：$nullT=\{0\}$
* 满射（surjective）定义：$rangeT=W$
* 应用：**齐次/非齐次线性方程组解的个数问题**

> 齐次：射零->找零空间的维度
>
> 非齐次：射像->找值域的维度

秩（rank）：若$A$是一个大小为$m·n$的矩阵，则行秩和列秩分别为每行和每列向量所张成的空间维度。

* $dim\ rangeT=rank\ M(T)$ 
* 行秩等于列秩。

### The Matrix of Linear Map

线性变换的本质是基变换，而基中的每个元素变换后，都可被新空间的基唯一表达。矩阵是一种**高效表达基变换**的方式。

* 将大小为$m·n$，元素在集合$F$中取值的矩阵集合记作$Mat(m,n,F)$，可以验证$Mat(m,n,F)$也构成一个向量空间。
* 矩阵乘法的含义：两个线性变换的依次作用（乘积）

> 留意「线性变换矩阵」、「向量矩阵」和「矩阵乘法」三者之间**定义和运算的统一性**。

### Invertibility

线性变换可逆（Invertible）：同时满足内射和满射。

* 若在两个向量空间中存在可逆线性变换，则两向量空间同构，且维数相等。

* 线性变换和矩阵表达的形式完全等价（同构）
* **自映射**（Operator）：$T\in L(V,V)$，简记为$L(V)$。

对有限维向量空间上的自映射而言，**可逆$\Leftrightarrow$满射$\Leftrightarrow$内射**。

> 重要习题：
>
> 习题3：线性变换的构造方式
>
> 习题16：零空间和值域的变换关系
>
> 习题24

## 多项式

> 本章内容基本不涉及线性代数，目的是为后面章节作知识铺垫。部分定理可参考「应用代数」课程笔记。

### Degree

度（Degree）：最高次项的次数。

* “根与因子等价”
* 多项式根的数量不超过度。
* $(1,z,...,z^m)$线性无关。
* 多项式除法

### Complex Coefficients

* **代数基本定理**：无常数项的复系数多项式至少有一个根。

* 多项式在复数域上的唯一分解定理。
* 复数运算的基本性质

### Real Coefficients

* 无常数项的实系数多项式的唯一分解形式。
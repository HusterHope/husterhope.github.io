---
title: "弱典型性和强典型性"
layout: post
tags:
  - AEP
  - InformationTheory
categories: 做笔记
---

信息论中的两个重要概念，概念杂乱，稍作梳理。

<!-- more -->

## 典型性

先从「典型性」的概念入手，何为「典型性」？

设想一个投掷N次硬币的实验，若正面出现的次数约为1/2，则可以说该序列为典型序列(Typical Sequence)。

为了寻求更为一般化的描述，引入「弱典型性」和「强典型性」的概念。

支撑两个概念中主要定理即为「弱渐进等分性定理」和「强渐进等分性定理」，即弱AEP和强AEP（Asymptotic Equipartition Property），它们是弱大数定律产生的结果。

由弱典型性的概念能够引出**信源编码定理**。

## 相关概念

* 弱大数定律(Weak Law of Large Numbers, WLLN)

对独立同分布的随机变量$Y_1,Y_2,···$（用$Y$表示对应的一般随机变量），当$n\to\infin$，有$\frac{1}{n}\sum_{k=1}^{n}Y_k\to EY$。

* 经验熵(Empirical Entropy)：$-log\ p(x_k)$ 的算术平均

$$-\frac{1}{n}log\ p(\textbf{x}) = \frac{1}{n}\sum_{k=1}^{n}[-log\ p(x_k)]$$

* 信源编码模型（图自[Coursera - Information Theory 5-2](https://www.coursera.org/learn/information-theory/lecture/3ARLK/chapter-5-section-5-2)）

![](https://github.com/HusterHope/blogimage/raw/master/20181105-1.png)

编码器将长度为n的随机信源序列$\textbf{X}$映射到集合$I=\{1,2,···,M\}$中（即$\chi^n \to M$，$\chi$为字母表），该方式成为分组编码，n为分组长度，解码器根据映射关系重构序列$\hat{X}$。

* 错误率（Error Probability）

由于映射不一定为一一映射，所以解码器可能得到与$\textbf{X}$不一致的序列$\hat{X}$，称$P_e=P_r\{\hat{X}\neq\textbf{X}\}$为错误率。

* 码率(Coding Rate)

$$R = \frac{1}{n}log\ M$$

若$\vert M\vert=\vert \chi^n\vert$（即形成一一映射），则$R=log\vert\chi\vert$。通常会选用$R<log\vert\chi\vert$，以实现数据压缩。

* 信源编码定理

正定理：给定任意小的错误率$P_e$，当$n$足够大时，存在码率任意接近于$H(X)$的分组编码；

逆定理：对任意码率小于$H(X)-\zeta$的长度为$n$的分组编码，当$n\to \infin$时，$P_e \to 1$。

也就是说，若码率大于熵，则可靠通信能够实现；若码率小于熵，可靠通信不能实现。

* 经验分布（Empirical Distribution）

设$\textbf{x}\in\chi^n$.

$N(x;\textbf{x})$表示序列$\textbf{x}$中$x$出现的次数；

$\frac{1}{n}N(x;\textbf{x})$表示序列$\textbf{x}$中$x$的相对频率；

$\{\frac{1}{n}N(x;\textbf{x}):x\in\chi\}$为$\textbf{x}$的经验分布。

例如：设$\textbf{x}=(1,3,2,1,1)$，则

$N(1;\textbf{x})=3, N(2;\textbf{x})=N(3,\textbf{x})=1$；

$\textbf{x}$的经验分布为$\{\frac{3}{5},\frac{1}{5},\frac{1}{5}\}$。

## 两种典型性的比较

* 弱典型序列的性质是：经验熵近似于真实熵。

此外，对于足够大的n，弱典型序列出现的概率极大程度上趋近于$2^{-nH(X)}$，且$\chi^n$中的序列$\textbf{x}$共有$2^{nH(X)}$个为弱典型序列。（这就体现出了“等分性”的含义）

我们可以在这些弱典型序列上构建一一映射的分组编码，对其他序列任意映射，可以极大概率上获得极低错误率的信源编码。

* 强典型序列的性质是：经验分布接近于真实概率分布。

* 强典型能够推出弱典型，繁殖不成立。
* 强典型性只能用在有限字母表上，弱典型性可以用在可数无限字母表上。
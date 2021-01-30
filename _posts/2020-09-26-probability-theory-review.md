---
title: "概率论和数理统计 | 复习（1）"
layout: post
categories: 做笔记
tags:
  - Math
---

迫于Paper中出现的相关概念早已遗忘，重读《概率论与数理统计》，主要整理自同济大学数学系的教材。

<!-- more -->

## 随机事件与概率

概率论：研究随机现象的统计规律性

->对事物进行观察：试验(Trial)

相同条件下可重复进行/试验结果不止一种，试验前需明确所有结果（样本空间）/每次试验的结果无法预知

随机事件是样本空间的子集，只包含一个样本点的随机事件称为基本事件

等可能概型包含古典概型与几何概型，前者样本空间包含有限个样本点，后者包含无线个（一维区间、二维平面或三维空间等）。

古典概型的关键在于样本空间及事件A中的样本点个数。

几何概型->计算机模拟->蒙特卡罗方法

条件概率$P(B|A)=\frac{P(AB)}{P(A)}$

条件概率乘法公式

相互独立：$P(AB)=P(A)P(B)$ -> 系统可靠性问题

**独立和相交没有任何关联。**

完备事件组：交集为空，并集为样本空间。完成了对样本空间的一个分割。

全概率公式（先验概率）：$P(B)=\sum_{i=1}^{n}P(A_i)P(B|A_i)$

**贝叶斯公式（后验概率）**：先根据条件概率的定义写出第一步，对分子再次套用条件概率，分母使用全概率公式

$$P(A_i|B)=\frac{P(A_iB)}{P(B)}=\frac{P(A_i)P(B|A_i)}{\sum_{i=1}^{n}P(A_i)P(B|A_i)}$$

## 随机变量及其分布

**（一维）随机变量**：对样本空间$\Omega$中的每一个样本点$\omega$，有唯一一个实数$X(w)$与它对应，那么就把这个定义域为$\Omega$的单值实值函数$X=X(\omega)$为~.

随机变量是函数，一般大写，取值一般小写。

-> 可以让不同样本点对应不同的实数，也可以让多个样本点对应于一个实数。

-> 引入随机变量后，可以使用数学中的微积分工具讨论随机变量的分布。

随机变量$X$的**分布函数**：$F(x)=P(X\leq x), -\infty <x<\infty$ 

-> 值域[0,1]，单调不减，右连续

**概率函数**=分布律=分布列，用以描述**一维离散型随机变量的取值和概率对应关系**，注意与上述概念区分。

对于连续型随机变量，$f(x)$为**概率密度函数**。满足非负性和规范性（$\int_{-\infty}^{+\infty} f(x)dx=1$）

分布函数和概率密度函数的关联：$F(x)=\int_{-\infty}^xf(t)dt$  

> 离散型：概率函数/分布律
>
> 连续型：概率密度函数

### 离散型随机变量

* 二项分布/伯努利分布/两点分布（非正即负）

$X\sim B(n,p)$ , $P(X=k)=C_n^kp^k(1-p)^{n-k}$

* 泊松分布（二项分布n很大而p很小时的一种极限形式）

$X\sim P(\lambda)$ , $P(X=k)=\frac{\lambda ^k}{k!}e^{-\lambda}, \lambda >0, k=0,1,2,...,n,...$

另：一个被遗忘的数学公式

$lim_{n\to+\infty}(1+\frac{\lambda}{n})^n=e^{\lambda}$  

* 超几何分布（不放回抽样）

$X\sim H(N,M,n)$ , $N$件产品中包含$M$件不合格品，从中不放回抽取$n$件得到的不合格品件数$X$

当$n<<N$时，可用二项分布近似

* 几何分布

在伯努利试验中记每次试验中$A$事件发生的概率$P(A)=p\ (0<p<1)$, 设随机变量$X$表示$A$事件首次出现时已经试验的次数。$P(X=k)=p(1-p)^{k-1}, 0<p<1$, 记作$X\sim Ge(p)$ 

* 负二项分布

在伯努利实验中，设随机变量$X$表示$A$事件第$r$次出现时已经试验的次数，则$X$的取值为$r,r+1,...,r+n,...$,相应的分布律为

$$P(X=k)=C_{k-1}^{r-1} p^r (1-p)^{k-r}$$, $k=r,r+1,...,r+n,...$

称随机变量$X$服从参数为$r,p$的负二项分布，记为$X\sim NB(r,p)$

### 连续型随机变量

* 均匀分布 $X\sim U(a,b)$ 

* 指数分布 $X\sim E(\lambda)$ 常用于描述电子元件的寿命分布

  概率密度函数为$f(x)=\lambda e^{-\lambda x}, x\geq 0, \lambda >0$ ($x<0$时$f(x) = 0$ )

* 正态分布 $X\sim N(\mu, \sigma ^2)$ ，概率密度函数$f(x)=\frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(x-\mu)^2}{2\sigma ^2}}, -\infty < x<+\infty$ , 当$X\sim N(0,1)$时，分布函数$F(x)$记作$\Phi(x)$

  对$X\sim N(\mu,\sigma^2)$, 则$\frac{X-\mu}{\sigma}\sim N(0,1)$

### 随机变量函数的分布

* 离散型随机变量函数的分布：$Y=g(X)$的分布律应该对$g(x_i)$相同取值的概率进行相加
* 连续型：服从正态分布的随机变量线性函数仍然服从正态分布

两个常用定理：

-> 设连续型随机变量X的密度函数为$f_x(x)$, $Y=g(X)$是连续型随机变量，若$y=g(x)$为严格单调函数，$x=g^{-1}(y)$为相应的反函数，且为可导函数，则$Y=g(X)$的密度函数为

$f_Y(y)=f_X(g^{-1}(y))·\mid g^{-1}(y)'\mid$

-> 设$X\sim N(0,1)$，则当$k\neq 0$时，$Y=kX+b\sim N(k\mu +b, k^2\sigma ^2)$, 特别地, $\frac{X-\mu}{\sigma} \sim N(0,1)$.

## 二维随机变量及其分布

二维随机变量：

样本空间$\Omega$，样本点$\omega$，对应有序实数$(X(\omega),Y(\omega))$，值域$\Omega_{(X,Y)} \subset R^2$

设$(X,Y)$为二维随机变量，对于$\forall (x,y)\in R^2$，联合分布函数：

$F(x,y)=P(X\leq x, Y\leq y)$

联合分布函数性质：值域$[0,1]$；固定任意一个自变量，$F(x,y)$均为另一自变量的右连续单调非减函数；单一自变量趋向$-\infty$时$F(x,y)\to 0$，两自变量均趋向$+\infty$时有$F(x,y)\to 1$；**矩形公式**.

对二维连续型随机变量，使用**联合分布函数刻画统计规律比较复杂，通常会使用联合密度函数。**

常用的二维随机变量：

* 二维均匀分布
* 二维正态分布 $N(\mu_1,\mu_2, \sigma_1^2,\sigma_2^2,\rho)$

![](https://github.com/HusterHope/blogimage/raw/master/20200926-1.png)

**边缘分布**：若已知二维随机变量的联合分布$F(x,y)$，则其中一个随机变量的分布被称为~

例如，$F_X(x)=P(X\leq x)=P(X\leq x, Y<+\infty)=F(x, +\infty), -\infty <x<+\infty$ 为随机变量$X$的边缘分布函数。

随机变量之间的**相互独立性**：

若$\forall x,y \in R$，有$F(x,y)=F_X(x)F_Y(y)$成立，则称随机变量$X$与$Y$相互独立。其中$F_X(x)$和$F_Y(y)$分别为$X$和$Y$的边缘分布函数。

-> 离散型：当$X$取定$x_i$时，$Y$的取值不受任何影响。

-> 连续型：在一切公共连续点上都有$f(x,y)=f_X(x)f_Y(y)$ 成立，其中$f(x,y)$为$(X,Y)$的联合密度函数，$f_X(x)$和$f_Y(y)$为边缘密度函数。

对于二维正态分布，参数$\mu_1$、$\sigma_1$描述$X$的分布，参数$\mu_2$、$\sigma_2$描述$Y$的分布，$\rho$反映了两者的关系。$X$与$Y$相互独立的充分必要条件时$\rho=0$.

---

（未完待续）

概率论的基础概念部分基本收尾，后面刷完随机变量的数字特征和大数定律，就正式进入统计的部分。

> 大一第一次学统计的时候一堆概念也没整清楚，最终靠着考前刷课本一周+背公式应付了考试，果然出来混早晚要还啊（
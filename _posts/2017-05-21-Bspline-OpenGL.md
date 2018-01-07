---
title: "【OpenGL】绘制递推定义的B样条曲线"
layout: post
excerpt: "实验：OpenGL固定管线绘制递推定义的、具有首尾重节点的、幂次可变的、可编辑的B样条曲线。"
last_modified_at: 2012-02-05T10:27:01-05:00
tags:
  - B-Spline
  - OpenGL
  - Algorithm
categories: 解问题
---

实验：OpenGL固定管线绘制递推定义的、具有首尾重节点的、幂次可变的、可编辑的B样条曲线。

感想：定义最重要！定义最重要！定义最重要！

学习B样条的详细教程：

[密歇根理工大学CS3621-Notes](http://www.cs.mtu.edu/~shene/COURSES/cs3621/NOTES/) Unit 6

------

生成B样条曲线，需要用到**节点**、**控制点**和**基函数**(系数)，其中基函数的计算最复杂，特别是要求利用**递推式(recursive definition)**计算，这成为了这次实验中最难的地方。下面先简单介绍一下这些概念，如果没有完全理解，请查阅上文中的课程笔记。

首先，B样条曲线是一种**参数曲线**，即将$[0,1]$范围内（0代表曲线起点,1代表终点）的一个参数 u 的值映射到二维（或更高维，不过本文主要讨论二维）空间上。

**节点(knots)**：给定集合$U$，它包含$m+1$个有理数$u_0, u_1, u_2, …, u_m$，且满足：

$u_0\leq u_1\leq u_2 \leq … \leq u_m$

**节点矢量(knot vector)**：由节点数据组成的$n+1$维矢量$[u_0, u_1, u_2, …, u_m)$.

**节点区间(knot span)**：$[u_i,u_{i+1})$ 称为第i个节点区间.

**k重节点(k-multiple knot)**：如果$u_i=u_{i+1}=…=u_{i+k-1}$且$k>1$，则称$u_i$为$k$重节点.

**基函数(Basis functions)/系数(Coefficients)/权重(Weights)**：可以看作控制点对曲线上的点的作用力，且曲线上任何一点受到的来自多个控制点的作用权重之和为1.

节点和曲线有什么关系？

节点可以看成曲线上的**细分点(division points)**，所有B样条曲线的基函数都需要依赖这些细分点来定义。为了计算方便，我们通常会让集合$U$落在区间$[0,1]$中，即：

**$0=u_0 ≤ u_1~ ≤ u_2~ ≤ … ≤ u_m=1$**

这样，曲线的参数$u$在$[0,1]$中的每一个取值，必定有唯一的一个节点区间与之对应，进而可以通过计算相应的基函数来绘制$t$在该取值下的二维坐标。

接下来引入B样条基函数计算的递推式：
$$
N_{i,0}(u) = 
\left \{
	\begin{aligned}
	&1\qquad if\ u_i \leq u\leq u_{i+1} \\
	&0\qquad otherwise
	\end{aligned}
\right.
\\
N_{i,p}(u) = \frac{u-u_i}{u_{i+p}-ui}N_{i,p-1}(u) + \frac{u_{i+p+1}-u}{u_{i+p+1}-u_{i+1}}N_{i+1,p-1}(u)
$$
这个递推的计算过程如下：

![img](http://ohn6qfqhe.bkt.clouddn.com/bs-2.jpg)

说明：节点个数为$m+1$；控制点个数为$n+1$，曲线的幂次为$p$。

如果觉得计算起来比较抽象，可以看这里的例子：[B样条的定义](http://www.cs.mtu.edu/~shene/COURSES/cs3621/NOTES/spline/B-spline/bspline-basis.html)(同样出自上面的Notes)。

看完以后感觉最精华的部分：系数的含义。

通过一定的分析我们知道，在计算$N_{i,p}(u)$时，需要计算$N_{i,p-1}$和$N_{i+1,p-1}(u)$，前者的非零区间为$[u_i, u_{i+p}]$，后者为$[u_{i+1}, u_{i+p+1}]$。下面这张图很好地给出了它们的关系：

![img](http://ohn6qfqhe.bkt.clouddn.com/bs-4.jpg)

所以，$N_{i,p}(u)$是$N_{i,p-1}(u)$两项的线性组合。

------

基本的定义就说这些，下面讲讲计算基函数的算法实现。

(为了便于区分，我们**用参数 t 替代上文中的参数u**进行说明)

如果顺着定义走，我们不难想到：用一个$m+1$行，$n+1$列的二维数组来存放基函数的表，计算起来也直观，使用也方便。但是写程序的时候就发现，因为$ t$ 从$0-1$变化的时候，每过一个节点都需要更新基函数的表，并且用递归计算基函数开销非常大。整个程序的实现随即陷入困难。

当然，写Notes的作者也知道这个问题，所以自然地在他的笔记里给出了一种改进的算法，可以看成即用即取，是一种正向计算的过程。核心算法的伪码请参考：

[计算系数的方法](http://www.cs.mtu.edu/~shene/COURSES/cs3621/NOTES/spline/B-spline/bspline-curve-coef.html)

根据$t$的值来反推哪些基函数在这个区间内不为零，而这些不为零的因子又是由哪一个$0$次基函数生成的，这种构思十分巧妙，值得学习。注意一点，用这种方法实现时，我们需要**首尾都设置$p+1$重节点**，以满足计算需要。

作者在最后还启发式地提出了问题：你能让这个算法更优吗？

能，但是我就先按照原文来作实现了。理解了以后实现起来不难，主要注意参数的值不能混了，特别是控制点的数量、节点的数量和递推式里的$n、m$是相差了$1$的。

```
#define MAX_CP 9    //控制点数量=MAX_CP+1
#define MAX_P 5     //支持的最高幂次

int num_cp = 0;                 //控制点个数 =n+1
int p = 3;                      //幂次
int m;                          //实际结点个数-1

float N[MAX_CP+1];              //最大系数数组

float u[MAX_CP+1+MAX_P+1];      //m+1个节点组成的向量数组

// 初始化基函数表
//
void initN()
{
    for(int i = 0;i < num_cp;i++)
        N[i] = 0;
}

// 根据节点个数初始化节点表，默认为均匀节点
//
void initU()
{
    m = num_cp+p;
    for(int i = 0;i < m+1;i++)
    {
        if(i <= p)
        {
            u[i] = 0;
            continue;
        }
        else if(i >= m-p)
        {
            u[i] = 1;
            continue;
        }
        u[i] = (float)(i-p)/(float)(m-2*p);
    }
        //printf("u[%d] = %.2f\n",i,u[i]);
}

// 根据曲线参数t计算基函数序列
//
void calN(float t)
{
    //将数组N初始化
    initN();
    initU();
    //处理边界情况
    if(0 == t)
    {
        N[0] = 1.0;
        return;
    }
    else if(1 == t)
    {
        N[num_cp-1] = 1.0;
        return;
    }
    //遍历节点表，找出t所在的区间:u[k] <= t < u[k+1]
    int i,k = 0;
    for(i = 1;i < m+1;i++)
        if(t < u[i])
        {
            k = i-1;
            break;
        }
    N[k] = 1.0;
    //阶数d
    for(int d = 1;d <= p;d++)
    {
        N[k-d] = N[k-d+1] * (u[k+1]-t)/(u[k+1]-u[k-d+1]);
        for(i = k-d+1;i <= k-1;i++)
            N[i] = N[i] * (t-u[i])/(u[i+d]-u[i]) + N[i+1] * (u[i+d+1]-t)/(u[i+d+1]-u[i+1]);
        N[k] = N[k] * (t-u[k])/(u[k+d]-u[k]);
    }
}
```

基函数算出来以后，画B样条就只需要套用我的上一篇文章里画Bezier曲线的OpenGl代码模板就ok，并且能很容易地实现编辑、改精度、删除上一个控制点、清屏等功能。代码还在完善中，就先不贴出来了，有需要参考可以留言私信。

放一张画出来的B样条曲线证明这种方法的可行性（幂次为3，控制点数为7）：

![img](http://ohn6qfqhe.bkt.clouddn.com/bs-3.png)

------

再次感谢MTU的课程笔记。图片侵删。
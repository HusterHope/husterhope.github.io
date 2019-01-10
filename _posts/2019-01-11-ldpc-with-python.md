---
title: "【Python】LDPC码模拟实验"
layout: post
tags:
  - Python
  - LDPC
  - InformationTheory
categories: 解问题
---

低密度奇偶校验码（LDPC）是在含噪声的通信信道中使用的纠错码，可降低错误率。

<!-- more -->

本文主要参考[Ta, Tuan. “A Tutorial on Low Density Parity-Check Codes.” (2009)](https://pdfs.semanticscholar.org/5ef7/e9fffad481404be2160f9763caf9eaacc39c.pdf)整理，并用Python实现LDPC解码过程和模拟信道传输。

## 构造校验矩阵

$m$行$n$列的校验矩阵，设每行有$w_r$个1，每列有$w_c$个1，且满足：

* $w_r\ll m, w_c \ll n$ 
* $w_r=w_c \frac{n}{m}$ (regular)

本文均假设$n=2m$，则$w_r=2w_c$。通过每次右移两位的方式（类似循环码），可以构造出符合要求的校验矩阵$H$。代码如下所示（用$p$代替$w_c$，最小为2）

```python
def genH(m,n,p):
    H = np.zeros((m, n))
    for i in range(m):
        j=2*i
        for j in range(2*i,2*p+2*i):
            H[i,j%n]=1
            j=j+1
    return H
```

## 模拟信道传输

整个传输过程需要经过 编码->调制->传输->解调->解码 这些步骤。

为了省去生成矩阵的计算，我们直接使用全0码字（codeword）作为待传输消息，长度为$n$。

### 调制和解调

采用BPSK算法：

* 调制（modulate）：对于待发送的二元信息，将0编码为1，1编码为-1进行传输。
* 解调（demodulate）：对于接收到的信息，将大于0的值解调为0，小于0的值解调为1。

代码如下：

```python
# BPSK(c for codeword)
def modulate(c):
    for i in range(np.size(c)):
        if c[i] == 0:
            c[i]=1
        else:
            c[i]=-1
    return c

def demodulate(y):
    for i in range(np.size(y)):
        if y[i] > 0:
            y[i]=0
        else:
            y[i]=1
    return y
```

### 噪声模拟

传输的过程会引入噪声，这里采用高斯噪声模拟，即消息的每个位置（entry）会受到一个满足高斯分布噪声信号的扰动。基础噪声分布为$N(0,1)$，并给予不同的信噪比（SNR），定义如下：

$$SNR=10log_{10}(\frac{p}{\sigma ^2})dB$$ 

为了方便，取信号强度$p=1$，则噪声强度$\sigma=10^{-\frac{SNR}{20}}$。

这部分可以直接用Numpy包中的函数实现，注意使噪声信号长度和码字长度相等，便于直接相加。

```python
sigma = 10**(-snr/20)
noise=np.random.normal(0,sigma,size=n)
```

## LDPC解码

LDPC解码是个略为复杂的过程，这里实现其中较为简单的一种：*Hard-decision Decoder*。

首先理解一下校验矩阵$H$的含义，$H$的每一行对应了一个奇偶校验方程，而每一列则分别对应了码字的每个位。$H[i,j]=1$表示码字的第$j$位需要由第$i$个方程进行校验。由这个思路，可将码字的每一位看作一个消息结点（message node），每一个方程看作一个校验节点（check node），由它们之间的联系可生成一幅信息传播图（也叫做Tanner Graph）。一个满足LDPC条件的$H$和对应的Tanner Graph如下图所示。

![](https://github.com/HusterHope/blogimage/raw/master/20190111-1.png)

为了演示解码算法如何执行，假设我们有Codeword $c=[1,0,0,1,0,1,0,1]^T$，经过信道传输后收到消息$y=[1,1,0,1,0,1,0,1]$（第二位出现错误），下面通过信息节点和校验节点的相互通信来实现纠错（只是类比为通信，实际上没有发生信息传输），主要分为以下几步：

* 信息节点将$y$的每一位依次发给与它相连的校验节点；
* 校验节点根据每个信息节点发来的消息，构造出Receive表；
* 校验节点对所有收到的信息求和，之后分别剔除每个信息节点的消息，作奇偶校验，以确定给每个节点发回的消息，构造出Send表；（Receive表和Send表共同组成下图中的Table 3）

* 信息节点收到校验节点发回的消息后，与原有消息放入同一行，并依据每行1或0的个数作出Decision（少数服从多数，如第二行两个0和一个1，则decision为0，实现了纠错），组成Table4；

![](https://github.com/HusterHope/blogimage/raw/master/20190111-2.png)

> Table4的最后一行中$f1\to1$ 应该改为$f1\to 0$，属于paper原作者的笔误。

* 最后输出解码后的值$y'$，过程结束。

在用Python实现的过程中需要注意索引的准确表达，特别是处理发送和接收时，可能需要多次遍历校验矩阵。个人实现的版本如下（这里同样假设n=2m）：

```python
# LDPC decode(hard decision)
def decode(H,y,m,n,p):
    fr = np.zeros((m, 2 * p)) # check nodes received
    fs = np.zeros((m, 2 * p)) # check nodes send
    sum = np.zeros(m) # check nodes received sum(for parity check)
    # message nodes table
    # 前p列为校验节点发来的消息，第p+1列为原始消息，第p+2列作为游标
    c=np.zeros((n,p+2)) 
    y1=np.zeros(n)

    # Fill the check nodes received table
    for i in range(m):
        count=0
        for j in range(n):
            if H[i][j] == 1:
                fr[i,count]=y[j]
                sum[i]=sum[i]+y[j]
                count = count+1

    # Calculate the check nodes send table
    for i in range(m):
        for j in range(2*p):
            fs[i,j]=(sum[i]-fr[i,j])%2

    # Fill the message node table
    for i in range(m):
        count=0
        for j in range(n):
            if H[i][j]==1:
                index=int(c[j,p+1])
                c[j,index]=fs[i,count]
                count = count+1
                c[j,p+1]+=1

    # Fill the last column with y
    for i in range(n):
        c[i, p] = y[i]

    # Decision
    for i in range(n):
        count=0
        for j in range(p+1):
            if c[i,j] == 1:
                count+=1
        if count > (p+1)/2:
            y1[i]=1
    return y1
```

用上面的消息$y=[1,1,0,1,0,1,0,1]$和对应的$H$测试，可以得到$y$被纠错后的结果$y1$。

```python
m=4 # Number of rows
n=8 # Number of columns
p=2 # Number of 1s in a colomn

H=np.zeros((4,8))
H=[[0,1,0,1,1,0,0,1],[1,1,1,0,0,1,0,0],[0,0,1,0,0,1,1,1],[1,0,0,1,1,0,1,0]]

y=np.zeros(n)
y=[1,1,0,1,0,1,0,1]

y1 = decode(H,y,m,n,p)
print(y1)
```

## 错误率

最后计算一下不同密度和不同信噪比（即手动修改$p$和$SNR$，反复运行）的错误率：

```python
def errorRate(c,y1):
    err=0
    total=np.size(c)
    i=0
    for i in range(total):
        if c[i] != y1[i]:
            err+=1
    r = err/total
    return r
```

可以描绘SNR和p对错误率影响的曲线：

![](https://github.com/HusterHope/blogimage/raw/master/20190111-3.png)

![](https://github.com/HusterHope/blogimage/raw/master/20190111-4.png)

## 结论

* 根据上述曲线，可以初步判断出错误率随信噪比的提升而下降，前期总体呈线性趋势，最终缓慢收敛到0；

* p的取值对错误率的影响较大。
* 由于本文只进行了较为浅层的实验，构造的LDPC校验矩阵比较随意，且采用了*Hard Decision*作为解码和纠错标准，因此不能很好地体现出p的取值在LDPC中的作用。相关问题有更为细致的方法和深入的文献。



后记：LDPC码早在上世纪60年代就由Robert G. Gallager在博士论文中提出，当时并未引起太大轰动，近些年来随着软硬件的发展而变得应用广泛，因此LDPC码也称为**Gallager码**。



ps. 本文的真实性质是《信息论与编码》课程的最后一个Groupwork，在写正式Report前给自己整理个思路。
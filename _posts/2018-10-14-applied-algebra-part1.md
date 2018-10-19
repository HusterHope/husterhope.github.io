---
title: "【应用代数】多项式和有限域"
layout: post
categories: 做笔记
tags:
  - Math
  - Algorithm
---

初次接触《应用代数》课程，作为数学学渣还是要整理一遍概念和思路。

虽说是课堂笔记，但部分理论框架还有些价值，发到博客里给有需要的同学看看吧。

<!-- more -->

经过5节大课的跟进，目前个人理解：

将传统意义上的多项式、整数/有理数/实数/复数集合上的加法、乘法、公约数、素数之类的基本概念加以抽象和归纳，形成一套形式上较为抽象的代数系统。

既然称为「应用代数」，那么必然存在其应用价值，目前教授只提及了构造有限域在密码学方面比较有用这一个点。抱着”不谈问题直接抛定义扔性质结论都是中国式教科书“的态度，在正式整理笔记前还是来回顾一个应用。

> 离散对数与密码学：
>
> 设想一个应用，比如需要构建字母表a-z元素间的对应关系，从而进行加密。比较传统的做法可以是将每个元素映射为右移5个单位，即{a-f,b-g,...x-c,y-d,z-e}。显然这种加密方式的安全性较差。
>
> 而如果利用有限域的知识，我们可以构造出一个大小为27（3^3个）的多项式有限域，将a-z与域中的多项式对应，再利用**模一个不可约多项式**来构建这些多项式之间的对应关系（找出每一个的逆元），形成一套离散对数系统，达到稳定的加密效果。

为了理清上面这些看上去比较杂乱的概念和它们之间的关系，就有了这5节课的内容。

第一节课主要为一些抽象代数系统，包括群、域、整环和欧式环相关的概念；

第二节课和第三节课主要在探讨欧式环这一代数系统，欧式环上有很多优良性质，并抽象出最大公约数、互素等实用概念；

第四节课就进入了核心：有限域的构造，并引出多项式域。由于域的定义比欧式环更严格，因此从欧式环构造域时需要附加额外条件。

第五节课主要归纳总结有限域上的性质和推论。

---

## 群、域、整环、欧式环

### 群 Group

集合 $(G, \circ)$，运算满足封闭性、结合律，存在单位元、逆元。

* 交换群，也称为阿贝尔群（Abelian group）：在群的基础上同时满足交换律。

### 域 Field

集合 $(F, +, · )$ ，满足在「+」运算和「·」运算下均为交换群，「·」对「+」有分配律。

### 整环 Integral Domain

集合 $(D, +, ·)$ ，在「+」运算下满足交换群，在「·」运算下具有单位元并满足结合律和消去律，「·」对「+」有分配律。

* 结合律的存在就已经包含了封闭性。
* 整环相比域，缺少了「·」下的逆元。

### 欧式环 Euclidean Domain

在整环定义的基础上，同时满足：

* 若$b\neq 0$，$g(a)\leqslant g(ab)$；
* 对所有$a,b\neq 0$，$\exist q,r$使得$a=qb+r$，其中$q=0$或$g(r)<g(b)$.

其中$g(x)$称为Size function，为非负整数。

例：证明高斯整数集合$\{a+b\sqrt{-1},a,b\in Z\}, g(a+bi)=a^2+b^2$是欧式环。

解答：[Prove that the Gaussian Integer's ring is a Euclidean domain](https://math.stackexchange.com/questions/600424/prove-that-the-gaussian-integers-ring-is-a-euclidean-domain)

## 最大公约数和欧几里得法

* 最大公约数**（Greatest Common Divisor, GCD）**
* 在整环中，$a \mid b$（读作a整除b）意味着 $\exist c$ 使得$b = c·a$，称a为b的**因子**（divisor）。

### 整环上的GCD

公约数：对集合$B = \{ b_1,···,b_n\}$，若$a\mid b_i$ 对$\forall i=1,···,n$成立，则称a为集合B的公约数。

最大公约数：设d是集合B的公约数，若满足其他所有公约数都**整除**d，则d是集合B上的最大公约数，记作：$d = gcd\{b_1,···,b_n\}$.

* 在抽象代数中，“最大”不是直接比较大小，而是沿用整除的概念。

* 并不是所有整环上的集合都存在GCD。

### 欧式环上的GCD

在欧式环D下，若$d = gcd\{b_1,···,b_n\}$且$\{d_1,···,d_k\}$为$\{b_1,···,b_n\}$的公约数，则：

$g(d)=max\{ g(d_1),g(d_2),···,g(d_k)\}$

* Size function可以比较大小，由欧式环定义易得上述结论。

**Theorem 0.1**：欧式环D下任意有限集合$B=\{b_1,···,b_n\}$，一定存在gcd（设为d），且能够表示成集合B中所有元素的线性组合：$d = \sum_{k=1}^{n} \lambda_kb_k$.

> **证明：（构造法）**
>
> 构造 $S=\{\sum_{k=1}^{n}\mu_kb_k\mid \mu_k \in D\}$,
>
> 取 $d:g(d) \triangleq min\{g(x)\mid x \in S\}.$ （因为g有下界，所以g(d)一定存在）
>
> 声明：d就是$gcd(b_1,···,b_n)$ 
>
> 即证：(1) $\forall b_i, d \mid b_i$；(2)$\forall e$（e为$b_1,···,b_n$的公因子，有$e\mid d$ .
>
> 先证(2)：对 $\forall e$，有$e \mid b_i, i=1,2,···,n$，则$b_i = q_i e$ .
>
> $\because d\in S$，$\therefore d =  \sum_{k=1}^{n} \mu_k b_k=e \sum_{k=1}^{n} \mu_k q_k$，则$e\mid d$。
>
> 再证(1)：$d =  \sum_{k=1}^{n} \mu_k b_k$ ，由欧式环性质有
>
> $\forall b_i ,\exist q,r \in D, s.t. b_i=qd+r$，其中$r = 0$或$g(r)<g(d)$.（即只需证$r=0$）
>
> $\because r=b_i-qd \in S$ $ \therefore g(d)\leqslant g(r)$，与$g(r) < g(d)$矛盾。
>
> $\therefore r = 0$，即$d \mid b_i \quad \square$ 

**Lemma 0.1**（辗转相除法）：$\forall s,t \in D$ （D为欧式环），有 $gcd(s,t) = gcd(s,t-s)$.

**Theorem 0.2** （欧几里得法）：$\forall s,t,r \in D$ （D为欧式环），有 $gcd(s,t) = gcd(s,t-rs)$.

* 以上引理和定理由Theorem 0.1易证。

**Theorem 0.3** 设D是一个欧式环，对$\forall t\in D, \forall m,n\in Z_+$，有$gcd(t^n-1,t^m-1)=t^{gcd(m,n)-1}$。

> 证明：逐层递推

![](https://github.com/HusterHope/blogimage/raw/master/AppliedA1-1)

欧几里得扩展算法（Euclid’s Extended Algorithm）：用于寻找线性组合的系数。

## 欧式环上的唯一分解定理

设D为一个欧式环。

### unit

将乘法单位元的因子称为unit。

例如，整数域上的unit为$\pm1$；复数域上的unit为$\pm1,\pm i$；多项式有限域上的unit为**最高幂次为零的多项式**（因为这些常数均有逆元，在后面的小节中介绍）。

### 相伴 Associates

若$\exist u,s.t. a=ub$，其中u为unit，则称a,b**相伴**。若$b\mid a$而a与b不相伴，则称b为a的**真因子**。

### 平凡因式分解 Trivial Factorization

对于D中的元素b，b的因式分解可写作$b=a_1a_2···a_r$。若每个a要么为unit要么为b的相伴元，则称$b=a_1a_2···a_r$为平凡因式分解。

### 素元和互素 Prime and Relatively Prime

若对D中元素p的所有可能的因式分解均为平凡因式分解，则称p为**素元**。

对D中两元素a,b，若$gcd(a,b)=1$（或任意其他unit），则称a与b**互素**。

**Lemma 0.2** 若a,b互素，则$\exist s,t \in D$，使得$as+bt=1$. （套用定理0.1易得）

**Lemma 0.3** 若p为D中的一个素元，则对于$\forall a\in D$，若p不整除a，则a与p互素。

> 证明：由欧式环性质知$a=qp+r (r \neq 0)$，由定理0.2（欧几里得法）知
>
> $gcd(p,a)=gcd(p,a-qp)=gcd(p,r)=1$，即a与p互素。

**Lemma 0.4** 若p是素元，且$p\mid ab$，则$p\mid a$或$p\mid b$（或两者同时）成立。

> 证明：若$p\mid a$，命题得证。
>
> 否则，由引理0.2，得$as+pt=1$
>
> 两边同乘b，得$b=asb+ptb$
>
> $\because p\mid ab\quad \therefore \exist c,s.t. cp=ab$
>
> 即$b=scp+tbp=(sc+tb)p, p\mid b .$ 
>
> 命题得证。

**Lemma 0.5** 在欧式环内，若a是b的真因子，则$g(a)<g(b)$。

> 证明：由已知：$a\mid b, b=ac$ （c不为unit）
>
> 则$a=qb+r, g(r)<g(b)$ 
>
> $\therefore r = a-qb=a(1-qc)$
>
> $\because c\neq unit\quad \therefore qc\neq1$
>
> $\therefore g(a)\leqslant g(r)<g(b)\quad \square$ 

### 唯一分解定理

**Theorem 0.4** 假设$b\in D$不是unit，则：

(i)b可被写成素数的乘积：$b = p_1p_2···p_r$，其中$p_i$为素数；

(ii)如果b能够被写为另一种表达形式，比如$b=q_1q_2···q_s$，其中$q_i$为素数，则必有$r=s$，且经过适当排序后，$p_i$和$q_i$为相伴元。($i=1,2,···,r$)

> 主要证明思路：数学归纳法+引理0.4+引理0.5

## 由欧式环构造域

### 同余和等价类

对欧式环D中的元素m，当且仅当$m\mid (a-b)$时，称a与b关于m**同余**（congruence），记作$a\equiv b\quad mod\ m$。

同余关系将D划分为若干个不相交的子集，这些子集即为**等价类**（equivalence classes），记作$D\ mod\ m$。

包含a的等价类通常记作$\bar{a}$，一般用最小非负元素表示等价类（$Z_4 = \{\bar{0},\bar{1},\bar{2},\bar{3}\}$）

### D mod m 环

定义两个二元运算如下：

* 加法：$\bar{a}+\bar{b}=\overline{a+b}$ 

* 乘法：$\bar{a}·\bar{b}=\overline{a·b}$ 

则D mod m与上述两个二元运算一并构成环，其中

* 加法单位元：$\bar{0}=\{x \in D, x\equiv0\ mod\ m\}$
* 乘法单位元：$\bar{1}=\{x \in D, x\equiv1\ mod\ m\}$ 

> 注：这个环相对域而言只缺少了乘法下的逆元，因此后面满足特殊条件时就可以构造域了。

### D mod m 域

**Theorem 0.5** 若p为素元，则D mod m连同上述两种运算共同构成域。

> 证明：只需找出任意元素$a\in D$的逆元即可。
>
> $\because gcd(a,m)=1$
>
> $\therefore \exist s,t, s.t. as+mt=1\ (mod\ m) $
>
> 所以s即为a的逆元，证毕。

### 多项式域

* 复数域的构造

设$D=R[x]$为拥有实系数的多项式集合，$p(x)=x^2+1$为不可约多项式（即在实数域下无法进行因式分解），则$p(x)$是$R[x]$中的素元。且$R[x]\ mod\ p(x)=\{a+bx,a,b\in R\}$形成域。该域的加法和乘法分别定义为：

![](https://github.com/HusterHope/blogimage/raw/master/AppliedA1-2)

> Tip：$mod\ x^2+1$可以理解为$x^2+1=0$，即$x^2=-1$。
>
> Note：上述加法和乘法运算规则等同于$x=i$的情形。因此这个域实际上就是复数域C。

* $F_p[x]$下的多项式除法规则

![](https://github.com/HusterHope/blogimage/raw/master/AppliedA1-3)

> Tip：与常规多项式除法类似，只是注意系数相乘后需要模除p，负值可以加p后得到正值（回顾等价类的概念）

* **有限域的构造规则**

有了上面的知识，我们就可以开始构造有限域了。一个有限域可以用如下方式构造：

有限域可以是一个包含若干多项式的集合。这些多项式的系数为mod p，多项式的表达形式通过**mod一个不可约多项式**后得到。例如在上面复数域的构造中，$mod(x^2+1)$就得到了$ax+b$的形式。

特性：给定一个素数域$F_p$和最高幂次为m的不可约多项式，则能够构造出含有$p^m$个元素的有限域。这就能够方便地从小素数构造出一个大有限域。

> 密码学中的离散对数应用

## 有限域的性质

**Theorem 0.6** （有限域的元素数量）有限域$F_q$上元素的数量一定等同于某个素数的整数幂。即$q=p^m$，$p$为素数。

* 加法：p为加法循环周期（$u_k=u_{k+p}$），称为**特征**(characteristic)；
* 乘法：所有非零元的集合$F_q^*$构成循环群，**阶**(order)为$q-1$。

**Theorem 0.7**（拉格朗日定理）若$\alpha \in F_q^*, ord(\alpha)=t$（即$\alpha^k=\alpha^{k+t}$），则$t\mid(q-1)$。

**Definition 0.1** （不可约多项式）多项式$p \in F(x)$，p的最高次幂为正数，若对所有可能的$p=bc$中（$b,c\in F(x)$），均有b或c为常数，则称多项式p为**不可约多项式**。

* Note：判定p是否可约，很大程度上取决于域的范围。例如$p=x^2-2$在有理数域上为不可约多项式，而在实数域上即为可约多项式。

**Theorem 0.8**（解的数目）对$\forall p(x)\in F[x]$，F为一域，若$deg(p(x))=m$，则$p(x)=0$至多有m个解（在F上的解）

> 证明：（数学归纳法）
>
> 当$m=1$时，记$p(x)=ax+b, a,b\in F$，则有解$x=-a^{-1}b$，满足结论；
>
> 设$m=t$时结论成立($t\geqslant1$)
>
> 则$m=t+1$时，若$p(x)=0$无解，满足结论。
>
> 否则，设$\alpha$为该方程的一个解($\alpha \in F$)
>
> $\because p(x),x-\alpha \in F(x)$ 
>
> $\therefore p(x)=q(x)(x-\alpha)+r(x)$，其中$deg(r(x))<1$或$r(x)=0$。
>
> 将$x=\alpha$代入上式，得$p(\alpha)=q(\alpha)(\alpha-\alpha)+r$，即$r=0$。
>
> 对$\forall \beta \neq \alpha$，若$\beta$为$p(x)=0$的解，则$\beta$一定为$q(x)=0$的解。
>
> $\because q(x)=0$至多有t个不同的解
>
> $\therefore p(x)=0$至多有$t+1$个解。 $\square$

**Lemma 0.6** （求阶公式）令$order(\alpha)=t$，则$ord(\alpha^i)=t/gcd(i,t)$.

在证明L0.6前，先给出一个引理L0.60：$\beta ^a=1$是$ord(\beta)\mid a$成立的充分必要条件。

> 证明L0.60：
>
> 先证已知$\beta^\alpha=1$的情况。
>
> 设$ord(\beta)=t$，则$\beta^t=1$。
>
> $\because \beta^a=1, a\geqslant t$
>
> $\therefore a=qt+r$ 
>
> $\because r<t$，若$r\neq0$，则与$ord(\beta)=t$矛盾。$\therefore r=0$，即$a=qt$成立，正向得证。
>
>  反向易证，略。

> 证明：（互相整除推相等）
>
>

**Theorem 0.9**（阶为t的元素数目）

（L0.6/T0.9待整理，LaTex虽好看，写起来真的慢..）

ps.网页中「\exist」，这其实就是那个和「任意」对应的「存在」符号..不知道为什么Web端的LaTex加载不出来..
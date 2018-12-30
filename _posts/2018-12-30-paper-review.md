---
title: "论文评审学习参考"
layout: post
categories: 做笔记
tags:
  - PaperRelated
---

迫于某个课程需要不同小组之间盲审他人的论文（或许叫做报告比较合适），学习一些评审论文的基本知识。包括评审流程、内容结构和常用句式。

<!-- more -->

## 总评审流程

以[CVPR2019](http://cvpr2019.thecvf.com/)的流程为例：

* **议程主席（Program Chairs，下文简称PC）**将需要评审的论文指派给每个**主要领域主席（Primary Area Chair，下文简称AC）**
* 由AC建议大约8名**审稿人（Reviewer）**参与评审（大约每3人一组），论文由机器算法下发
* 审稿人写**评审意见（Reviews）**，由AC过目后发给论文作者
* 作者向AC和审稿人提供对评审意见的反馈（Rebuttal）
* 审稿人更新评审结果
* AC与审稿人讨论确定论文最终意见（接收or拒稿），并考虑是否作为亮点（Spotlight）和口头报告（Oral）推荐候选
* PC根据时间/空间限制，最终确定Spotlight和Oral的论文

作为审稿人，需要重点关注如何写好一份评审意见。

## 内容结构

清晰的结构有助于AC快速作出判断。一份好的评审意见应当包括如下内容：

### 必要内容

* 摘要 Summary

* 优点 Strengths（Pros）

* 缺点 Weaknesses（Cons）

* 评级和理由 Rating and Justification

每部分的具体要求见参考资料[2]。

### 可选内容

* 置信度 Confidence

  部分期刊和会议需要提供，代表审稿人对自身评审意见的把握程度（通常是论文内容相关知识的能力），一般评级为1-5，分数越高代表越有把握。比如：

  	5: 完全确定 absolutely certain

  	4: 很大程度上确定 confident but not absolutely certain

  	3: 比较确定 fairly confident

* 细节问题 Minor Points（Minor comments）

  可以包含参考文献问题、拼写问题、语法/句式表达问题，以及对一些细节的追问（比如具体参数、结构、实现方法之类）

## 参考表达

 摘要表达：

The authors develop an novel scheme for xxx

The paper is grounded on a solid theoretical motivation and the analysis is sound.

The paper is generally well written, easy to read and understand, and the results are compelling.

优点表达：

（引子）The paper is to be commended for the following aspects: ...

（写作）This paper was a delight to read. / The paper is very well written and easy to follow.

（观点）The ideas build up in an intuitive way. / The motivation for xxx is well delivered.

（实验）Sound experiments showing the benefit of ...

（内容）This paper is a nice addition to xxx / The maths is very rigorous. / This paper offers a promising direction for xxx / 

缺点表达：

（补充内容）There is no mention of xxx

I was expecting the authors to mention xxx in their related work.

（补充实验）xxx would require more justification (more developed experimental section.)

Experiments limited to xxx

（补充逻辑）The link between xxx and xxx is not obvious.

...

后续看到合适的再补充

## 参考资料

[1] https://openreview.net/

\[2\] [How to Write Good Reviews for CVPR](https://www.dropbox.com/s/725p60wcajbb8xh/How%20to%20Review%20for%20CVPR.pptx) （对应中文版：[如何做好论文评审工作？CVPR 2019程序委员会有话说](https://zhuanlan.zhihu.com/p/49975649)）






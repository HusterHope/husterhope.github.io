---
title: "Mac+LaTeX学习笔记"
layout: post
tags:
  - LaTeX
categories: 做笔记
---

记头一次正经学习使用LaTeX..

<!-- more -->

为了便于修改论文LaTeX模板里的内容，需要相对完整的了解一下LaTeX所支持的功能和各种命令及文件的作用。

使用LaTeX前，首先安装一个LaTeX发行版，Mac环境下推荐使用[MacTex](https://www.tug.org/mactex/mactex-download.html)。

接下来整理部分主要参考自文档：[110分钟了解LaTeX](https://mirrors.tuna.tsinghua.edu.cn/CTAN/info/lshort/chinese/lshort-zh-cn.pdf)。这篇文档被简称为`lshort`，可以看成LaTeX的简易使用手册。目前大版本已更新至第6版，最新中文版可以在清华大学开源镜像站找到：[lshort/Chinese](https://mirrors.tuna.tsinghua.edu.cn/CTAN/info/lshort/chinese/)。粗读了一遍这份说明，整理部分前期可能需要的信息留作参考。

---

* 基本特性

$\LaTeX$是一个文档准备系统 (Document Preparing System)

$\LaTeX$对大小写敏感。

（这两句话中的$\LaTeX$均使用`\LaTeX`命令打出，后文为方便书写直接采用LaTeX表述）

* 初次编译

创建一个文本文件，命名为`helloworld.tex`，可以理解为文档内容的源代码文件。随后输入以下内容：

```
\documentclass{article} %%文档类型，如article/book/report等
%%导言区：通常使用\usepackage调用宏包以及全局文档设置

\begin{document}
``Hello world!'' from \LaTeX. %%正文内容
\end{document}
```

编译后得到一系列文件：helloworld.aux；helloworld.log；helloworld.pdf。其中第三个文件即为输出的文档。

对于投稿论文而言，对应单位会给出投稿模板，而这类模板往往会出现各种后缀的文件，如bst/bib/aux/ouy/cls等，第一次看可能会一头雾水，遂摘录些关于这些文件的说明：

> .sty 宏包文件。宏包的名称与文件名一致。
>
> .cls 文档类文件。文档类名称与文件名一致。
>
> .bib BIBTEX参考文献数据库文件。
>
> .bst BIBTEX用到的参考文献格式模板。
>
> .log 排版引擎生成的日志文件，供排查错误使用。
>
> .aux LATEX生成的主辅助文件，记录交叉引用、目录、参考文献的引用等。
>
> .toc LATEX生成的目录记录文件。.lof LATEX生成的图片目录记录文件。
>
> .lot LATEX生成的表格目录记录文件。
>
> .bbl BIBTEX生成的参考文献记录文件。
>
> .blg BIBTEX生成的日志文件。
>
> .idx LATEX生成的供makeindex处理的索引记录文件。
>
> .ind makeindex处理.idx生成的用于排版的格式化索引文件。
>
> .ilg makeindex生成的日志文件。
>
> .out hyperref 宏包生成的 PDF 书签记录文件。 

因此我们只需重点关注`.tex`文件即可。

* 文件组织

`\include{<filename>}`可以在源代码中插入文件，便于将文章分章节编写和管理（尤其针对毕业论文和书籍等）。`\include`命令读入后会将文件另起一页，当不需分页时，可直接使用`\input{<filename>}`命令。

* 快速编译

当我们仅进行错误检查而不输出文档时，可以使用`syntonly`宏包进行快速编译。加载这个宏包后在导言区使用`\syntaxonly`指令即可。

* 章节和目录

  * 章节层次

    ```
    \chapter{⟨title⟩}
    \section{⟨title⟩} \subsection{⟨title⟩} \subsubsection{⟨title⟩}
    \paragraph{⟨title⟩} \subparagraph{⟨title⟩}
    ```

  * 目录：只需在适当的位置加上`\tableofcontents`

  * 文档结构：可分为前言、正文和后记，每个部分可以单独形成附录。同时也会自动调整页码计数方式和格式。

  * 标题和作者：分别是最常用的`\title{}`和`\author{}`，均为标题页中的必须项。

* 交叉引用

在能够被引用的地方（如公式、章节、图表等位置）使用`\label{<label-name>}`命令。之后在其他地方使用`\ref{<label-name>}`或`\pageref{<label-name>}`分别生成交叉引用的编号和页码。

> 注意 参考文献的引用有专用的`\cite`命令

* 脚注/边注/列表/代码段/表格/图片等

这些特定格式通常会由模板给出，暂不整理。

* 浮动体

> 内容丰富的文章或者书籍往往包含许多图片和表格等内容。这些内容的尺寸往往太大，导致
> 分页困难。LaTeX为此引入了浮动体的机制，令大块的内容可以脱离上下文，放置在合适的位置。

常用的浮动体环境为图片和表格（figure和table），可以使用不同参数使得浮动体出现在不同位置。双栏环境下，可以使用横跨两栏的浮动体格式（即论文中常见的占满整个横行的表格）。

* 数学公式排版

这方面在最早学习使用MarkDown语法时就顺便整理过。可参考3年前的一篇Blog：[MarkDown学习笔记](https://leohope.com/%E5%81%9A%E7%AC%94%E8%AE%B0/2017/08/21/Markdown/)。往往在写的时候查对应的符号表示方法即可。

---

2020.05.06：暂时整理上述内容，后续有需要使用的部分再补充更新。
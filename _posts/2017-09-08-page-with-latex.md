---
title: "在Github pages中渲染LaTex公式"
layout: post
excerpt: " "
categories: 解问题
tags:
  - Github
  - Jekyll
  - LaTeX
  - Typora
  - Markdown
---



[跳过介绍，进入正文](#1)

博客环境：Github Pages／Jekyll

文本编辑器：Typora（以Latex语法书写数学公式）

稿件格式：Markdown

碰到的问题就是，原本在Typora中能够很好的支持LaTex公式的书写（包括行内和整行的公式），保存为`.md`文件Push到Github Page之后，却出现了Latex公式的源码（而不是渲染后的格式），严重影响阅读体验。强迫症表示一定要解决掉这个问题。

曾经在Wordpress里就碰到了这样的问题，但因为Wordpress后台太大，而笔者对于Web知识懂的又不多，改后台甚是麻烦，干脆在页面挂载了一个PDF，但对于搜索引擎很不利，因为内容全都检索不到了。

于是转而使用免费又好用的Github page。

那么，如何把Typora中支持的LaTex公式在网页中正常显示？

---

<div id="1"></div>

这个问题跟使用评论功能是类似的思路，即：在静态页面中挂载Javascript代码。而[MathJax](https://www.mathjax.org/)已经为我们做好了这个功能。正如其官网所说的：

> **Beautiful math in all browsers.**

在Jekyll模版中使用MathJax的过程只需要两步：

* 首先找到博客的目录：

![](http://ohn6qfqhe.bkt.clouddn.com/latex.png)



* 编辑`_include`目录下的`head.html`文件，在`<head>`和`</head>`中间的任意位置加上官方提供的脚本链接：

```
<head>
	...
	<script type="text/x-mathjax-config"> 
   		MathJax.Hub.Config({ TeX: { equationNumbers: { autoNumber: "all" } } }); 
   	</script>
    <script type="text/x-mathjax-config">
    	MathJax.Hub.Config({tex2jax: {
             inlineMath: [ ['$','$'], ["\\(","\\)"] ],
             processEscapes: true
           }
         });
    </script>
    
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript">
    </script>
</head>
```

* 完毕！

上面的三个脚本分别实现了：

1. 整行公式自动编号；
2. 将两个单美元符号`$`中间的内容看作行内数学公式（若文本内容中美元符号出现频率较高，建议禁用这一脚本）；
3. 从mathjax官网挂载脚本。

这一方法针对我使用的模板亲测有效，不同的环境可能会有所差异。

参考资料：

[MathJax官方文档](http://docs.mathjax.org/en/latest/start.html)

[How to set up MathJax on Jekyll and GitHub properly?](http://csega.github.io/mypost/2017/03/28/how-to-set-up-mathjax-on-jekyll-and-github-properly.html)

[Github问题讨论区](https://github.com/github/pages-gem/issues/307)
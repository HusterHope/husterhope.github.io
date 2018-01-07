---
title: "64位Win10下配置汇编用debug环境"
layout: post
excerpt: ""
categories: 解问题
tags:
  - Environment
  - Win10
---



新学期开始了，我终于开始啃书了..（没错就是浪了一个寒假并且连游记都没整理出来

------

首先在室友的推荐下选择了教材：

![img](http://ohn6qfqhe.bkt.clouddn.com/book.jpg)

《汇编语言（第三版）》 王爽<著> 清华大学出版社

作为初学来看，这本书很容易理解（至少前几章是这种感觉）。

看到第一个实验部分，就要进入虚拟8086模式的DOS环境了。

不巧的是，WIN10版本过高，不仅找不到debug.exe，甚至下载好的也不能运行（最后一代能启动debug.exe的系统是Windows XP），碰到了下面的问题：

![img](http://ohn6qfqhe.bkt.clouddn.com/64_16.png)

debug.exe下载链接：

> 链接: <https://pan.baidu.com/s/1qY6jExu> 密码: evd2

然后我们用DOSbox来解决上面弹窗中的问题。

> DOSbox下载地址：<http://www.dosbox.com/download.php?main=1>

安装完成后打开，在前对话框中输入命令。

![img](http://ohn6qfqhe.bkt.clouddn.com/dosbox.png)

首先对debug.exe所在的磁盘做一个<mount>操作，然后进入你的debug.exe存放的路径（比如这里我的路径是C:/Learn/AssemblyLanguage/debug.exe），命令如下图所示：

![img](http://ohn6qfqhe.bkt.clouddn.com/cmd.png)

这样debug.exe就成功运作了，输入汇编指令也可以按照预期执行。

![img](http://ohn6qfqhe.bkt.clouddn.com/ALr.png)

当然用虚拟机装个WINDOWS XP系统来运行也可以，不过略麻烦并且占空间..不推荐。

------

以上。

By SillyDog.

开学第一天的午后
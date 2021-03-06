---
title: "图床迁移说明"
layout: post
tags:
  - Github
categories: 写随笔
---

> 图床，简单的说就是用于存储博客静态页面中图像的外部服务器，将图像放入图床后，服务器可以返回一个外部链接，使用外部链接以节约本地服务器空间和数据请求量，并提升页面加载速度。

从前年博客建站开始就一直在使用七牛云服务作为图床，国内图像访问速度较快，免费存储空间10G，每个月提供的免费访问流量也从来就用不完。

所谓习惯成自然，从来没有考虑过这类服务会有一天突然挂掉，或者说，在它挂掉的那一刻，才意识到“免费的往往是最贵的”。

<!-- more -->

## 迁移方案

在征得一些替代方案的推荐后，最终选择将图像迁移至github仓库。

其他替代方案有：

* 新浪微博图床

配合同名chrome插件，可以快速拖拽上传并获取外链，但对象命名规则和目录管理不太清晰。万一哪天微博图床也因为有人上传违规文件惨遭水表，再迁移起来就麻烦的多了。

* 阿里云对象存储

鉴于之前用阿里云做过博客服务器，并不稳定的站点环境和日常宕机的服务器总觉得并不靠谱。

以及，当时买产品的时候客服态度相当好，但产品快到期的时候每天一个电话简直就是骚扰。所以查了一下文档和服务价格，还是不打算入坑了。
## 过程参考

github仓库作图床就像存储文本文件一样，丢上一堆图像上去依然稳定，但国内访问速度明显不如七牛云。

* 批量下载七牛云文件

七牛云网页端支持批量上传和删除，却不支持批量下载..没办法只能用脚本了，Windows用户可以参考[这篇blog](https://boke112.com/4288.html)，mac用户可以查看官方文档。

* Github仓库使用

http://jingpin.jikexueyuan.com/article/36279.html

* 更换原文章中的图像外链域名

---

最后，整体过程较为顺利，唯一的缺点就是站内图像加载速度可能会变慢。

以后不想再迁移了，如果哪天github凉了或者国内墙又高了，那就可以考虑关站了（flag
---
title: "Ubuntu相关环境配置索引"
layout: post
categories: 解问题
tags:
  - Linux
  - Environment
  - Ubuntu
---

为了查找方便，整理一下Ubuntu下的环境配置、包管理和相关工具用法。

<!-- more -->

## Apt包的相关操作整理

Linux自带的apt-get和相关的包管理指令。

> 详见[这篇文章](https://leohope.com/%E5%81%9A%E7%AC%94%E8%AE%B0/2018/04/02/linux-apt/)的整理

## 虚拟机下的磁盘扩展方法

> 参考链接：https://unix.stackexchange.com/questions/196512/how-to-extend-filesystem-partition-on-ubuntu-vm

## Shadowsocks客户端

Mac和Windows都有现成的图形化界面，使用方便。Linux下需要一些额外的配置。

> 详见[这篇文章](https://leohope.com/%E8%A7%A3%E9%97%AE%E9%A2%98/2018/04/07/ubuntu-shadowsocks-client/)的整理。

启动方法：

```
sudo sslocal -c '/home/username/Desktop/ss.json'
```

## Apache+MySQL / phpMyadmin相关配置

> [How To Install Linux, Apache, MySQL, PHP (LAMP) stack on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-14-04)
>
> [How To Install and Secure phpMyAdmin on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-ubuntu-14-04)

## 交叉编译ffmpeg

> 详见[这篇文章](https://leohope.com/%E8%A7%A3%E9%97%AE%E9%A2%98/2018/04/17/ffmpeg-cross/)的整理。

## 管理员权限打开Folder

```
sudo nautilus
```

## 查看局域网ip

```
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```

## 16.04安装CUDA

> 详见[这篇文章](https://leohope.com/%E8%A7%A3%E9%97%AE%E9%A2%98/2018/08/09/install-cuda-on-ubuntu/)的整理


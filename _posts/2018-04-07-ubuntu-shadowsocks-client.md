---
title: "Ubuntu 14.04中使用Shadowsocks客户端"
layout: post
categories: 解问题
tags:
  - Linux
  - Environment
---

Shadowsocks在Windows、Mac、Android、iOS下都有对应的图形化程序或者App，完全不需要配置，不知Linux有什么毒..居然挂个梯子都有坑

尝试过：Shadowsocks-qt5（图形界面） + [Proxy SwitchyOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?hl=zh-CN)，一段时间可以用，后来就再也上不了。

后改用shadowsocks命令行工具 + json文件 + Proxy SwitchySharp，成功。

部分参考：https://zhjpaul.github.io/2017/04/22/how-to-use-shadowsocks-in-ubuntu.html

<!-- more -->

### 安装ShadowSocks

终端(Terminal)依次输入以下指令：

```
sudo apt-get install python-gevent python-pip
sudo pip install shadowsocks
sudo apt-get install python-m2crypto
```

### 配置连接服务器的文件

创建一个空文件（New Document -> Empty Document），命名为`ss.json`（名称随意，后缀为json即可），文件内容如下：

```
{
  "server": "xx.xx.xx.xx",
  "server_port": 8388,
  "local_port": 1080,
  "password": "xxxxxx",
  "timeout": 600,
  "method": "xxxxx"
}
```

每行内容的含义依次为：服务器ip/端口号/本地端口（默认为1080，可不必修改）/服务器密码/时延/加密方法。

比如我的服务器ip为`233.233.233.233`，端口号`666`，密码`23333333`，加密方法为`aes-128-cfb`，则这份文件内容应该为：

```
{
  "server": "233.233.233.233",
  "server_port": 666,
  "local_port": 1080,
  "password": "23333333",
  "timeout": 600,
  "method": "aes-128-cfb"
}
```

### ss本地启动

假如上述json文件放置于桌面，则终端内输入以下指令：

```
sudo sslocal -c '/home/username/Desktop/ss.json'
```

### Proxy SwitchySharp配置

下载`SwitchySharp.crx`文件（百度能找到，网盘不让上传，避免水表就不放链接了），拖入Chrome浏览器中的扩展程序（`chrome://extensions/`）里，作如下配置：

![](http://ohn6qfqhe.bkt.clouddn.com/ss-linux-1.jpg)

然后在浏览器里启用ss代理选项，即可正常使用。

---

如果没有服务器，可以到[这篇文章](http://leohope.com/%E8%A7%A3%E9%97%AE%E9%A2%98/2018/01/19/v-p-n/)里查看快速搭建方法。
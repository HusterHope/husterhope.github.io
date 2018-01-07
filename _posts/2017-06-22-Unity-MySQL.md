---
title: "【Unity】网络游戏编程实践（3）MySQL存储历史记录"
layout: post
excerpt: "接着上篇的多人聊天应用来讲，如何添加一个聊天记录的功能呢？这就需要用到数据库了。"
categories: 做笔记
tags:
  - Unity
  - Database
  - Network
  - MySQL
---

接着上篇的多人聊天应用来讲，如何添加一个聊天记录的功能呢？这就需要用到数据库了。

简单的标准查询语句（增删改查）这里不再赘述，可以去看SQL的相关语法。

一般来说，客户端将数据写入数据库需要经过这样的过程：

开启服务器->服务器连接数据库->开启客户端->客户端连接服务器->用户在客户端发送消息->服务器接收消息->服务器将消息发送给数据库->数据库存储消息

如果要从客户端获取历史记录，只需在客户端和服务器中定义双方认同的通信字符串，比如GET_HISTORY。服务器在收到来自客户端的指定字符串时，判断其为“指令”而非“消息”，并从数据库获取相关数据，返送给客户端。

原理大致就由上文所述，不过最难的步骤在于“打通”，即将上面的原理在代码中实现出来，并且保证每个环节不能出差错。下面先从配置环境开始，使用的数据库为MySQL。

1.配置环境

（1）安装并启动MySQL（mysql-server）。

去年学数据库系统原理的时候用过MySQL，版本mysql-5.5.28-win32。

直接去官网下载就好了。[MySQL](https://dev.mysql.com/downloads/installer/)

安装过程中需要记住自己设置的用户名和密码。

（2）安装connector

用C#操作MySQL数据库时，需要安装MySQL官方提供的链接文件。

[MySQL-Connector](https://dev.mysql.com/downloads/connector/net/)

（3）引用库文件

上面两个安装好后，打开MonoDevelop项目面板的Edit References按钮，找到MySql.data。同时也需要引用System.Data。

（4）安装Navicat for MySQL

Navicat是个可视化操作数据库的程序，相比直接用命令行操作MySQL来说更方便。（当然习惯用命令行的Geek可以忽略了）

2.连接数据库

Navicat->文件->新建连接的时候，我们需要填入与Unity中的服务器相同的ip以连接数据库。新建一个数据表，增加id/name/msg三个属性，其中id为主键。

一般来说，操作MySQL的流程为：连接MySQL->选择数据库->执行sql语句->关闭数据库。

3.历史记录功能的服务端程序

（1）插入数据

运用语句：ExecuteNonQuery()

过程：

- 创建数据库连接；
- 创建Command对象，并指定一个SQL Insert、Update、Delete查询或存储过程；
- 把Command对象依附到数据库连接上；
- 调用ExecuteNonQuery方法；
- 关闭连接。

其中第二步到第四步的实现如下：

```
string cmdStrFormat = "insert into msg set name = '{0}',msg = '{1}';";
string cmdStr = string.Format (cmdStrFormat, conn.GetAddress (), str);
MySqlCommand cmd = new MySqlCommand (cmdStr, sqlConn);
try
{
	cmd.ExecuteNonQuery();	
}
catch(Exception e) 
{
	Console.WriteLine ("[数据库]插入失败 " + e.Message);
}
```

（2）获取数据

除了将新消息放入数据库中，我们还需要查看数据库中现有的历史记录，因此给出一个服务器的特定指令端口，服务器接收到对应指令后，进行获取数据的操作。

获取数据并简单处理，后通过socket返送给客户端。

这里我们将接口设定为“_GET”

4.运行效果

（1）发送若干数据后，发送“_GET”命令，可以获取当前数据库中的所有条目。

![img](http://ohn6qfqhe.bkt.clouddn.com/unityN3-1.png)

（2）服务器的状态

![img](http://ohn6qfqhe.bkt.clouddn.com/unityN3-2.png)

（3）数据表中的状态

![img](http://ohn6qfqhe.bkt.clouddn.com/unityN3-3.png)

比较遗憾的是，Unity对中文的支持不算太好，上面乱码的数据本身应该是“你好”，网上大多数解决方法都是解决脚本中文乱码的，但没有解决工程内中文乱码的。好在这次项目只是记录物体坐标，对语言没有什么限制。

 

《Unity3D 网络游戏实战》罗培羽·著

以及若干数据库的使用笔记。

------

明日目标：理解小型网络游戏服务端框架。
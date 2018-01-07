---
title: "【Unity】网络游戏编程实践（2）多人聊天小应用"
layout: post
excerpt: "编写异步Socket程序，理解多线程，构建简单的多人聊天应用。"
categories: 做笔记
tags:
  - Unity
  - Network
---

编写异步Socket程序，理解多线程，构建简单的多人聊天应用。

#### 1.连接器类(Conn)

上一节中讲到，服务器和客户端进行通讯时，需要通过socket这样的接口进行数据传输。当处理多个客户端的时候，就需要多个这样的连接器了。因此下面定义连接器类：

成员：

- socket:与客户端之间的套接字，在上一篇文章中介绍过。
- BUFFER_SIZE:缓冲区大小
- readBuff:读缓冲区
- buffCount:当前缓冲区的长度
- isUse:标识当前连接器接口是否被使用

方法：

- Init():初始化
- BufferRemain():缓冲区剩余字节数
- GetAddress():获取客户端地址
- Close():关闭连接

#### 2.连接池

通过建立一个连接池及一套连接使用、分配、管理策略，使得不必每次新建连接都要生成接口实例。连接池即为上面连接器类的数组。

连接池使用了固定数组，会占用更多内存，相对省去了开辟连接池的时间，是一个用空间换时间的策略。

#### 3.服务端程序架构

启动：Start():Socket->Bind->Listen->BeginAccept

接收客户端连接：BeginAccept():获取连接池中的连接请求->对连接请求执行BeginReceive()-> 重新查找连接池BeginAccept(回调)

接收客户端数据：BeginReceive():处理数据->若断开则调用Close()->继续接收数据BeginReceive()(回调)

**处理数据：广播**

将来自某一个客户端的数据以循环的方式，广播给所有连入服务器的客户端（包括当前客户端），这就是多人聊天应用的核心部分。

```
//广播
for(int i = 0; i<conns.Length;i++)
{
	if(conns[i]==null)
		continue;
	if(!conns[i].isUse)
		continue;
	Console.WriteLine("将消息传播给 " + conns[i].GetAddress());
	conns[i].socket.Send(bytes);
}
```

#### 4.客户端

主要包括：连接Connection()/接收消息回调ReceiveCb()/发送消息Send()。

- 连接：清理消息框->Socket->BeginReceive()
- 接收回调：处理收到的数据格式->显示->继续接收回调
- 发送消息：处理数据格式->通过Socket发送

#### 5.测试

将项目发布，打开服务器，再打开多个exe。

![img](http://ohn6qfqhe.bkt.clouddn.com/2-1.png)

聊天效果：

![img](http://ohn6qfqhe.bkt.clouddn.com/2-2.png)

服务器：

![img](http://ohn6qfqhe.bkt.clouddn.com/2-3.png)

参考资料：《Unity 3D 网络游戏实战》罗培羽 著

------

明日目标：Unity+MySQL
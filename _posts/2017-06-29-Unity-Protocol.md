---
title: "【Unity】网络游戏编程实践（5）协议"
layout: post
excerpt: "服务端框架中重要的一个模块，单独开一节详谈。"
categories: 做笔记
tags:
  - Unity
  - Protocol
  - Network
---

服务端框架中重要的一个模块，单独开一节详谈。

上一节中提到，所有协议都从字节流中读取消息，又把某种形式的消息转换为字节流发送出去。

所以，只要定义好编码解码的接口，便能够支持多种协议。而一套通用的服务端框架，要支持不同的游戏使用的各种协议格式。下面先从一些基本知识讲起。

####1.粘包分包现象及其处理

由于TCP协议本身的机制，客户端和服务器会维持一个连接发送数据。如果发送的网络数据包大小，TCP则会合并较小的数据包再发送；类似地，如果发送的数据包太大，TCP有可能会将它拆分成多个包发送，接收端的一次Receive可能只收到一部分数据。这便是粘包分包现象。

这里使用一种比较好理解的处理方法，即在每个数据包前面加上长度字节。每次接收到数据后，先读取长度字节，如果缓冲区的数据长度大于要提取的字节数，则取出相应的字节，否则等待下一次数据接收。

####2.协议基类

我们将一次发送的完整数据称为消息，将消息内容称为协议内容。由上文所述，一条消息的前四个字节用于粘包分包处理消息长度，随后才是消息内容。

服务端框架使用了两层协议：一层用于处理底层网络，另一层用于格式化数据。

协议基类ProtocolBase定义协议接口，其他协议类型必须继承ProtocolBase。通用的接口包括编码、解码、获取协议名称和用于打印输出的描述。

```
//协议基类
//规定协议中编码、解码等方法的格式
public class ProtocolBase
{
	//解码器，解码readbuff中从start开始的length字节
	public virtual ProtocolBase Decode(byte[] readbuff, int start, int length)
	{
		return new ProtocolBase ();
	}

	//编码器
	public virtual byte[] Encode()
	{
		return new byte[] { };
	}

	//协议名称，用于消息分发
	public virtual string GetName()
	{
		return "";
	}

	//描述
	public virtual string GetDesc()
	{
		return "";
	}
	public ProtocolBase ()
	{
	}
}
```

####3.字符串协议

这种协议即是上一篇文章中所实现的，特点是直观、简单。我们规定字符串协议的形式是“参数1 参数2 参数3…”我们规定参数1是协议名称，且各个参数之间用空格隔开。但缺点是，客户端若发送了含有空格符的字符串，就会引起混淆。

```
//基于字符串的协议
//形式 名称 参数1 参数2 参数3
public class ProtocolStr : ProtocolBase
{
	//传输的字符串
	//整个协议都用字符串表达
	public string str;

	//解码器
	//解码过程便是将字节流转换成字符串
	public override ProtocolBase Decode(byte[] readbuff, int start, int length)
	{
		ProtocolStr protocol = new ProtocolStr ();
		protocol.str = System.Text.Encoding.UTF8.GetString (readbuff, start, length);
		return (ProtocolBase)protocol;
	}

	//编码器
	//编码过程便是将字符串转换为字节流
	public override byte[] Encode()
	{
		byte[] b = System.Text.Encoding.UTF8.GetBytes (str);
		return b;
	}

	//协议名称
	//获取第一个空格前的字符串，即为协议名称
	public override string GetName()
	{
		if (str.Length == 0)
			return "";
		return str.Split (' ') [0];
	}

	//描述
	//用字符串代表协议描述
	public override string GetDesc()
	{
		return str;
	}

	public ProtocolStr ()
	{
	}
}
```

####4.字节流协议

字节流协议是一种最基本的协议。它把所有参数放入byte[]结构中，客户端和服务端按照约定的数据类型和顺序解析各个参数。本节编写的字节流协议支持整型、浮点型和字符串三种数据类型。我们规定协议的第一个参数必须是字符串，它代表协议名称。

这种协议的好处是，只要客户端和服务端按照约定好的类型进行读取，总能够正确地解析出各个参数。

####5.心跳协议

正常情况下，断开TCP会经历四次挥手。然而如果客户端电脑死机，或者网线被拔出，四次挥手便无法完成。服务器在同一时间能够接入的客户端数量是有限的，如果出现太多这样的死连接，新连接便无法接入。

为了防止出现上述情况，引入了心跳机制。心跳机制规定客户端每隔一定时间要给服务器发送一个特定信号，服务器会记录客户端最后一次发送心跳信号的时间，如果相隔太久，便认为客户端连接已经断开，于是主动断开连接。

 

参考资料：《Unity3D 网络游戏实战》罗培羽·著

------

明天是这周实训的最后一天，下周就要开始答辩。希望能够把能做的模块都做好。

明日目标：完成卡牌部分的数据库更新。
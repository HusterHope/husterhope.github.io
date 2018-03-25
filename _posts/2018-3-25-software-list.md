## Mac上安装Linux的后续

mac分区-U盘制作启动盘-装Deepin（Linux系统）-Linux无线网卡bug-返回mac-尝试换个linux系统-合并mac分区-合并分区出bug（linux的空间不见了）-尝试重启-重启gg

![](http://ohn6qfqhe.bkt.clouddn.com/macover-1.jpg)

想了想原因，应该是没有格式化分区的情况下强行合区，导致两个分区的头部在硬盘中重合，linux和mac的启动第一条语句互相冲突。因此不但60G空间找不到了，还导致存储器中的数据一起没了..如果再来一次肯定要先格式化再合区（然而只是猜想，必然不想再试一次了）

但这暴露出mac的磁盘保护措施很弱...这么看的话，想毁掉一台mac的数据非常简单了：

**给它分个区，再随便装个开源系统，再切回来合区，再重启，over。**

千万别试，试之前备份好数据，以及若干官方售后维修费。

---

裸机重装备忘：

* 网络：Chrome、ShadowsocksR、Aria2


* 社交软件：QQ、Wechat
* 开发者工具：Xcode、Git、Github desktop、PyCharm、Unity
* 办公软件：Pages、Numbers、Keynote、Microsoft Office大礼包
* Adobe大礼包：Photoshop、Premiere、DreamWeaver
* 辅助工具：Clean my Mac
* 其他想起来再补..


永远不想把系统再搞崩一次了，但愿我永远用不到这个list。

在今天裸机装机的时候，看到之前自己整理的配环境相关的笔记，重新在裸机上装起来的时候，仿佛跟过去的自己进行了时空握手，被自己感动了。

谢谢你啊。
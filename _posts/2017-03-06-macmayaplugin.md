---
title: "MacOS配置Maya 2017插件环境"
layout: post
excerpt: "前后折腾了4个小时，终于算是把这个环境彻底配出来了，这里记录一下踩坑过程。"
tags:
  - Maya
  - Environment
  - MacOS
categories: 解问题
---



前后折腾了4个小时，终于算是把这个环境彻底配出来了，这里记录一下踩坑过程。

### 1.准备阶段

首先，这种环境的东西肯定是要参考官方文档和安装包的：

> 官方文档教程：<http://help.autodesk.com/view/MAYAUL/2017/ENU/?guid=__files_Setting_up_your_build_environment_Mac_OS_X_environment_htm> 可以和我这份博文对比看（大神可以绕路看官方文档了）
>
> 官方安装包（mac 2017版）：<https://apps.autodesk.com/MAYA/en/Detail/Index?id=6303159649350432165&appLang=en&os=Mac>

官方安装包的下载速度是比较慢的，这里我还是放一个百度盘的共享链，自取。

> 链接: <https://pan.baidu.com/s/1jHLFV2A> 密码: fbmh

得到安装包后，会发现一个名为Maya2017_DEVKIT_MacOSX.dmg的文件。

在你的用户根目录($HOME)下创建名为devkitBase的文件夹，打开刚才的.dmg文件，把内部的四个文件全部拖入devkitBase中。



![img](http://ohn6qfqhe.bkt.clouddn.com/mos1.png)

下面把devkit 和 mkspecs这两个文件夹拷贝至/Applications/Autodesk/maya2017目录中（这一步我当时没注意把四个文件夹都拷过去了，还好没出什么事）

 

![img](http://ohn6qfqhe.bkt.clouddn.com/mos2.png)

### 

### 2.环境检查

编译插件和应用程序必须保证电脑具有以下环境：

（1）**系统版本10.9.5以上**（左上角的苹果标志菜单->关于本机）

 

![img](http://ohn6qfqhe.bkt.clouddn.com/mos6.png)

 

（2）**Xcode版本6.1.1以上**（打开Xcode，左上角Xcode->About Xcode，应该装了的都比这个版本高吧…

 

![img](http://ohn6qfqhe.bkt.clouddn.com/mos7.png)

 

（3）**SDK版本号10.9以上**

具体在这个目录下查看

/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs

（Xcode右键->显示包内容，进入后面的路径。这里请**记下这个版本号**，后面会用到）

（4）**clang和libc++**（貌似Xcode自带的就有？要是这一步卡着了在评论区说一声）

（5）**推荐装个「vim」**，方便修改文本文件

### 3.使用makefile进行环境配置

个人对Xcode开发用的很少，设置什么参数经常找不到在哪个窗口里，干脆直接用文档里说的makefile来做了，想着实在不行把makefile和buildconfig的代码给读一遍，有错也不怕（结果还真是把代码都读了不止一遍才配好环境的，一路都是坑QAQ

#### （1）设置maya位置的环境变量

打开终端（Terminal），因为我的终端是bash(其他类型的终端这一步请按照官网文档操作)，所以在根目录下($HOME)执行以下指令：

```
export MAYA_LOCATION=/Applications/Autodesk/maya2017/Maya.app/Contents
```

再用vim打开 .profile 文件，把上面的代码加入文件中（这样以后就不用每次都设置路径了）。

```
vim .profile
```

> vim下编辑文本需要先按下「i」键，输入完毕后先按ESC退出编辑模式，再输入 :wq 保存并退出vim。



![img](http://ohn6qfqhe.bkt.clouddn.com/mos8.png)

 

#### （2）创建使用makefile的文件夹和文件

> 官方表示：因为用户通常没有在应用程序目录内的写权限，所以我们把内容提取出来，在外部编译。

```
mkdir -p $HOME/devkit/plug-ins
cd $HOME/devkit/plug-ins
cp -r /Applications/Autodesk/maya2017/devkit/plug-ins/* .
```

在终端内依次执行上述三条指令。

第一条指令是在根目录下创建devkit文件夹，并在devkit文件夹下创建plug-ins文件夹。

第二条指令是让当前目录从HOME进入刚才创建的plug-ins文件夹。

第三条指令是把maya2017/devkit/plug-ins目录下的文件全部拷贝到当前文件夹（注意复制这行指令时不要漏掉最后一个小英文句号）。

> 第三步可能会出现无权限的情况，只需要在指令前面加上sudo(空格)即可，密码是开机密码。

这样我们就得到了如下文件目录和内容：

 

![img](http://ohn6qfqhe.bkt.clouddn.com/mos9.png)

#### 

#### （3）DEVKIT_LOCATION和MAYA_LOCATION

这一步我被卡了很久，直到读懂了每种报错的产生原因..

打开 $HOME/devkit/plug-ins 下的buildconfig，使用文本编辑工具打开就行，如果存在权限问题或者已锁定，可以先拷贝一个副本，删掉原来的，再把副本重命名成原来的名字，就可以开始更改了。

- 把文件头的MAYA_LOCATION改成前面设置的路径，并删除此行的「#」（让该行由注释变成代码）

```
MAYA_LOCATION=/Applications/Autodesk/maya2017/Maya.app/Contents
```

- 在下方加入DEVKIT_LOCATION的值(**注意把myDocuments替换成你自己的Home名称，下同**)

```
DEVKIT_LOCATION = /Users/myDocuments/devkitBase
```

开头改完效果如下图：

![img](http://ohn6qfqhe.bkt.clouddn.com/mos10.png)

 

**（4）准备make**

- 修改sdk版本号

搜索「command+f」buildconfig中的macosx字样，找到最后一条记录，它的后面紧跟着的就是sdk版本号，改为我们前面记下的跟自己电脑一样的sdk版本号。

 

![img](http://ohn6qfqhe.bkt.clouddn.com/mos11.png)

 

- 安装命令行工具（为后面的make命令做准备）

```
xcode-select --install
```

以上全部没有问题后，就可以开始执行下面的命令了。祝顺利:)

**（5）MAKE！**

到了整个过程最惨烈的一步了，如果你前面全部按照我的过程做了下来，应该会一切顺利。

先找到Makefile文本文件，把后缀改成txt(方便等下让终端找到它)。

然后执行下面的指令。

```
sudo make -f Makefile.txt clean
```

输入密码后执行，你就能看到终端刷着一串warning了，只要没有error就都ok。

![img](http://ohn6qfqhe.bkt.clouddn.com/mos12.png)

等待大约2分钟后进程结束，如果没有任何错误信息(errors)，则继续执行：

```
sudo make -f Makefile.txt
```

同样会闪一堆warning，再等待大约2分钟，进程正常结束，胜利就在眼前！

### 4.将make生成的文件分类存放

#### （1）新建文件夹

还是在刚才的plug-ins文件夹下，终端执行

```
mkdir plug-ins scripts icons
```

建立三个名称分别为plug–ins、scripts、icons的文件夹。

#### （2）文件转存

终端依次执行：

```
cp */*.bundle plug-ins
cp */*.{mel,py} scripts
cp */*.{png,xpm,cur,rgb} icons
```

### 5.重写Maya.env文件

打开finder，按下「Command+Shift+G」(前往文件夹)，在弹出的对话框中输入

/Users/~~myDocuments~~/Library/Preferences/Autodesk/maya/2017/

就可以看到一个名为Maya.env的文件（要是上述目录下没有这个文件，就自己新建一个空白文本文件，把后缀改成.env，放到刚才的目录下）

在Maya.env中输入以下内容：

MAYA_PLUG_IN_PATH = $HOME/devkit/plug-ins/plug-ins
MAYA_SCRIPT_PATH = $HOME/devkit/plug-ins/scripts
XBMLANGPATH = $HOME/devkit/plug-ins/icons/%B

![img](http://ohn6qfqhe.bkt.clouddn.com/mos13.png)

存储所作的修改，并把Maya.env复制一份放入上级目录（/Users/~~myDocuments~~/Library/Preferences/Autodesk/maya/）中

![img](http://ohn6qfqhe.bkt.clouddn.com/mos14.png)

有关env文件的作用可参考官网：

> <http://help.autodesk.com/view/MAYAUL/2017/ENU/?guid=__files_GUID_8EFB1AC1_ED7D_4099_9EEE_624097872C04_htm>

### 6.验收

上述操作都结束后，我们打开Maya 2017，进入Windows->Settings/Preferences->Plug-in Manager，就能够看到我们刚刚安装好的插件了。

![img](http://ohn6qfqhe.bkt.clouddn.com/mos15.png)

maya插件编程之旅就此开启！

------

后记：博主在网上各个地方搜相关配置教程，一方面没有中文的教程，另外也没有比官方文档更详细的说明，而官方文档本身还有一堆坑（makefile针对版本的兼容性不好、配置需要手动修改的太多、表达不准确等）。在这里记完踩坑的过程是为了给以后有需要的朋友提供一个快速配置的参考。



有任何问题欢迎在评论区或留言板反馈。
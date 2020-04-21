---
title: "【Pico Neo2】VR一体机Locomotion实现记录"
layout: post
categories: 解问题
tags:
  - VR
  - Unity
  - Locomotion
---

因项目和宅家工作需要，入手VR一体机进行相关开发。

正好项目牵头方用了Pico系列的一体机，遂也借助项目经费入手一台宅家开发测试。

<!-- more -->

开箱图：

![](https://github.com/HusterHope/blogimage/raw/master/20200421-3.png)

下面进入正题。

官方API说明文档：[中文](http://static.appstore.picovr.com/docs/sdk/cn/chapter_one.html)/[ENG](https://github.com/wearvr/pico-vr-unity-sdk-instructions)

> 如确实有开发需求，个人更推荐参考英文站，一方面开源代码+说明文档兼备，另一方面细节程度更深。

本文讲的核心需求是Locomotion，即VR大规模场景中的位置移动功能。

以往在PC-VR设备中实现相关功能时，基本可以借助SteamVR+VRTK的框架，直接调用现成的工具包，而对于这个上个月刚上市的一体机而言，除了开发文档之外并没有太多能够参考的材料，于是花了些时间摸通里面的玄机。

首先需求如标题所说，开发Locomotion的功能。

一般来说手柄的TouchPad部分是进行Locomotion的首选，而Pico Neo2这个手柄的TouchPad直接变成了游戏操纵杆的样式，因此考虑使用操纵杆控制摄像机的位置移动。

## 输入映射

Unity里面根据输入的横纵轴向，能够很方便地改变一个对象的位置。默认的方向输入是WASD和四个方向键。

```c#
inputH = Input.GetAxis("Horizontal");
inputV = Input.GetAxis("Vertical");
transform.Translate(inputH,0,inputV);
```

这里的值为[-1,1]区间，而Pico Neo2的TouchPad为[0,255]区间，因此需要写一个简单的输入映射，将[0,255]转化到[-1,1]区间内。

```C#
private Vector2 TouchpadMapping(Vector2 touchPosition)
{
	if(touchPosition[0] == 0 && touchPosition[1] == 0)
	{
		// Not touched
		return touchPosition;
	}
	touchPosition[0]=touchPosition[0]/127.5f - 1;
	touchPosition[1]=touchPosition[1]/127.5f - 1;
	return touchPosition;
}
```

这里还有一个神奇的小坑，手柄操纵杆的横向为Y，纵向为X（图自[Pico VR Unity SDK硬件产品开发指南](http://static.appstore.picovr.com/docs/sdk/cn/chapter_five.html)）

![](http://static.appstore.picovr.com/docs/sdk/cn/_images/5.4.png)

于是控制位置变化的语句为：（完整代码见后文）

```C#
//注意这里的坐标映射关系：Unity中Y为上方，X为左右向，Z为前后向，因此映射关系是(touchY,0,touchX)
transform.Translate(touchY,0,touchX);	
```

## 地形和追踪

如果仅仅控制相机的前后左右移动，那么上面的代码就已经基本实现了功能。但VR场景也会包含一些起伏不平的地形。如果为了简单易实现，可以利用Unity GameObject里的【Capsule】创造一个随地形变化而起伏移动的虚拟角色，而虚拟角色的引入就会带来相机追踪的问题。

Pico VR的SDK已经自带了一个追踪目标`target`的脚本，核心API调用的部分在`Pvr_UnitySDKHeadTrack.cs`这个文件中。（详见[Github链接](https://github.com/wearvr/pico-vr-unity-sdk-instructions/blob/master/examples/PicoUnityVRSDKExample/Assets/Pvr_UnitySDK/Sensor/Pvr_UnitySDKHeadTrack.cs)）

简单观察后可以发现，这个脚本默认情况下是选择追踪用户头部的，如果`target`不为空，则追踪`target`。而我们希望实现一种更自然的方式：既让用户能够小范围行走（局部6自由度漫游），也可以通过手柄的控制进行更大范围的移动（Locomotion）。因此需要在追踪`target`和追踪头部之间切换。

> 这里有个插曲，最开始我是想把这两个功能合在一起做的，即追踪头部运动的同时控制Capsule的移动。但经过若干种尝试，发现并不能很稳定的实现，奇怪的效果比如相机漂移、Capsule不跟随头部移动而一起移动等。

观察到代码中，系统获取设备旋转和位置的接口分别为：

```C#
var rot = Pvr_UnitySDKManager.SDK.HeadPose.Orientation;
...
Vector3 pos = Pvr_UnitySDKManager.SDK.HeadPose.Position;
```

旋转的部分，在交互时自然只用追踪用户自身的头部旋转，因此统一一下就没问题了（如果加入手柄控制的旋转可能很容易眩晕）。

问题就出在这个`pos`接口上。经理解上下文代码和实际使用测试发现，`pos`变量读入的值其实是用户头部相对起始点产生的偏移值。同时`Pvr_UnitySDKManager.SDK.HeadPose.Position`接口并不支持写入，也就是不能将世界坐标直接用于改变头部位置。这也导致了每次使用默认方式更新用户头部坐标时，一定会回到起始点附近。

但是Unity中能够改变物体位置的方法千千万，既然没办法直接改变局部位置，那么可以从改变父级对象的世界位置入手。（下图为Pvr_UnitySDK的Prefab结构，上面的脚本片段出自`Head`对象）

![](https://github.com/HusterHope/blogimage/raw/master/20200421-1.png)

```
transform.parent.position = target.transform.position; //将Head的父节点的世界坐标与Target对齐
```

在了解头部追踪和Locomotion功能的区别和实现细节后，我们希望通过手柄上的一个按键，在两种追踪方式之间切换（手柄按键的相关接口一样可以参考[Pico VR Unity SDK硬件产品开发指南](http://static.appstore.picovr.com/docs/sdk/cn/chapter_five.html)或者Github文档）。比如获取左手柄的侧键点击情况：

```C#
if(Controller.UPvr_GetKeyDown(0, Pvr_KeyCode.Left)
{
    //do something
}
```

## 测试工程

到这里，基本上主要的坑已经过去了。搭建一个简单的工程测试一下：

![](https://github.com/HusterHope/blogimage/raw/master/20200421-2.png)

为方便有需求的同学测试功能，放上改动后的代码。

第一个文件是Head上挂载的`Pvr_UnitySDKHeadTrack.cs`：

```C#
// Discription:Main tracking,manage the rotation of cameras.Be fully careful of  Code modification
using UnityEngine;
using Pvr_UnitySDKAPI;
public class Pvr_UnitySDKHeadTrack : MonoBehaviour
{
    public bool trackRotation = true;
    public bool trackPosition = true;
    //public Transform target;
    public GameObject target;
    private bool updated = false;
    private bool dataClock;
    
    // Custom variables
        // isTraveling: Change the headposition in a large scale with touchpad.
    private bool isTraveling = false;
    public Ray Gaze
    {
        get
        {
            UpdateHead();
            return new Ray(transform.position, transform.forward);
        }
    }

    void Start()
    {
        target.SetActive(false);
    }
    void Update()
    {
        updated = false;
        UpdateHead();
        Locomotion();
    }

    // Change this method, let the headset only track target position
    private void UpdateHead()
    {
        if (updated)
        {
            return;
        }
        updated = true;
        if (Pvr_UnitySDKManager.SDK == null)
        {
            return;
        }
        if (trackRotation)
        {
            var rot = Pvr_UnitySDKManager.SDK.HeadPose.Orientation;
            transform.localRotation = rot;
            /*
            if (target == null)
            {
                transform.localRotation = rot;
                //player.transform.rotation = rot;
            }
            else
            {
                transform.rotation = rot * target.transform.rotation;
            }
            */
        }

        else
        {
            var rot = Pvr_UnitySDKManager.SDK.HeadPose.Orientation;
            if (target == null)
            {
                transform.localRotation = Quaternion.identity;
            }
            else
            {
                transform.rotation = rot * target.transform.rotation;
            }
        }
        if (trackPosition)
        {
            Vector3 pos = Pvr_UnitySDKManager.SDK.HeadPose.Position;
            //if(target == null)
            if (!isTraveling)
            {
                transform.localPosition = pos;
                //transform.position = transform.position + transform.rotation * pos;
            }
            else
            {
                transform.position = target.transform.position + target.transform.rotation * pos;
            }
        }
    }

    private void Locomotion()
    {
        if(Controller.UPvr_GetKeyDown(0, Pvr_KeyCode.Left))
        {
            if(isTraveling)
            {
                transform.parent.position = target.transform.position;
                //Pvr_UnitySDKManager.SDK.HeadPose.Position = target.transform.position;
                target.SetActive(false);
                isTraveling = false;
            }
            else
            {
                target.SetActive(true);
                target.transform.position = transform.position;
                isTraveling = true;
            }
        }
    }
}
```

第二个是挂载在`Capsule`上的自定义控制脚本，我们命名为`CharacterController.cs`，上文提到的输入映射已在这里详细实现，仅供参考。

```C#
using UnityEngine;
using Pvr_UnitySDKAPI;

public class CharacterController : MonoBehaviour
{
    public float speed = 10.0f;
    private float touchX = 0.0f;
    private float touchY = 0.0f;

    // Start is called before the first frame update
    void Start()
    {
        //transform.position = Pvr_UnitySDKManager.SDK.HeadPose.Position;
    }

    // Update is called once per frame
    void Update()
    {
        Vector2 touchPosition = Controller.UPvr_GetTouchPadPosition(0);
        touchPosition = TouchpadMapping(touchPosition);
        touchX = touchPosition[0]*speed*Time.deltaTime;
        touchY = touchPosition[1]*speed*Time.deltaTime;
        transform.Translate(touchY,0,touchX);
        transform.rotation = Pvr_UnitySDKManager.SDK.HeadPose.Orientation;
    }

    private Vector2 TouchpadMapping(Vector2 touchPosition)
    {
        if(touchPosition[0] == 0 && touchPosition[1] == 0)
        {
            // Not touched
            return touchPosition;
        }
        touchPosition[0]=touchPosition[0]/127.5f - 1;
        touchPosition[1]=touchPosition[1]/127.5f - 1;
        return touchPosition;
    }
}
```

至此，我们就能够通过点击手柄上的一个侧键，在Locomotion和局部追踪两种模式间切换了。其中Locomotion使用Pico Neo2的操纵杆实现。

---

文章的小尾巴/简要近况：

> 似乎又有好久没写博客了。
>
> 4月的状态可以说是起起伏伏，平均每周能够真正像实验室一样高效工作的日子大概只有3-4天，剩下的几天经常会波动。随着武汉逐步解封，返校和城市开放却仍然杳无音讯，于是一点点接受了这学期只能这样下去，同时必须严格要求自己的状态。
>
> 其实到了直博第二年下学期还没有在投的论文是有些沮丧的事，这也一定程度产生自我怀疑：究竟自己的水平在什么层次？做的工作是否有实际价值？博客上的内容究竟是有用的思考还是复读而已？
>
> 我试着思考自己提出的这些问题。
>
> 水平层次：论文投出去就知道了；
>
> 工作是否有价值：真正深入下去一定有价值；
>
> 博客内容：相比普遍意义上计算机的研究生来说，我的个人能力（特别是Coding相关）还是远远不够的。
>
> 好在现在正在发力做些能做的事，为第一个能投的论文打些基础。
>
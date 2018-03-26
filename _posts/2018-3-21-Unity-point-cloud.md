---
title: "【Unity点云预览】Point Cloud Free Viewer"
layout: post
categories: 解问题
tags:
  - Unity
  - PointCloud
  - 3D
---

点云是一些坐标系统中的一组数据点。在三维坐标系中，这些点通常由X，Y和Z坐标定义，并且通常用于表示物体的外表面。例如一个圆环的点云（摘自维基百科）：

![](http://ohn6qfqhe.bkt.clouddn.com/upc-1.gif)

当然，这是由计算机渲染之后的效果，一般情况下数据会以下面的方式存储：

![](http://ohn6qfqhe.bkt.clouddn.com/upc-2.jpg)

很明显人类比较喜欢第一种呈现方式，因为直观。如果找到一种方法让存储的纯数字列表转化为空间可见的结构，就能解决点云预览的问题。

<!-- more -->

---

运行环境：Unity 2017 2.0f3 personal

插件：[Point Cloud Free Viewer](https://assetstore.unity.com/packages/tools/utilities/point-cloud-free-viewer-19811)

### 操作流程

* 在Unity内部的`Asset store`里找到这个插件，下载并导入工程。


* 创建一个空对象，将脚本`PointCloudManager.cs`添加至此对象的组件。

![](http://ohn6qfqhe.bkt.clouddn.com/upc-3.jpg)

* 将脚本`EnablePointSize.cs`添加至`Main Camera`的组件。
* 在脚本`PointCloudManager.cs`的参数栏填写点云数据文件路径（路径从工程的`Asset`文件夹算起，利用插件自带的测试点云文件，此路径应填写为`PointCloud/xyzrgb_manuscript`），设置缩放系数（对于刚才的测试文件，这个值设置为0.25比较合适）


* 选择顶点着色器`VertexColor`。GameObject的组件设置如下：

![](http://ohn6qfqhe.bkt.clouddn.com/upc-4.jpg)

* 在Script文件夹内创建一个新脚本`CameraController.cs`，用来操控`Main Camera`的视角，便于浏览整个点云。一种简单的操控脚本如下（可以参考Unity官网文档获取更多操作方式）：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CameraController : MonoBehaviour {

	// Use this for initialization
	void Start () {

	}

	void Update()
	{
		var x = Input.GetAxis("Horizontal") * Time.deltaTime * 150.0f;
		var y = Input.GetAxis("Vertical") * Time.deltaTime * 150.0f;

		transform.Rotate(0, x, 0);
		transform.Rotate(y, 0, 0);
	}
}
```

将上面这个脚本也添加至`Main Camera`的组件。测试效果：

![](http://ohn6qfqhe.bkt.clouddn.com/upc-5.png)

---

### 测试结果

插件自带样例数据集为`.off`文件格式，共200万+个点，加载时间约为30秒，能够正常显示。

测试[Stanford-ply数据集](http://graphics.stanford.edu/data/3Dscanrep/)，插件未能识别。以下为脚本`PointCloudManager.cs`的部分源码，该脚本默认处理`.off`的点云数据，**不支持PLY格式**。但可以利用其他工具转换格式或手动修改插件的数据读取方式。

```c#
void loadPointCloud(){
		// Check what file exists
		if (File.Exists (Application.dataPath + dataPath + ".off")) 
			// load off
			StartCoroutine ("loadOFF", dataPath + ".off");
		else 
			Debug.Log ("File '" + dataPath + "' could not be found"); 
		
	}
```

使用[MeshConv](http://www.patrickmin.com/meshconv/)脚本，将ply格式转换为off格式，再次载入插件，能够正常运行。下面是`bunny`点云的运行效果。

![](http://ohn6qfqhe.bkt.clouddn.com/upc-6.jpg)

大功告成。

---


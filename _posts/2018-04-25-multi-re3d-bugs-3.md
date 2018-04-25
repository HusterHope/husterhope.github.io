---
title: "三维重建系统踩坑记录(3)-C/S"
layout: post
categories: 解问题
tags:
  - Re3D
  - SfM
  - MVS
  - PointCloud
  - Java
  - Android
  - Linux
  - Environment
---

PC端运行效果差不多了之后，开始部署移动端。思路是将采集和渲染放在移动端App中，重建过程的计算放在服务器上，双方传输必要的数据。

<!-- more -->

之前的踩坑记录参见：

[三维重建系统踩坑记录（1）——MacOS](http://leohope.com/%E8%A7%A3%E9%97%AE%E9%A2%98/2018/03/22/multi-re3d-bugs/)

[三维重建系统踩坑记录（2）——Linux](http://leohope.com/%E8%A7%A3%E9%97%AE%E9%A2%98/2018/04/11/multi-re3d-bugs-2/)

因为没什么移动端开发的经验，只好从手头上能用的环境和设备出发：

* 设备型号：Samsung S7 SM-G9350
* IDE：【客户端】Android Studio 3.1.1(Linux Version) /【服务器】IntelliJ IDEA Community Edition
* C/S架构参考：[3D_Scan_Project](https://github.com/ohza/3D_Scan_Project)
* 服务器流程参考：[linux下使用Bundler + CMVS-PMVS进行三维重建](https://blog.csdn.net/zouyu1746430162/article/details/78638133)
* 环境：【客户端】Java/Android 【服务器】Java/Netty/Apache


下载**3D_Scan_Project**，用Android Studio打开客户端工程目录（3D_Scan），用IntelliJ IDEA打开服务器工程目录（3D_Scan_Server）。

### 客户端修改

3D_Scan_Project这个App的整套流程比较完整，缺点是路径基本都写死了，以及交互上还是要作些修改。

首先是一些常量的修改，进入`ClientThreadService.java`，修改`SERVERPORT`和`SERVER_IP`，获取系统当前ip的命令为(Linux)：

```
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```

然后是采集的交互，这部分功能在`Fotolist.java`中。为了测试和使用的方便，将每次拍摄图片改为从相册选取图片，改监听器和返回值。

第一个`OnClickListener()`改为：

```java
neuesFoto.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {

                java.util.Date date = new java.util.Date();
                imageTime = date.getTime();

                Intent gallery = new Intent(Intent.ACTION_PICK, MediaStore.Images.Media.INTERNAL_CONTENT_URI);
                startActivityForResult(gallery, PICK_IMAGE);

            }
        });
```

改`onActivityResult`，为了获取文件名称这里使用了`Ringtone`变量。

```Java
if(resultCode == RESULT_OK && requestCode == PICK_IMAGE)
        {
            outputFileUri = data.getData();
            Ringtone r = RingtoneManager.getRingtone(this, outputFileUri);
            //String path = outputFileUri.getPath();
            imList_names.add(r.getTitle(this));
            dateTime.add(java.text.DateFormat.getDateTimeInstance().format(Calendar.getInstance().getTime()));
            Bitmap bitmap_scaled = Bitmap.createScaledBitmap(BitmapFactory.decodeFile(Environment.getExternalStorageDirectory().getPath() + "/DCIM/Camera/" + r.getTitle(this) + ".jpg"), 80, 60, true);
            imList.add(HelperClass.encodeTobase64(bitmap_scaled));
            adapter = new MyArrayAdapter(listContext, imList, dateTime);
            imListView.setAdapter(adapter);
            adapter.setNotifyOnChange(true);
        }
```

然后解决连接中的一些小问题，比如`NetworkOnMainThreadException`，主要原因是：

> 一个APP如果在主线程中请求网络操作，将会抛出此异常。Android这个设计是为了防止网络请求时间过长而导致界面假死的情况发生。[参考解决方案](https://blog.csdn.net/withiter/article/details/19908679StrictMode)

虽然不推荐使用StrictMode，但因为3D_Scan这个App只是在主线程中连接了一下Socket，所以不会存在太久延迟，为了方便直接在`Oncreate`函数中的`setContentView(R.layout.activity_main);语句之后加入：

```java
if (android.os.Build.VERSION.SDK_INT > 9) {
            StrictMode.ThreadPolicy policy = new StrictMode.ThreadPolicy.Builder().permitAll().build();
            StrictMode.setThreadPolicy(policy);
        }
```

可能还有其他小问题，但没记全。

### 服务器流程

因为原作者的服务器是部署在Windows系统下的，所以路径的写法和脚本的执行都很不一样。为了能完成端到端的传输，首先要解决不传输的情况下在本地跑通，再次推荐[这篇参考文章](https://blog.csdn.net/zouyu1746430162/article/details/78638133)，记录了从多视角图片到稠密点云的生成过程。使用的重建系统为Bundler+CMVS/PMVS。

改流程主要是让服务器代码不那么臃肿，原本作者开了三个线程依次执行重建过程，但用Linux的脚本来处理这些过程就十分简便（比如我将7步操作放在脚本`AllRec.sh`中）：

```Sh
cd /home/path/to/bundler/examples/BilderVonClient/

echo "--- Total Process: 1/7 done ---"

../../RunBundler.sh

echo "--- Total Process: 2/7 done ---"

../../bin/Bundle2PMVS prepare/list.txt bundle/bundle.out

echo "--- Total Process: 3/7 done ---"

sh pmvs/prep_pmvs.sh

echo "--- Total Process: 4/7 done ---"

../../bin/cmvs pmvs/

echo "--- Total Process: 5/7 done ---"

../../bin/genOption pmvs/

echo "--- Total Process: 6/7 done ---"

../../bin/pmvs2 pmvs/ option-0000

echo "--- Total Process: ALL DONE! ---"
```

其中运行第四步之前需要进入脚本手动修改变量`BUNDLER_BIN_PATH`。为了让过程比较顺畅，我们可以在Bundler的源代码中（路径`bundler/src/Bundle2PMVS.cpp`）修改该变量的默认值并重新执行`make`，这样输出脚本`prep_pmvs.sh`时就免去了手动修改的麻烦。

Java命令为：

```
p = Runtime.getRuntime().exec("/home/path/to/bundler/AllRec.sh");
```

目前这个脚本在终端下可以正常运行完成输出，但在Java下到第5步就不太正常，第6、7两步也有效果，但输出的`.ply`文件几乎为空。仍未解决。

下面是用终端脚本运行重建系统后的效果，共使用9张640*480的照片，内容为一个外星人小玩偶。点云数量为7626个。

![](http://ohn6qfqhe.bkt.clouddn.com/3drecon-bug-8.jpg)

---

这个Bug已经卡了两天依然不知道什么原因..只好把现在已经调通的部分记录一下。
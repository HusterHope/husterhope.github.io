---
title: "全景视频投影方式对比"
layout: post
categories: 解问题
tags:
  - OpenCV
  - 360Videos
  - Python
  - VR
---

球面和平面之间图像的映射问题，是一个从古时候起就一直困扰着地图测绘员的问题。时至今日，随着VR视频内容（也称全景视频/360度视频）的发展，又将这一问题摆在了研究人员面前。

与地图测绘所不同的是，视频流通常需要传输，这就涉及到带宽占用问题。人们一直以来不断探索如何在传输过程中尽可能保证视频内容的质量。本文对当下比较热门的几种映射方式作简单整理。

<!-- more -->

## 等距柱状投影

> Equirectangular projection，简称ERP

![](https://github.com/HusterHope/blogimage/raw/master/20190715-1.png)

这是一种沿用了约两千的方法，直观理解就是地球仪和世界地图之间的映射关系。

详细介绍见维基百科：[Equirectangular projection](<https://en.wikipedia.org/wiki/Equirectangular_projection>)

设视频宽度$w$，高度$h$，任意像素点$(x,y)$映射到球面后，坐标变为：

$$\phi = \frac{2\pi x}{w}, \theta = \frac{\pi y}{h}$$ 

其中，$\phi$和$\theta$分别对应经度和纬度。

这一方法的优势是简单且直观，便于直接播放和编辑。

与地图不同的是，视频的南北极内容（特别是北极，因为南极很多时候会成视频三脚架..）通常也包含许多有价值的信息。经过这种投影变换后，这两部分内容会被严重拉伸，且存在大量像素冗余。

## 立方体投影

> Cube Map Projection，简称CMP

![](https://github.com/HusterHope/blogimage/raw/master/20190715-2.png)

关于Cubemap的映射原理可以参考以前的一篇blog：[Python+OpenCV+Unity | 全景视频投影至立方体](<https://leohope.com/%E8%A7%A3%E9%97%AE%E9%A2%98/2017/12/14/P2Cube/>)

该方法有效解决了南北极投影畸变的问题，但引入了新的问题：从球体投影到立方体时，立方体角落的若干像素被拉伸。

![](https://github.com/HusterHope/blogimage/raw/master/20190715-3.png)

## 等角方块映射投影

> Equi-Angular Cubemap，简称EAC

这是Google于2017年提出的映射方式，可以看作是在Cubemap基础上的改进。

如上一节所述，传统的Cubemap映射在不同视角位置会分布不同数量的像素点，即相同角度的情况下，Cube上的长度发生改变，靠近赤道和南北极的部分对应了正方形的中心区域，投影区域较短，斜45度方向对应了正方形的边缘，投影区域较长。

EAC在Cubemap的结果之上，额外做一个映射，将原本长度不同的块映射为相同。

![](https://github.com/HusterHope/blogimage/raw/master/20190715-4.png)

数学推导部分不是特别复杂，由下图的映射关系给出。

![](https://github.com/HusterHope/blogimage/raw/master/20190715-5.png)

在代码实现时，由于opencv读取图像时，坐标中心在左上角，需要在变换前后各进行一次坐标变换，分别为：

$$p_x=\frac{j-\frac{c}{2}}{c}$$ 

$$p_y=\frac{\frac{r}{2}-i}{r}$$ 

$$q_u' = c*q_u$$ 

$$q_v'=r(1-q_v)$$ 

其中，$i,j$分别为遍历原始图像行和列的迭代器。为了防止图像中出现黑斑，实际会使用逆变换构造输出图像，即：

$$i=\frac{r}{2}[1-tan(\frac{\frac{1}{2}-\frac{q_v'}{r}}{2}\pi)]$$ 

$$j=\frac{c}{2}[1+tan(\frac{\frac{q_u'}{c}-\frac{1}{2}}{2}\pi)]$$ 

核心代码如下（in python）：

```python
def EAC(imgIn):
    row, col, ch = imgIn.shape
    imgOut = np.zeros((row, col, ch), np.uint8)
    for i in range(row):
        for j in range(col):
            uf = row*(1-tan((0.5-i/row)*pi/2))/2
            vf = col*(1+tan((j/col-0.5)*pi/2))/2
            u=floor(uf)
            v=floor(vf)
            if u >= row:
                u = u - 1
            if v >= col:
                v = v - 1
            (b,g,r) = imgIn[u,v]
            imgOut[i,j] = (b,g,r)
    cv2.imwrite('imgName.jpg', imgOut)

```

输出CMP和EAC的对比如下：

![](https://github.com/HusterHope/blogimage/raw/master/20190715-6.png)

---



部分图片和部分内容参考自 [Google AR AND VR - Bringing pixels front and center in VR video](https://blog.google/products/google-ar-vr/bringing-pixels-front-and-center-vr-video/)


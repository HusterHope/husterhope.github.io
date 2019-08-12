---
title: "【Unity】Shader初探"
layout: post
tags:
  - Unity
  - Shader
  - ShaderGraph
categories: 做笔记
---

现在可参考学习Shader的材料非常多，Unity官方也已经在去年推出了[Shader Graph](https://unity.com/cn/shader-graph)这种可视化编辑工具，使得Shader的创作难度降低了很多。本着从基础学起的姿势，优先从Shader的代码实现开始。

<!-- more -->

这里先推荐两个教程：

1. [Unity技术美术 - 零基础入门Shader](https://zhuanlan.zhihu.com/unityTA)

2. [Writing Your First Shader in Unity](https://learn.unity.com/tutorial/writing-your-first-shader-in-unity?language=en)

之前基本完全没摸过Shader，通过先看第一个专栏了解个大概，再跟着第二个教程动手实践，还是有不少收获的。更建议对英语没什么阅读障碍的同学从第二个教程入手。先看效果，然后看代码实现。

![](https://github.com/HusterHope/blogimage/raw/master/20190812-1.jpg)

## 第一个Shader源码解析

教程二的讲解主要分四个步骤：

1. 介绍`Unlit`类型Shader的代码结构；
2. 在片元着色器（frag）中加入自发光属性；
3. 在片元着色器（frag）中加入透明度属性；
4. 在顶点着色器（vertex）中加入glintch effect（如上动图所示）

教程中的关键笔记就直接写成代码注释了，理解起来比较方便（网页显示的代码高亮有点问题..建议使用VS Code作为IDE打开

```
// Top level: filename and editor menu place
// Language: ShaderLab - mix of two kinds of language.
//      High level: telling unity how we are going to render things.
//      CGPROGRAM: running on GPU
// This tutorial will not cover the [fog] part.
Shader "Unlit/Special/Cool Hologram"  // We can change the string here without changing the name of the file.
{
    // Public variables in Unity
    Properties
    {
        // Variable name, display name, and default value
        _MainTex ("Albedo Texture", 2D) = "white" {} // No semicolons!
        _TintColor("Tint Color", Color) = (1,1,1,1)
        _Transparency("Transparency", Range(0.0,0.5)) = 0.25
        _CutoutThresh("Cutout Threshhold", Range(0.0,1.0)) = 0.2

        _Distance("Distance", Float) = 1
        _Amplitude("Amplitude", Float) = 1
        _Speed("Speed", Float) = 1
        _Amount("Amount", Range(0.0,1.0)) = 1
    }
    // Instructors for Unity about how to set up the renderer
    //      Note that we could have several subshaders for different platform
    SubShader
    {
        //Queue: render order
        //  In order to render sth transparent over other things, we need to render other things first.
        //  Four kinds of render queues: Background / Geometry(default) / AlphaTest / Transparent / Overlay
        Tags {"Queue"="Transparent" "RenderType"="Transparent"}
        //Tags { "RenderType"="Opaque" }
        
        LOD 100 //level of detail.
        
        // More about Zwrite: https://docs.unity3d.com/Manual/SL-CullAndDepth.html
        Zwrite off
        // More about Blend: https://docs.unity3d.com/Manual/SL-Blend.html
        Blend SrcAlpha OneMinusSrcAlpha

        // Instructions for GPU
        Pass
        {
            CGPROGRAM
            // two functions
            #pragma vertex vert
            #pragma fragment frag

            // helper functions for render in Unity
            #include "UnityCG.cginc"

            // two structs pass to two functions
            struct appdata //pass to vert function
            {
                // four float numbers: x,y,z,w
                float4 vertex : POSITION; //semantic binding: tells shader how this is going to be used in rendering
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 vertex : SV_POSITION; //Screen space position
            };

            sampler2D _MainTex;
            float4 _MainTex_ST;
            float4 _TintColor;
            float _Transparency;
            float _CutoutThresh;

            float _Distance;
            float _Amplitude;
            float _Speed;
            float _Amount;

            // Vertex function
            //      Gets all the vertices of the model ready to be rendered,
            //      and convert from object space to clip space(relative to the camera).
            //      eg. modify the positions of vertices in the vertex function - glitch effects
            //      Pass the result to fragment function.
            v2f vert (appdata v) //pass in as a struct: just make it looks easier
            {
                v2f o; //vert to frag, what we are going to return
                // Not animation, just happening on vertex level.
                v.vertex.x += sin(_Time.y * _Speed + v.vertex.y * _Amplitude) * _Distance * _Amount; //_Time.y: in seconds
                // local space -> world space -> view space -> clip space -> screen space
                //      All in UnityObjectToClipPos function
                o.vertex = UnityObjectToClipPos(v.vertex);
                // Transform the texture
                o.uv = TRANSFORM_TEX(v.uv, _MainTex); //takes data from public variables.
                return o;
            }

            // Fragment function
            //      Paints in the pixels.
            //      Output to a render target. 
            fixed4 frag (v2f i) : SV_Target // bind to render target
            {
                // sample the texture
                // Add _TintColor: self-luminous 
                fixed4 col = tex2D(_MainTex, i.uv) + _TintColor; //color
                col.a = _Transparency;
                // The meaning of clip: if(col.r < _CutoutThresh) discard;
                clip(col.r - _CutoutThresh);
                return col;
            }
            ENDCG
        }
    }
}
```

## 后续学习路线

Shader Graph教程推荐：[Unity at GDC - Shader Graph Introduction](https://www.youtube.com/watch?v=NsWNRLD-FEI&t=3220s)

此外，不推荐【Unity learn】的教程[Make a Flag Move with Shadergraph](https://learn.unity.com/project/make-a-flag-move-with-shadergraph)，虽然整个过程讲的很细，但不像上面GDC的教程那样由浅入深，给人的感觉是学完了1+1马上动手做9*9...制作飘动的旗帜在另外一个博客的资源中有完整的ShaderGraph，然而作为初学者，基本没吃透：[Art That Moves: Creating Animated Materials with Shader Graph](https://blogs.unity3d.com/2018/10/05/art-that-moves-creating-animated-materials-with-shader-graph/)，以后再啃。
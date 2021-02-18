---
title: 我所理解的Unity-IK
date: 2019-05-28 15:39:52
permalink: the-inverse-kinematics-wat-i-understand

categories:
- Unity

tags:
- IK
---
在程序中人物模型出现很自然的运动的时候，大部分情况下我都觉得应该是动画的运动。由动作设计师准备好的动画，工程师将动画拆分组合这样的。然而实际上貌似并不是。

<!--more-->

## Root Motion
- [Unity - Manual: Root Motion - how it works](https://docs.unity3d.com/2018.3/Documentation/Manual/RootMotion.html)

在此之前，先介绍一下RootMotion。

- [Generic 动画中 Root Motion的概念和使用](https://blog.csdn.net/swj524152416/article/details/54973164)

## Inverse Kinematics
- [Unity - Manual: Inverse Kinematics](https://docs.unity3d.com/2018.3/Documentation/Manual/InverseKinematics.html)

反向运动学，这是中文翻译，一开始是懵的。

> IK和FK对应，正向运动学就是根骨骼带动节点骨骼运动。而反向运动学就是由子节点带动父节点运动。

- [游戏中反向运动学(ik)的研究与简介 - 风恋残雪 - 博客园](https://www.cnblogs.com/ghl_carmack/archive/2012/11/12/2765658.html)
话说这个大佬的文章，我应该常常关注的。
### IK使用
在上面提到的官方文档中有大致的使用方法。具体的使用之后整理


### VR - Final IK
这个算是主角了，这篇文章的目的就是熟悉并使用这个Asset。（好贵）

主要还是看人家的官方视频：
- [FINAL IK TUTORIAL](https://www.youtube.com/watch?v=7__IafZGwvI&list=PLVxSIA1OaTOu8Nos3CalXbJ2DrKnntMv6)

- [Final IK Document](http://www.root-motion.com/finalikdox/html/index.html)

- [Final IK の VRIK の Solver にある各値の説明](https://qiita.com/_karukaru_/items/b74bb5bdf08f5de32d0e)

这个东西怎么说呢，我试着用了用，方便是方便，源码是C#的，读起来也不是很费力。但是最后我也就是用了FinalIk里的InteractionSystem，其他的都放弃了。想要在自己的代码里引用他的脚本文件会出error，然后一大堆的初始化内容需要重写，在Unity的Editor里直接拖拽就很方便，自己在代码里写就各种null。



搜了一下感觉这篇很有意思，先保存一下。
- [UnityとHTC VIVEでバーチャルアイドルに変身（ATL客員研究員寄稿記事） | Advanced Technology Lab.](https://atl-hiroo.recruit-tech.co.jp/2018/01/unity_vive_virtual_idle/)


## Leap Motion


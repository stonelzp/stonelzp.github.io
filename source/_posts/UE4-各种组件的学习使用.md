---
title: UE4-各种组件的学习使用
permalink: ue4-components-learning
date: 2019-04-22 16:44:47

categories:
- UnrealEngine4

tags:
- UE4

---

内容太少我决定把这篇合并一下，记录一些UE4中的插件亦或者某个组件（Component）的使用方法等等。

<!--more-->

# Spline

偶然发现了一个很有趣的控件，**Spline Component**，感觉会很有用就记下来了。
- [Using Splines & Spline Components | Live Training | Unreal Engine](https://www.youtube.com/watch?v=wR0fH6O9jD8)

这是四年前的视频你敢信？！

记录一下这个视频里出现的常用的关于Spline的操作函数。

## Functions

1.GetNumSplinePoints

Store number of Spline points.

在Spline的Component中输入Get就能看到好多和其相关的函数。

2.AddSplinePoint


## Spline Mesh



## 在C++中使用Spline
### 从Actor中获取SplineComponent
```C++
AActor* Object;

USplineComponent * comp = Object->FindComponentByClass<USplineComponent>();

float splineLength = comp->GetSplineLength();
```

原本我是把这个Spline控件放在了一个Actor上面，然后更新DistanceAlongSpline的变量值来控制Actor在Spline上的位置，但是我却忽视了Actor移动的时候这个Spline也在移动，所以没有按照预期的进行。只有将Actor和spline分开，放到另外一个Actor上了。

# USpringArmComponent
我只要遇见的场合就是Camera的操控，作为CameraBoom(相机臂)来使用的。

[USpringArmComponent - This component tries to maintain its children at a fixed distance from the parent, but will retract the children if there is a collision, and spring back when there is no collision.](https://docs.unrealengine.com/en-US/API/Runtime/Engine/GameFramework/USpringArmComponent/index.html)

按照上面的描述说就是可以简单实现相机和目标之间有Collision的时候缩短相机臂，经常见到的那种效果。

还有一些那种相机与目标之间存在Collision或者Mesh的时候吧Mesh半透明化的使用，不知道要怎么实现。







# Gameplay Tag

- [UE4 Gameplay Tagを使ってゲームプレイ時のタグ管理をより扱いやすくする](http://unrealengine.hatenablog.com/entry/2017/02/21/220000)


# 其它
- [Everything you always wanted to know about Unreal Engine physics (but were afraid to ask)](http://www.aclockworkberry.com/unreal-engine-substepping/)

- [100 UE4 Tips and Tricks | Unreal Fest Europe 2019 | Unreal Engine](https://www.youtube.com/watch?v=zX0gilGIpRQ)

---
title: UE4中的TraceFunctions
date: 2019-06-19 13:02:38
permalink: ue4-trace-fnctions

categories:
- UnrealEngine4

tags:
- UE4
---

UE4中的Trace比我想象中的要重要，除了模拟交互之外，获取两点之间的距离，也可以起到非常大的作用。(未整理）

<!--more-->

## 如何知道物体离地面的高度？
这是我迫切想要知道的事情，搜了一下果然发现有人问了相同的问题。

- [How to obtain landscape height?](https://answers.unrealengine.com/questions/83065/how-to-obtain-landscape-height.html)

于是我就用Trace试了一下，完美。
## Trace Functions
[Trace Functions](https://wiki.unrealengine.com/Trace_Functions)

基本上就是人家这篇Wiki写的内容，但是`LineTraceSingle这个函数编译没有通过，我用了[UWorld::LineTraceSingleByChannel](https://docs.unrealengine.com/en-US/API/Runtime/Engine/Engine/UWorld/LineTraceSingleByChannel/index.html)
来替代了。

### LineTraceSingleByChannel

#### FCollisionQueryParams


### LineTraceSingleByObjectType

#### FCollisionObjectQueryParams

## Collision
关于UE4中的碰撞系统，是个很复杂的问题，在这里只是针对Trace需要的碰撞记记载。

### Collision Response Reference
- [Collision Response Reference](https://docs.unrealengine.com/en-US/Engine/Physics/Collision/Reference/index.html)



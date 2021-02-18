---
title: Unity-Performance提升
date: 2019-07-04 20:01:31
permalink: unity-performance
categories:
- Unity
tags:
- Performance
---

我知道这只是早晚的事儿。

<!--more-->

- [【CEDEC2018】一歩先のUnityでのパフォーマンス/メモリ計測、デバッグ術](https://www.slideshare.net/UnityTechnologiesJapan/unity-111054310)

## UI部分
大佬总是跟我说不要用Canvas，当然是指游戏运行中的某种情况，貌似公司的大佬对于UI的效率很是关心。因为我也不懂太多关于UI的渲染问题所以只能先记下。

- [Unity UI の最適化に関するヒント](https://unity3d.com/jp/how-to/unity-ui-optimization-tips)
- [Unite Europe 2017 - Squeezing Unity: Tips for raising performance](https://www.youtube.com/watch?v=_wxitgdx-UI&index=7&list=PLX2vGYjWbI0Rzo8D-vUCFVb_hHGxXWd9j)

上面的视频我看了两遍都没看完，中途都困得不行了...但是确实很重要，一定要整理。

### Unity中UI需要了解
摘取上面的视频中的内容

> **The Absolute Basics**
>
> - Canvases generate meshes & draw them
>   - Drawing happens once per frame
>   - Generation happens when "something changes"
> - "Something changes" = "1 or more UI elements change"
>   - Change one UI element, dirty the whole canvas
>   - Yes, one.


> **What's a rebuild? Theoretically...**
> 
> - Three steps:
>   - Recalculate layouts of auto-layouted elements
>   - Regenerate meshes for all *enabled* elements
>       - *Yes, meshed are still created when alpha=0!*
>   - Regenerate materials to help batch meshed

> **What's a rebuild? In Reality...**
>
> - Usually, all system are dirtied instead of individually
> - Notable exceptions:
>   - Color property
>   - fill* properties on Image

> **After all that...**
>
> - Canvas takes meshes, divides them into batches
>   - Sorts the meshes by depth (hierarchy order)
>   - *Yes, this involves a sort() operation!*
> - AllUI os transparent. No matter what.
>   - Overdraw is   a problem!


> **The Vicious Cycle**
> 
> - Sort means performance drops faster-than-linear as number of drawable elements rises.
> - Higher number of elements means a higher chance any given element will change.

> **Slice up your canvases!**
> 
> - Add more canvases.
> - Each canvas owns its own set of UI elements
>   - Elements on different canvases will not batch.
> - Main tool for constraining size of UI batches.

> **Nesting Canvases**
> 
> - Canvases can be created within other canvases
> - Child canvases inherit rendering settings.
>   - Maintain own geometry.
>   - Perform own batching.


后半段我开始听不太懂了......










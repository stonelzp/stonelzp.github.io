---
title: UE4中的WorldComposition场景构建
date: 2022-01-20 14:17:18
permalink: ue4-world-composition
categories:
- UnrealEngine4
tags:
- UE4
---

关于在UE4中的超大场景构建就不得不提WorldComposition特性，就我所理解的WorldComposition其实就是利用LevelStreaming功能来实现的。

<!--more-->

# Overvirew

# WorldComposition
- [官方文档- World Composition User Guide](https://docs.unrealengine.com/4.26/en-US/BuildingWorlds/LevelStreaming/WorldBrowser/)

WorldComposition不是立马就可以使用的，想要使用这个功能的话需要去`WorldSettings -> World -> EnableWorldComposition`开启。

开启之后会自动检测你当前加载中的Level的同级文件夹中的所有Level并将其自动加载到SubLevel。所以在开启的时候保证同级文件夹下没有其它的Level最好。


# 注意点

## Level Bounds
> The first time a level is loaded in World Composition, a new Level Bounds Actor is automatically created in the level. The Level Bounds Actor is used to calculate the size of the level.
>
>By default, the Level Bounds Actor automatically resizes to include all Actors found in the level. However, some Actors, like skyboxes, can have very large bounding boxes, which will result in overly large level tiles in the world minimap. If you have Actors like this that you do not want to include in the level bounds calculations, you can disable automatic level bounds calculations and set a fixed size for the Level Bounds Actor.

[官方文档](https://docs.unrealengine.com/4.27/en-US/BuildingWorlds/LevelStreaming/WorldBrowser/#minimap)

我是在移动Map到另一个工程的时候才注意到这个Actor，一开始我以为是没删干净的测试数据，删掉之后各种不妙之后才意识到这个Actor的功能。

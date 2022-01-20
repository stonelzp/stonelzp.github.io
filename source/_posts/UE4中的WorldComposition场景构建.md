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

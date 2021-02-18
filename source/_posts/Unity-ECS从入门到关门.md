---
title: Unity-ECS从入门到关门
permalink: unity-entity-component-system
date: 2020-02-12 18:16:23
categories:
- Unity
tags:
- Unity
- C#
- ECS
---

ECS(Entity Component System)是**Unity Data Oriented Tech Stack**的一部分，是其默认的高性能编码方式。

<!--more-->

也就是我经常看到的那个DOTS的一部分啦，其他部分的话暂时先不管（有点管不过来），这东西完全是一个新的概念，或者说试图用以前的Unity的GameObject，MonoBehavior的概念去理解的话，就会掉坑里，完全不知所云。拿面向对象的思想去套貌似也会吃点亏，当然我还没到这个地步只是听说的。

# Entity Component System
- [Entity Component System](https://docs.unity3d.com/Packages/com.unity.entities@0.5/manual/index.html)
> The Entity Component System (ECS) is the core of the Unity Data-Oriented Tech Stack. As the name indicates, ECS has three principal parts:
>
> - Entities — the entities, or things, that populate your game or program.
> - Components — the data associated with your entities, but organized by the data itself rather than by entity. (This difference in organization is one of the key differences between an object-oriented and a data-oriented design.)
> - Systems — the logic that transforms the component data from its current state to its next state— for example, a system might update the positions of all moving entities by their velocity times the time interval since the previous frame.

如何理解这个概念？看上面的英文。如果想要细致的理解，我觉得上面那个链接的内容都要看。

# EntityComponentSystem Samples
- [EntityComponentSystemSamples](https://github.com/Unity-Technologies/EntityComponentSystemSamples)

官方的GitHub例子，可以用来观摩学习。

# Blog
关于ECS的博文有一些很重要，或者说我是读了这些博文才逐渐能理解ECS的。
- [World, system groups, update order, and the player loop](https://gametorrahod.com/world-system-groups-update-order-and-the-player-loop/)

这里讲到了我在UniRx文章中提到的PlayerLoop的内容，有时间整理一下。



- [On DOTS: Entity Component System](https://blogs.unity3d.com/2019/03/08/on-dots-entity-component-system/)

Unity Blog中的一篇文章，我觉得Unity的官方博客里面有好多非常好的文章，时常关注一下没有坏处（我得看啊）。


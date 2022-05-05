---
title: UE4中的AnimationSystem理解
date: 2022-05-05 19:03:58
permalink: ue4-animation-system
categories:
- UnrealEngine4
tags:
- UE4
---

主要是用来记录遇见的关于AnimationSystem相关的知识。估计会是一篇很长的文章。

<!--more-->

# Overview
不光是UE4，也会包含UE5的内容。

官方文档：
- [Animation System Overview](https://docs.unrealengine.com/4.27/en-US/AnimatingObjects/SkeletalMeshAnimation/Overview/)

这个文章我很久以前看过，再看应该也会有新收获。
- [[CEDEC2018] UE4アニメーションシステム総おさらい](https://www.slideshare.net/EpicGamesJapan/cedec2018-ue4-111104578)

## AnimationNode的制作


- [Animation Node Technical Guide - Guide to creating new nodes for use within graphs in Anim Blueprints.](https://docs.unrealengine.com/4.26/en-US/AnimatingObjects/AnimationNodeTechnicalGuide/)
- [受け取ったポースを返すだけのアニメーションノードを作ってみた](https://qiita.com/go_astrayer/items/accf6772c38979ffecb5)
- [UE4 アニメ―ションノードの作成（ボーンのトランスフォーム）を試してみる](https://qiita.com/unknown_ds/items/b343a2ab56d77f1b97c5)
- [アニメーションノードをまとめたアニメーションノードを作ってみた](https://qiita.com/go_astrayer/items/eb8b2d249018ac4f472f)
- [【UE4】C++アニメーションノードを見よう見まねで作ったメモ](https://qiita.com/Rinderon/items/6c5351df66f82d16a6f3)


## 惯性混合
之前在调查惯性混合的时候查找了很多文章，UE4中提供了某种程度上的惯性混合支持，但是在播放Montage的时候就没有相应的支持了。UE5的版本添加了支持这一个特性，由于当时是使用了旧的UE4的版本，于是参考了UE5的源码对UE4引擎源码做了一些修改。
与此同时，也稍微对UE的AnimationSystem有了一些了解。

- [	Inertialization: High-Performance Animation Transitions in 'Gears of War'](https://www.gdcvault.com/play/1025331/Inertialization-High-Performance-Animation-Transitions)
- [慣性ベースドなアニメーションブレンド（解説編）](https://hogetatu.hatenablog.com/entry/2018/06/02/185613)
- [慣性ベースドなアニメーションブレンド（実装編）](https://hogetatu.hatenablog.com/entry/2018/06/10/232856)


# 尚未整理的文章
关于AnimationSystem的内容相当的多，<span style="color: red">以下的链接文章的内容都需要我整理。</span>
- [Editing Animation Layers - An idle animation is edited to create a new reload animation through Animation Layer Editing.](https://docs.unrealengine.com/4.27/en-US/AnimatingObjects/SkeletalMeshAnimation/AnimHowTo/LayerEditing/)
- [AnimDynamics - Describes how AnimDynamics can be used to provide physically based secondary animation.](https://docs.unrealengine.com/4.27/en-US/AnimatingObjects/SkeletalMeshAnimation/NodeReference/SkeletalControls/AnimDynamics/)

Control Rig真的很重要！！！
- [Control Rig - Rig and Animate characters in real-time using Control Rig.](https://docs.unrealengine.com/5.0/en-US/control-rig-in-unreal-engine/)

Advanced Locomotion System V4这个动画系统，由于是免费的，或许在制作动画的时候有些启发
- [Advanced Locomotion System V4](https://www.unrealengine.com/marketplace/en-US/product/advanced-locomotion-system-v1?lang=en-US)
- [UE4无缝动画系统 ALS V4 分析（Advanced Locomotion System V4） 拆解 文档 教程](https://www.bilibili.com/read/cv9226463)

- [UE4 SlotAnimationについて](https://qiita.com/unknown_ds/items/d06c7d8203c6420eac8b)
- [Play Slot Animation is not work](https://forums.unrealengine.com/t/play-slot-animation-is-not-work/417823)

- [Blend Nodes - Animation nodes that blend multiple animations together based on a set of criteria.](https://docs.unrealengine.com/4.26/en-US/AnimatingObjects/SkeletalMeshAnimation/NodeReference/Blend/)

- [Transition Rules - Guide to rules that govern State Machine Transitions](https://docs.unrealengine.com/4.26/en-US/AnimatingObjects/SkeletalMeshAnimation/StateMachines/TransitionRules/)

- [A deep look into animation framework in ue4](https://arrowinmyknee.com/2019/09/11/a-deep-look-into-animation-framework-in-ue4/)

关于SyncGroup具体的使用场景我还没有一个正确的概念。
- [Sync Groups - Sync Groups allow you to maintain the synchronization of animations of different lengths.](https://docs.unrealengine.com/4.26/en-US/AnimatingObjects/SkeletalMeshAnimation/SyncGroups/)

- [UE4动画系统的那些事（一）：UE4动画系统基础](https://zhuanlan.zhihu.com/p/62401630)

- [Using montage to blend between animation states](https://forums.unrealengine.com/t/using-montage-to-blend-between-animation-states/211360)
- [How get the animation blend weight of some state in C++](https://forums.unrealengine.com/t/how-get-the-animation-blend-weight-of-some-state-in-c/378915)

- [Docs on blend profile?](https://forums.unrealengine.com/t/docs-on-blend-profile/107004)

这篇文章我放在这里有些奇怪，实在是因为我找不到任何关于BlendProfile的UE4相关的文档，看能否触类旁通一下。
- [Animation Blend Profile in Unity](https://arrowinmyknee.com/2020/07/07/Animation-Blend-Profile-in-Unity/)
- [UBlendProfile - A blend profile is a set of per-bone scales that can be used in transitions and blend lists to tweak the weights of specific bones.](https://docs.unrealengine.com/4.26/en-US/API/Runtime/Engine/Animation/UBlendProfile/)

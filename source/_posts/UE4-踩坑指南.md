---
title: UE4-踩坑指南
date: 2019-10-28 18:10:04
permalink: ue4-fucking-moment
categories:
-  UnrealEngine4
tags:
- UE4
- error

---
在UE4中踩了不少的坑，这篇主要是用来介绍自己在使用过程中由于粗心或者知识浅薄而掉进去的大坑。
<!--more-->

# 关于蓝图
蓝图这个东西我看不顺眼好久了，方便那是非常方便，但是也不能只看脚下的路不是么，等到时间长了，写的多了，那变量维护起来就是爽到飞起了，前期轻松后期地狱。

## GetDisplayName
> - ブループリント周りで注意:
>
> 「Get Display Name」ノードは本番で使わないでください。エディタからのゲーム呼び出しとエディタなしのリリースモード等で挙動が異なります。

自己作死用了这个节点，而且还用这个取出来的数据作为判断条件，这不是找死么。

蓝图的话请使用**GetObjectName**节点。C++的话**GetName()**函数。

## Can not see the detail panel
在蓝图的使用中会出现看不到Component的Detail面板的问题，解决方案是重新做一遍继承操作，好像蓝图继承C++父类的时候会发生这种问题。

> There's a known bug that happens when you do this:
>
> Create C++ class,
>
> Create Blueprint based on it.
> 
> Add component in C++.
>
> BUG: Component doesn't have details in Blueprints!
> 
> Solution:
>
> un-parent, and then re-parent the BP to the C++ class.
>
> fixed!

解决来源： [Cant't see details in Blueprint](https://answers.unrealengine.com/questions/869223/cantt-see-details-in-blueprint.html)


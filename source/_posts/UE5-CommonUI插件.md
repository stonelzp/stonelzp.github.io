---
title: UE5-CommonUI插件
date: 2023-08-28 01:33:12
permalink: ue5-commonui-plugin
categories:
- UE5
- Plugin
tags:
- UE5
- CommonUI
---

CommonUI是UE推出的新的UI插件，本质上还是UMG那一套，相比于原来的基础系统，为我们做了一点使用上的简化。
<!--more-->

# Overview

# CommonUI-GuideLine
- [Design Guidelines - Guidelines for integrating CommonUI and using it in your project.](https://docs.unrealengine.com/5.2/en-US/design-guidelines-for-using-commonui-in-unreal-engine/)

上面是UE5.2版本的CommonUI插件的官方文档。在CommonUI相关的文档相对较少的现在，官方文档的内容就显得至关重要了。

> **Should I use CommonActivatableWidget or CommonUserWidget?**
> Not every CommonUI widget should be an Activatable Widget. A widget should only be an activatable widget if it needs to affect Input Routing on an on or off basis.
> 
> If the widget you are creating only needs to interact with CommonUI's Input Routing system to handle input by itself, consider creating a CommonUserWidget or a regular User Widget. CommonUserWidgets form the basis of many of CommonUI's classes, including UCommonButtonBase and UCommonTabListWidgetBase. Tooltips are a good example of a case where Activatable Widgets can be counterproductive since tooltips tend to quickly appear and disappear, don't need to forward input to child widgets, and don't need to seize input from the rest of your UI.
> 
> If the widget you are creating has multiple interactable children or needs to block input handling from the rest of your UI, then using an Activatable Widget is preferable. Pop-up windows and modal menus are good examples of this kind of behavior.


# Debug Tips

## Slate Debugger
- [Console Slate Debugger - A reference manual for the Console Slate Debugger tool, which helps users debug applications using the Slate UI framework.](https://docs.unrealengine.com/5.2/en-US/console-slate-debugger-in-unreal-engine/)

我怎么忘了，还有一个超级好用的工具，之前有用过的。

## 观察Log
可以使用`Log LogCommonUI Verbose`提高Log输入等级，观察CommonUI的日志。尤其重要的是Focus的移动。
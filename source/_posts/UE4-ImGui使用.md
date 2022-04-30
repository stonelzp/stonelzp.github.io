---
title: UE4-ImGui使用
permalink: ue4-imgui
date: 2020-11-18 12:53:02
categories:
- UnrealEngine4
tags:
- UE4
- ImGui
- Plugins
---
ImGui，或者说DearImGui，是一个非常优秀的使用C++构筑的Gui的框架。功能非常全面，很适合用来当作Debug的界面工具。更加让人激动的是在Github上有人公开了支持UE4和UE5的UnrealImGui插件。本篇就是主要介绍这个插件的使用。

<!--more-->
# Overview
由于是一种方便的Debug用的UI，所以需要在最后的Shipping打包的时候要全部摘除出去，所以在写代码的时候别忘了`#if !UE_BUILD_SHIPPING`。当然想在哪个配置里保留根据项目自由选择就是。

这篇文章主要是关注UnrealImGui插件如何在虚幻引擎上的使用。DearImGui的内容如果感兴趣直接参考Github[Dear ImGui](https://github.com/ocornut/imGui)

UnrealImGui的master分支已经更新最新的UE5的支持。
- [Unreal ImGui](https://github.com/segross/UnrealImGui)
- [【UE4】GUIフレームワーク「Dear ImGui」を使ってデバッグ・ツール用UIを作ってみよう！ < 導入・基本編 >](https://pafuhana1213.hatenablog.com/entry/UnrealImGui-HowTo-Basic)

# Unreal ImGui的使用

据说是可以远程实机操控imgui的插件，配合UnrealImGui一起使用的插件。也是需要调查使用方法的。
- [netimgui](https://github.com/sammyfreg/netImgui)


## ImGui各种控件的实现
- [ImGuiのWidgets](https://qiita.com/ousttrue/items/ae7c8d5715adffc5b1fa)


我在调查的过程中有一个很在意的东西，下面的这个文章里的一个叫cereal的东西：
> cerealとは、C++11用のjson, xml, binaryシリアライズライブラリです。

感觉可能以后会用到。
- [OpenGLやDirectXなGUIにimguiが最強すぎる](https://qiita.com/Ushio/items/446d78c881334919e156)
- [C++のcerealのシリアライズが快適すぎるやばい](https://qiita.com/Ushio/items/827cf026dcf74328efb7)
- [cereal - A C++11 library for serialization](https://github.com/USCiLab/cereal)

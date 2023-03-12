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
一些基础的想要深入使用的话最好要看的：
- [FAQ (Frequently Asked Questions)](https://github.com/ocornut/imgui/blob/master/docs/FAQ.md)
- [Dear ImGui: Using Fonts](https://github.com/ocornut/imgui/blob/master/docs/FONTS.md)

据说是可以远程实机操控imgui的插件，配合UnrealImGui一起使用的插件。也是需要调查使用方法的。
- [netimgui](https://github.com/sammyfreg/netImgui)

## ImGui的扩展资源
应该是别人使用ImGui的扩展，一定要看：
- [Awesome Dear ImGui](https://github.com/HankiDesign/awesome-dear-imgui)
- [Useful Extensions](https://github.com/ocornut/imgui/wiki/Useful-Extensions)

## ImGui各种控件的实现

### Widget控件
记录各种常用的控件（Demo里面的可能也会记录）。
1. ToggleButton
```
void ToggleButton(const char* str_id, bool* v)
{
    ImVec2 p = ImGui::GetCursorScreenPos();
    ImDrawList* draw_list = ImGui::GetWindowDrawList();

    float height = ImGui::GetFrameHeight();
    float width = height * 1.55f;
    float radius = height * 0.50f;

    if (ImGui::InvisibleButton(str_id, ImVec2(width, height)))
        *v = !*v;
    ImU32 col_bg;
    if (ImGui::IsItemHovered())
        col_bg = *v ? IM_COL32(145+20, 211, 68+20, 255) : IM_COL32(218-20, 218-20, 218-20, 255);
    else
        col_bg = *v ? IM_COL32(145, 211, 68, 255) : IM_COL32(218, 218, 218, 255);

    draw_list->AddRectFilled(p, ImVec2(p.x + width, p.y + height), col_bg, height * 0.5f);
    draw_list->AddCircleFilled(ImVec2(*v ? (p.x + width - radius) : (p.x + radius), p.y + radius), radius - 1.5f, IM_COL32(255, 255, 255, 255));
}
```
- [Toggle Button?](https://github.com/ocornut/imgui/issues/1537)
不知道为什么没加到ImGui的库里面去。

- [ImGuiのWidgets](https://qiita.com/ousttrue/items/ae7c8d5715adffc5b1fa)




我在调查的过程中有一个很在意的东西，下面的这个文章里的一个叫cereal的东西：
> cerealとは、C++11用のjson, xml, binaryシリアライズライブラリです。

感觉可能以后会用到。
- [OpenGLやDirectXなGUIにimguiが最強すぎる](https://qiita.com/Ushio/items/446d78c881334919e156)
- [C++のcerealのシリアライズが快適すぎるやばい](https://qiita.com/Ushio/items/827cf026dcf74328efb7)
- [cereal - A C++11 library for serialization](https://github.com/USCiLab/cereal)

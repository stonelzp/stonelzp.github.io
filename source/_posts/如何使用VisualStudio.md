---
title: 如何使用VisualStudio 
date: 2019-05-25 13:13:59
permalink: how-to-visual-studio

categories:
- 工具

tags:
- Plugins

descriptions: 记录一些自己在VS使用过程中遇到的问题或者是之前未遇见过的使用方法。
---

我觉得我应该好好钻研一下这个“宇宙第一”的各种用法了，好多意想不到的插件甚至是快捷键我都不知道。
<!--more-->

# VS中的DEBUG
如何使用VS进行Debug是非常重要的，致命的是我至今未能完全掌握，不要总是依靠Log的输出自己猜了。


## 在UE4中使用VS进行调试
首先要把UE4的源码下载，然后按照官方的指示操作生成VS的工程文件。

打开文件之后开始build，这里我遇见的一个问题是当我编译成功之后，不知道该怎么运行了。






## Plugins

### VsVim

### Productivity Power Tools

- [Productivity Power Tools 2017/2019](https://marketplace.visualstudio.com/items?itemName=VisualStudioPlatformTeam.ProductivityPowerPack2017)

具体的功能之后有时间整理一下。

## 关于注释
- [How to: Insert XML comments for documentation generation](https://docs.microsoft.com/ja-jp/visualstudio/ide/reference/generate-xml-documentation-comments?view=vs-2019)

主要是我想知道使用`///`来插入一段完整的段落注释的方法。

# 简单的设置VS

## Solution Configurations
使用VS的时候，尤其是我在使用UE4的时候可能会切换编译的模式，什么**Development Editor**, **Debug Editor**等等，又长又短，长了就看不清自己设置的什么模式。

就想让他变长，我说那个drop-down list。

- [Adjusting the width of “Solution Configurations” drop-down list in the Visual Studio toolbar](https://visualstudioextensions.vlasovstudio.com/2014/08/14/adjusting-the-width-of-solution-configurations-drop-down-list-in-the-visual-studio-toolbar/)

Tools -> Customize -> Commands -> Toobar -> 下拉找到Standard -> 找到Solution Configuration -> 右边的Modify Selection -> 把Width改成165

就OK了。


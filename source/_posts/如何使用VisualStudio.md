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

~打开文件之后开始build，这里我遇见的一个问题是当我编译成功之后，不知道该怎么运行了。~

### Build SolutionConfigutarions
关于UE4开发过程中对项目的编译构成的选择就有些讲究了。默认的就是`Development Editor`，这种默认的编译构成方式优化了许多的内容导致对项目进行Debug的时候会有些断点（BreakPoint）不管用或者无法看到数值，超级不方便。

参考资料:
- [UE4のコードをVisual Studioからビルドする際の推奨設定(定期更新)](https://qiita.com/4_mio_11/items/8b04879d5e0ebeed681a)

### UnrealVS - UBT编译&UE4热更新
- 2021/06/07 前来更新

看到上面自己写的，跟放屁没什么区别。UE4的源码编译参照官方提供的手顺一步一步来就行了。

这里介绍一下一个UE4的插件
- [UnrealVS Extension](https://docs.unrealengine.com/4.26/en-US/ProductionPipelines/DevelopmentSetup/VisualStudioSetup/UnrealVS/)

对应的版本是4.26

严格来说这一部分应该是放在其它UE4相关的文章中再介绍的，但是使用的话主要还是为VS服务的，想了想还是放到VS的部分了。

使用场景主要是针对每次编译好源码想要反映到项目中的时候就得重启UE4的编辑器，实在太费时间，使用这个UnrealVS的插件，可以提供UBT的编译和UE4的热更新(hot reload),这样就**不用一次次的重启UE4的Editor就能看到C++代码的更新内容了**。

下面介绍如何开启和使用这个插件。

#### UE4这边
首先需要先安装这个插件，去引擎的源文件这个路径下找到`Engine\Extras\UnrealVS\VS2019\UnrealVS.vsix`这个文件，点击安装，类似于给VS安装插件。

安装完毕，UE4这边就OK了。

版本根据自己的VS版本选择，我用的是2019专业版，就选择2019文件夹下的。

#### VS这边
成功安装好了之后打开VS，去**Extension->Manage Extension->Installed**下就可以看到我们安装好的VS插件：**UnrealVS**。

然后去**View->ToolBars->UnrealVS**将菜单栏显示，然后在**UnrealVS**菜单栏的最右边点击**UnrealVS ToolBar Options**，点开应该只有一个选项：**Add or Remove Buttons**，找到**Customize**。

这个时候我们就打开了**UnrealVS**的工具栏自定义界面，在**Commands**页面为UnrealVS的Toolbar添加新的Command。点击**Add Command->Extensions->Build Startup Project**。点击OK完成设置。

就这样在我们对源码进行编辑的时候想要查看工程的修改就直接点击上面的**Build Startup Project**等编译和HotReload完成就会反应到UE4的Editor中。

坏消息就是这个插件也不是什么万能的，函数内的修改倒是没什么，要是有序列化的属性(一般是指需要被显示到蓝图)被修改添加删除之类的操作，一般就算用它编译了，也不会反映到UE4Editor上，只有重启才会反映。轻则不反映，重则直接崩溃。

据我自己的体感就是**安全的范围是在.cpp文件中对函数内的内容修改**。

**一些小Tips**
1. 平常在SolutionExplorer右键工程可以找到**Set as Startup Project**，一般情况下第一次打开项目的时候就应该设置。去**Tools->Options->UnrealVS->General->Hide Non-Game Startup Projects**设置为True的话，那么菜单栏里只会显示**Startup Projects**。

2. 为上面我们添加的**Build Startup Project**添加快捷键：**Tools->Options->Environment->Keyboard**,**Keyboard Mapping Scheme**就随自己喜欢了，搜索“UnrealVS”就会找到`UnrealVS.BuildStartupProject`这个点击选择添加新的快捷键。在`Global`一行添加**Ctrl + Shift + Alt + B**，然后`Assign`，这样添加就完成了。

再就是UnrealVS提供的其他命令的使用方式我不是太明白，就是上面那个链接的其他内容。

## Plugins
这里稍微对自己使用VS插件的功能进行一些记录。

### VsVim
使用很方便，具体的文本之类的操作属于Vim的操作范畴，会在另外的文章记录

### VisualAssist
我觉得是UE4开发必不可少的插件，其它的都可以忍，但是UE4开发的时候VS默认的IntelliSence不能忍，这家伙实在是太慢了。

#### 快捷键篇
快捷键是使用的关键了，首先是去`Extensions -> VAssistX -> Help -> Keyboard Shortcuts`，确认一下已有的快捷键设置。

具体的描述可以参照官网，暂时不过多赘述，我只能说个个都是我用的上的，必须要记住的：
- [Top 10 features that improve productivity](https://www.wholetomato.com/learn/top10)


### Productivity Power Tools

- [Productivity Power Tools 2017/2019](https://marketplace.visualstudio.com/items?itemName=VisualStudioPlatformTeam.ProductivityPowerPack2017)

具体的功能之后有时间整理一下。

## 一些其它的VS功能记录

### 关于注释
- [How to: Insert XML comments for documentation generation](https://docs.microsoft.com/ja-jp/visualstudio/ide/reference/generate-xml-documentation-comments?view=vs-2019)

主要是我想知道使用`///`来插入一段完整的段落注释的方法。

### VS的Snippets
这个东西是我找类似的功能时候偶然发现的，真的别人不说我都不知道，这么重要的功能：**Visual Studio Snippets**

UE4也为其提供了许多Snippets方便我们使用，需要做的是

去VS的`Tools -> Code Snippets Manager -> Language(Visual C++) -> My Code Snippets -> Add`，这会允许你导入已经准备好的Snippets，由于UE4`\Engine\extras\VisualStudioSnippets`路径下已经有很多，可以直接导入。

至于如何制作自己的Snippet，参照那些例子就好了，有兴趣我再作记录。

- [Visual Studio Snippets](https://nerivec.github.io/old-ue4-wiki/pages/visual-studio-snippets.html)
- [VisualStudio UE4のCode Snippets登録について](https://papersloth.hatenablog.com/entry/2018/09/19/224110)

# 简单的设置VS

## Solution Configurations
使用VS的时候，尤其是我在使用UE4的时候可能会切换编译的模式，什么**Development Editor**, **Debug Editor**等等，又长又短，长了就看不清自己设置的什么模式。

就想让他变长，我说那个drop-down list。

- [Adjusting the width of “Solution Configurations” drop-down list in the Visual Studio toolbar](https://visualstudioextensions.vlasovstudio.com/2014/08/14/adjusting-the-width-of-solution-configurations-drop-down-list-in-the-visual-studio-toolbar/)

Tools -> Customize -> Commands -> Toobar -> 下拉找到Standard -> 找到Solution Configuration -> 右边的Modify Selection -> 把Width改成165

就OK了。

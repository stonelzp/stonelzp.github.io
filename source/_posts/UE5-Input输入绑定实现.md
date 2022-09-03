---
title: UE5-Input输入绑定实现
date: 2022-05-31 00:23:53
permalink: ue5-input-binding
categories:
- UnrealEngine5
tags:
- UE5
- UE4
---

记录一下关于UnrealEngine输入绑定的学习。意外的接触了UE很久，对输入的部分还是一知半解。
<!--more-->

# Overview
进入UE5之后，貌似EnhancedInput的插件变得相当的好用，使用的实际项目好像也逐渐多了起来。嘛，技术嘛肯定是要以新的优先，所以前半部分主要还是记录这个插件。
内容的话可能是UE4/UE5内容交叉着说，大致的实现部分我并没有感觉出来这两者的差别。

# EnhancedInput的使用
由于特殊原因暂时并不会使用这个插件，这个部分的更新暂时先搁置。

[Enhanced Input - Using the Enhanced Input Plugin to read and interpret user input](https://docs.unrealengine.com/4.27/en-US/InteractiveExperiences/Input/EnhancedInput/)
[UE4.26 Enhanced Inputについて](https://unrealengine.hatenablog.com/entry/2020/11/28/192500)
[【UE5】Lyra に学ぶ Enhanced Input](https://qiita.com/sentyaanko/items/dd4990d4aa0e84478b59)

# UE4中的输入绑定实现

这篇文章给出了一个UE4中实现输入绑定的一个比较好的方案，需要花时间总结一下。
[Arbitrary Key & Axis Binding in UE4 with C++](https://snardle.dev/posts/ue4-key-axis-binding/)

# 关于UE中的Input
尚未整理完毕：
[【UE4】入力イベントの順序、設定項目について](https://shuntaendo.hatenablog.com/entry/2019/06/12/200000)
[UE4 アクターの入力取得をコントロールする](https://katze.hatenablog.jp/entry/2016/07/21/185752)

这篇文章还有一个好处就是确认手柄的键位超级方便。
[Gamepad input process in UnrealEngine](https://baemincheon.github.io/2020/10/25/unreal-input-system-via-gamepad/)
在自己编写Actor的Component的时候想要加上input的事件，但是在UE4中加入这种事件不是那么简单的。这需要对Actor的层级关系有一些了解。

## InputProcessor
意外的可能会派上用场，比如说键盘手柄的Icon切换等等。
- [IInputProcessorによるキー入力処理について](https://shama-coo.hatenablog.com/entry/2020/03/20/182505)

# Input Latency
偶然发现这个功能，等有时间了的话可以整理再深入了解一下。
- [Low Latency Frame Syncing - Modifies the way thread syncing is performed to greatly reduce input latency.](https://docs.unrealengine.com/4.26/en-US/SharingAndReleasing/LowLatencyFrameSyncing/)

还有一篇解说原理的很有用的文章:
- [UE4のスレッドの流れとInput Latency改善の仕組み【2019】](https://www.docswell.com/s/EpicGamesJapan/K8V87K-UE4_Thread_InputLatency_2019)


参考链接：
- [C++Is there a way to get input from actor that isn a pawn/character ?](https://answers.unrealengine.com/questions/181782/c-is-there-a-way-to-get-input-from-actor-that-isn.html)
- [Check Keyboard Event in code](https://answers.unrealengine.com/questions/166084/check-keyboard-events-in-code.html)
- [UE4-学习笔记之二](sirokuma.cc/?p=567)
    - 这篇文章感觉好厉害
- [<<InsideUE4>>GamePlay架构(四)Pawn](https://zhuanlan.zhihu.com/p/23321666)
    - 知乎文章，可以一看


# Tips
## UE4中开启Focus的Debug表示
- ProjectSettings -> UserInterface -> Focus -> RenderFocusRule

上面的路径可以找到Focus的Debug表示设定，可以方便的确认当前的Focus位置。

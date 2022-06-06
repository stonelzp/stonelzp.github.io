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

记录一下关于UnrealEngine输入绑定的学习。
<!--more-->

[Arbitrary Key & Axis Binding in UE4 with C++](https://snardle.dev/posts/ue4-key-axis-binding/)
[UE4.26 Enhanced Inputについて](https://unrealengine.hatenablog.com/entry/2020/11/28/192500)

在自己编写Actor的Component的时候想要加上input的事件，但是在UE4中加入这种事件不是那么简单的。这需要对Actor的层级关系有一些了解。

<!--more-->

参考链接：
- [C++Is there a way to get input from actor that isn a pawn/character ?](https://answers.unrealengine.com/questions/181782/c-is-there-a-way-to-get-input-from-actor-that-isn.html)
- [Check Keyboard Event in code](https://answers.unrealengine.com/questions/166084/check-keyboard-events-in-code.html)
- [UE4-学习笔记之二](sirokuma.cc/?p=567)
    - 这篇文章感觉好厉害
- [<<InsideUE4>>GamePlay架构(四)Pawn](https://zhuanlan.zhihu.com/p/23321666)
    - 知乎文章，可以一看

---
title: UE4-关于Subsystem
permalink: ue4-subsystem
date: 2020-11-18 22:34:42
categories:
- UnrealEngine4
tags:
- UE4

---
这次要学习的是UE4的Subsystem，由4.22版本实装的功能。简单来说就是被某个系统所管理，会被自动实例化并拥有生命周期的类。

<!--more-->

# SububSystem概要

## GameInstanceSubsystem
多说无用，先介绍我目前最急迫使用的**GameInstance Subsubsystem**：

`UGameInstance`进行管理 <br>

参考的文章：
- [UE4 プログラミングサブシステムを試してみる](https://qiita.com/unknown_ds/items/afcff802ab17db486822)



## 参考资料
- [猫でも分かるUE4.22から入ったSubsystem](https://www2.slideshare.net/EpicGamesJapan/ue4-subsystem)

与之配套的讲解视频：  [第4回UE4何でも勉強会 in 東京 アーカイブ動画](https://www.youtube.com/watch?v=Wbq3KO3ZJaI)


我很早就应该看到但是却没有的文档：
- [Game Flow Overview](https://docs.unrealengine.com/en-US/Gameplay/Framework/GameFlow/index.html)

这里面的大图片想方法给它抠出来...



在使用这个Subsystem的时候需要对整个UE4的各种类的生命周期做些深入的了解：
- [UE4プログラマー勉強会 in 大阪 -エンジンの内部挙動について](https://www2.slideshare.net/com044/ue4-in)

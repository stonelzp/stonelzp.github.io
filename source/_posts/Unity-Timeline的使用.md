---
title: Unity-Timeline的使用
permalink: unity-timeline
date: 2019-11-13 12:38:00
categories:
- Unity
tags:
- unity
---
最近见到了好多Timeline的使用实例，但是对于Timeline的系统的学习和了解却没有。是时候仔细整理一下了。

<!--more-->

## Timeline



## Timeline中 「Marker」と「Signal、Signal Receiver」
参考[【Unity】Timelineからメソッドを呼ぶ新機能 「Marker」と「Signal、Signal Receiver」](http://tsubakit1.hateblo.jp/entry/2018/12/10/233146)这篇文章学习。

**这个功能在2019.1.a11之后的版本能够稳定使用。**

**由于并不是正式版本中的功能，今后可能会有变化。**







- [Timeline Signal の使用方法](https://blogs.unity3d.com/jp/2019/05/21/how-to-use-timeline-signals/)

## Animation
这一部分难以归类，主要是想要记录一下在Timeline中播放AnimationClip的问题的。

使用Timeline来播放动画十分方便，但是不足以应对所有情况。比如说两个动画之间的切换就是十分生硬的切换。这里有两种情况。

1. 同一个Timeline里面的AnimationClip切换: 这是最为普通解决方案也最是简单，那就是在Timeline中放置AnimationClip的时候，把**前一个Clip的最后面的部分和后一个Clip的前面的部分叠加放置**。没想到吧，在Timeline里面动画的片段是可以叠加放置的。就类似于旧Animation系统里面的`CrossFade`一样。
2. Timeline外的AnimationClip和Timeline内的AnimationClip动画状态的切换：这个我不知道更好的解决方案，自己的解决方案是**在AnimatorController中做好动画之间的切换部分，然后在Timeline中加入Signal调用Animator的Tirgger进行Animation的切换**。

再详细一点的说就是，将原本处在Timeline中的AnimationClip移到这个对象的AnimatorController中进行处理了。这里需要提及的是，AnimatorController的处理是挺耗时间 的，严重到旁边的大佬跟我说最好少使用的那种地步，实际上我也不清楚，既然人家说了就少用吧。但是想想那Timeline的使用，播放动画的时候也是需要这个对象有Animator组件的，也就是说也是用这个组件进行播放的。然后我看了一下，当不指定某一个AnimatorController(我们制作的AnimatorController)的时候，有一个叫**RuntimeAnimatorController**的东西顶替了指定的位置，Timeline有这个就可以播放AnimationClip而不需要我们指定AnimatorController，当然硬要指定也不是不可以。

但是最终我还是用了自己的AnimatorController......因为不知道更好的办法了。在完成可以让动画之间的切换如丝般顺滑之前要注意的是：
1. AnimatorController :HasExitTime  一定要取消，然后调整Setting中的参数，过渡的时间等等。
2. AnimatorController: Parameters 参数的设置主要是为了方便脚本在运行时切换，这里随机应变就好。
3. Timeline： Signal的制作。这就是这篇最开始的内容了，为什么要做这个就是因为方便调整时间或者方便其他比如设计师修改，毕竟你把人家的AnimationClip都移走了。
4. Scripts :最关键的脚本，要注意的是一是为Signal提供函数，二是设置AnimatiorController的状态。

暂时就想到这么多。

另外在对导入的模型的动画进行编辑的时候，发现了一下小的知识。
- Unity中对导入的模型**Edit**的时候，可以对导入的AnimationClip进行自定义，以其中一段的Clip为基础进行截取的操作，这样就可以方便的增加自己想要的片段。
    - 比如说想要在一段长的片段中截取Idle Clip，可以在**Clips**中添加新的片段，然后选定时间，调整**Loop Time**，方便循环播放，然后别忘了**Loop Pose**，这个可以让你截取的部分无缝循环播放。(这是Unity说的)

- 暂时没有在Unity中发现有能导入模型的动画而不导入模型的功能。这点不同于UE4的导入。

还有就是我在Inspector里面看到了下面的错误：
> The clip range is outside of the range of the source take.

虽然一眼就知道这个的原因是为什么，设置clip的Start和End的时候是从`-300`开始的，什么？还能从负数开始？这个我肯定不知道，但是事实就是这个项目里，这个clip，里面的Start，里面写着-300。

前几天遇见一个BUG，就是当**把Signal放到Timeline的第0帧的时候，这个Signal不会被执行，第二帧也不行**。据说是之Timeline的BUG。

参考文章:
- [《Unity備忘録》3DCGで設定したアニメーションをUnityで制御する方法](http://www.pointcloud.jp/blog_n29/)

---
title: Unity-DoTween-曲线的魅力
date: 2019-09-11 14:44:27
pemalink: unity-dotween-curve
categories:
- Unity 
tags:
- DoTween
- Curve
- Plugins
---
因为在使用DoTween的时候，有许多我不知道的用法，关于曲线的使用和一些很重要的名词。这篇文章主要用来记录这些曲线的操作和使用。

<!--more-->

首先是曲线的基础：

- [一から学ぶベジェ曲線](https://postd.cc/bezier-curves/)

虽然是随便搜也能搜到，暂且记下来吧。

## ease
- [Easing functions specify the rate of change of a parameter over time](https://easings.net/)

相当实用的网站，大佬教我搜**ease**的时候搜到的。

## DoTween
其实应该把这个放到**Unity-插件**的内容里的，感觉和曲线息息相关的插件，就放到了这里。
- [DOCUMENTATION](http://dotween.demigiant.com/documentation.php)

### DoPath/DoLocalPath
这个是我所需要的，我使用的地方。

关于这个插件有很多的使用，有免费版和Pro版。我看到有人整理的我就先上个链接
-[HowTo_DOTween.md](https://gist.github.com/anzfactory/da73149ba91626ba796d598578b163cc)

我自己的话充其量用了上面的路径功能，换了换set ease的条件。还有一些**RX**的使用，`Onstart()`,`OnComplete()`。

### DoTween停止操作
unity的Scene之间移动的时候需要停止DoTween的动作，当然tween的本体在切换Scene的时候不会被destory的。于是就出现了一个奇怪的现象。

**Tween的object把setActive(false)执行之后仍然继续活动**

这就奇了怪了，我的猜测是在Unity中跟gameobject相关的操作都是在主线程中运行，但是Tween的操作是子线程，但是反过来一想又不太对，Tween操作transform的移动不应该是主线程中吗，反正不太清楚，出现了这个问题，我的解决方案是：

先执行`Complete()`，再`setActive(false)`。

这里有一个疑问，我在使用Complete之前有尝试过Kill这个方法，但是，反正就是未能解决我遇到的问题，保险起见我是在执行Kill之前又添加了Complete的操作，虽然感觉不太好，但还是那么做了...

参考文章：[DOTweenをふわっとまとめてみた](https://qiita.com/kagigi/items/bdf4d42835add07b0077#complete-or-kill%E3%81%99%E3%82%8B%E3%81%BE%E3%81%A7%E6%AE%8B%E3%82%8A%E7%B6%9A%E3%81%91%E3%82%8B%E5%95%8F%E9%A1%8C)

## Unity DoTween Shortcut
> DOTween includes shortcuts for some known Unity objects, like Transform, Rigidbody and Material. You can start a tween directly from a reference to these objects (which will also automatically set the object itself as the tween target), like:
> ```
> transform.DOMove(new Vector3(2,3,4), 1);
> rigidbody.DOMove(new Vector3(2,3,4), 1);
> material.DOColor(Color.green, 1);

Dotween提供的这种快捷的方式很好理解，但是这种方式的实现就很厉害，值得学习。比如说这个transform的DOMove方法的shortcut实现：
```csharp
public static TweenerCore<Vector3, Vector3, VectorOptions> DoMove(this Transform target, Vector3 endValue, float duration, bool snapping = false);
```

C#该学的东西还是有很多很多啊，学无止境啊。

